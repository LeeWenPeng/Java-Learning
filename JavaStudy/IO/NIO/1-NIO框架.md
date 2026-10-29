Java 中的 NIO 框架是基于 Unix/Linux 的 IO 多路复用框架的。

- Linux 中依赖 epoll
- Windows 中依赖 iocp

## 1  传统 BIO 模型及其困境

传统阻塞 I/O（Blocking I/O）模型采用 "**每连接每线程**" 的模式：

- 主线程循环调用 `accept()` 阻塞等待新连接。
- 每个连接分配一个线程，在线程内循环执行 `read()`、处理、`write()` 等操作。
- 核心函数 `accept()`、`read()`、`write()` 均为同步阻塞，导致线程在等待 I/O 时被挂起。

### 1.1  通过多线程提升性能

1. 利用多核 CPU。
2. 当线程因 I/O 阻塞时，CPU 空闲，其他线程可以继续工作。

### 1.2  BIO 多线程的缺陷

该模型在连接数较低（<1000）时表现尚可，线程池还能充当缓冲漏斗，但根本问题在于**线程本身是昂贵的资源**：

- **创建/销毁成本高**：在 Linux 中线程就是轻量级进程，创建和销毁均涉及重量级系统调用。
- **内存占用大**：Java 线程栈通常需 512KB～1MB，数千线程会耗尽 JVM 内存。
- **切换开销高**：线程上下文切换需要保存/恢复现场，引发系统调用。线程过多时，CPU 用于切换的时间可能超过实际执行时间，表现为 `sy` 使用率飙升、load 升高，系统近乎不可用。
- **负载锯齿效应**：大规模阻塞线程被同时唤醒时，瞬间负载剧烈波动。

因此，在面对十万乃至百万级长连接时，传统 BIO 无能为力。

## 2  NIO 如何结合事件模型释放线程

BIO 不得不使用多线程的根源在于无法提前知道 I/O 是否就绪，只能 " 傻等 "，且无法从 `read()`/`write()` 中途退出。  
NIO 的读写函数立即返回的特性，提供了**单线程管理多个连接**的可能：若某连接不能读写（返回 0），就记录下来（通过 Selector 注册事件），然后立刻切换到其他就绪连接上工作。

### 2.1  基本事件与处理器注册

NIO 的核心事件：**读就绪、写就绪、新连接到达**。  

Reactor 编程模型：注册所有感兴趣的事件处理器，单线程轮询并分派。

1. 注册感兴趣的事件及对应处理器。
2. 在死循环中调用 `Selector.select()`，阻塞等待事件。
3. 事件到来后，根据标记分发执行相应处理器。

伪代码：

```java
// 事件处理器接口
interface ChannelHandler {
    void channelReadable(Channel channel);
    void channelWritable(Channel channel);
}

// 主循环
class IoThread extends Thread {
    public void run() {
        while (true) {
            Channel channel = Selector.select();  // 阻塞等待就绪事件
            if (channel.event == ACCEPT) {
                registerNewChannelHandler(channel);
            } else if (channel.event == WRITE) {
                getHandler(channel).channelWritable(channel);
            } else if (channel.event == READ) {
                getHandler(channel).channelReadable(channel);
            }
        }
    }
}
```

## 3  NIO 相对比 BIO 多线程模型的优化

NIO 多线程：

- **事件分发器（Reactor 线程）**：单线程处理 `select()` 事件就绪分发。  
- **I/O 处理线程池**：负责真正的 `read`/`write` 等纯 CPU 操作，一般线程数等于 CPU 核数。  
- **业务线程池**：处理协议解析、业务逻辑以及可能的阻塞操作（DB、RPC 等）。

**Java Selector 的并发限制**：同一 channel 的 select 操作不能被并发调用。因此，一个 socket 只能归属于一个 I/O 线程，但一个 I/O 线程可以管理多个 socket。

连接注册与读写处理可以进一步分离，提升海量连接的分配和读写能力。

## 4  Proactor 模式

两者核心都是通过**事件分发器**获取 I/O 事件通知并回调处理器。

|            | Reactor                                                | Proactor                                                   |
|------------|--------------------------------------------------------|------------------------------------------------------------|
| I/O 模型   | 同步 I/O                                                | 异步 I/O                                                   |
| 通知事件   | I/O 就绪（can read/can write）                          | I/O 完成（read/write done）                                 |
| 读写执行者 | 事件处理器（用户代码）自行执行实际读写系统调用              | 操作系统内核代为执行读写，完成后通知                        |
| 控制流     | 分发器通知处理器 " 可以操作 "，处理器执行操作并处理数据        | 处理器发起异步读写请求，分发器等待完成事件，然后回调处理器处理已就绪的数据 |

### 4.1  Proactor

### 4.2  Reactor 模拟 Proactor

**模拟 Proactor**：在不支持异步 I/O 的操作系统上，可以将读/写操作从处理器上移至分发器中执行。  

- 标准 Reactor：等待就绪 → 用户处理器执行 read → 处理数据。  
- 模拟 Proactor：等待就绪 → 分发器执行 read → 通知用户处理器 " 读完成 " 并处理数据。  
这样做可以对外暴露完整的异步接口，实现跨平台移植，且不会增加总体工作量或削弱性能。

## 5  客户端场景下的 NIO 应用

### 5.1  每连接顺序请求 —— Redis 客户端

- 利用 Redis 服务端全局串行的特性，单连接保证请求与响应顺序一致。
- 客户端使用单线程 + 阻塞队列缓冲命令，待 channel 可写时**管道式（pipeline）批量**发送。
- 读取响应时，匹配请求队列中的 Future 并完成回调，实现异步化。

### 5.2  多连接短连接 —— HttpClient 抓取

- 面对大量短 HTTP 连接，受线程数限制时，改用 NIO 单线程进行连接、写、读。
- 若操作系统暂时无法完成连接或读写，仅注册事件等待下一次循环，避免线程阻塞。
- 可通过 Socket 引用来映射不同的请求/响应，实现多路复用。

### 5.3  常见 RPC 框架（Thrift、Dubbo）

- 框架内部维护请求 ID 与 Future 的映射关系。
- 利用 NIO 长连接 + 多路复用，提供高性能异步调用能力。

## 6  Selector.wakeup()

**作用**：解除阻塞在 `Selector.select()` 上的线程，使其立即返回。  
典型唤醒场景：

- 有新 channel 或事件注册。
- channel 关闭或取消注册。
- 定时器事件等高优先级任务需要及时处理。

**原理**：

- Linux：基于 `pipe` 系统调用，创建管道并注册到 Selector 的 fd 集合，wakeup 时向管道写 1 字节。
- Windows：使用 loopback TCP 连接（因为 Windows 管道无法加入 fd_set），原理类似。
每次唤醒都涉及一次 I/O 写入，开销不可忽视。

## 7  Buffer 的选择

操作系统一次读写通常需要在内核空间与用户空间之间拷贝数据。

- **HeapByteBuffer**：数据分配在 JVM 堆中，I/O 操作时会进行 " 用户空间→内核空间 " 的拷贝（也可能多一次临时直接内存拷贝）。
- **DirectByteBuffer**：分配在直接内存，能够减少一次内核与用户空间的拷贝，提升效率。但创建和销毁成本高，通常配合内存池管理。

> [!suggestion]
> - 数据量小、并发不高的中小应用可用 HeapByteBuffer，维护更简单。
> - 高吞吐、大数据量的场景宜用 DirectByteBuffer，并借助内存池降低管理开销。
