## 1  Java 中的三种模型

- BIO：和 Unix/Linux 中的 BIO 模型一致
- NIO：和 Unix/Linux 中的模型基本一致，但底层实现是通过 epoll
- AIO：和 Unix/Linux 中的模型实现不太一致，只是提供了异步的形式，底层实际是使用 epoll

也就是说，在 JAVA 中，对于 NIO 和 AIO 两种模型，底层实际上是 I/O 多路复用的模型，JAVA 对该模型的接口进行了不同的封装，对外呈现了 NIO 和 AIO 的形式。

## 2  NIO

### 2.1  NIO 的原理

- NIOIO 流是基于管道 (Channel) 和缓存区 (Buffer) 的 [0-NIO](NIO/0-NIO.md)
- NIO 框架是基于 IO 多路复用的
	- [epoll相关面试题](NIO/epoll相关面试题.md)

## 3  AIO

### 3.1  AIO 对比 NIO

- 基本抽象相似
	- AsynchronousServerSocketChannel <-> ServerSocketChannel
	- AsynchronousSocketChannel <-> SocketChannel
- 业务逻辑不同
	- NIO 通过设置 selector，遍历 selectedKeys 集合进而实现业务逻辑
	- AIO 通过指定 CompletionHandler 回调接口实现业务逻辑
- 等待就绪阶段不消耗 CPU，属于 " 空等 "。  
- 数据操作阶段（如 memory copy）速度极快，带宽可达 GB/s 级，几乎不耗时。

| 模型  | `socket.read()` 行为                               |
| --- | ------------------------------------------------ |
| BIO | 若接收缓冲区无数据，阻塞等待直到数据到来并返回。                         |
| NIO | 若有数据则读入用户内存并返回；否则立即返回 0，永不阻塞。                    |
| AIO | 不仅等待就绪非阻塞，连数据从网卡到内存的拷贝过程也是异步的，用户只需关注 " 读完了 " 通知。 |
