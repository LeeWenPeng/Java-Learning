## 1  定义  

Channel 是 Java NIO 中对 I/O 操作的一种抽象，代表**数据源（source）与数据汇（sink）之间的开放连接**。它位于 `java.nio.channels` 包，是整个 NIO 体系的核心接口之一。

## 2  核心特性

1. **双向传输**  
   通道支持同时进行读和写操作，克服了传统流（InputStream/OutputStream）单向移动的局限。

2. **与 Buffer 协同工作**  
   通道自身不暴露直接读写数据的方法，所有数据传输都必须通过缓冲区（Buffer）中转：
   - 读取：从通道读到 Buffer（`channel.read(buffer)`）
   - 写入：从 Buffer 写到通道（`channel.write(buffer)`）

3. **非阻塞与多路复用**  
   - `SocketChannel`、`ServerSocketChannel` 和 `DatagramChannel` 可通过 `configureBlocking(false)` 设置为非阻塞模式。
   - 非阻塞通道配合 **Selector** 使用，能够以单线程管理多个 I/O 通道，实现高扩展性的多路复用 I/O。
   - *注意：`FileChannel` 不能工作在非阻塞模式。*

4. **分散读取与聚集写入（Scatter/Gather）**  
   通道支持在一次 I/O 操作中处理多个 Buffer：
   - **分散读取**：将数据从通道依次读入多个 Buffer。
   - **聚集写入**：将多个 Buffer 中的数据依次写入通道。

5. **I/O 操作可中断**  
   通道上因 I/O 而阻塞的线程可被其他线程中断（抛出 `ClosedByInterruptException`），线程状态管理更灵活。

---

**主要接口与实现体系**

```java
java.nio.channels.Channel (顶层接口)
├── ReadableByteChannel / WritableByteChannel (字节读/写接口)
├── GatheringByteChannel / ScatteringByteChannel (聚集/分散接口)
└── 具体实现类 (见下表)
```

| 实现类                                           | 用途               | 跨平台   | 非阻塞支持 |
| --------------------------------------------- | ---------------- | ----- | ----- |
| [FileChannel](FileChannel.md)                 | 文件读写、截取、锁定、内存映射  | 文件系统  | ✗     |
| `SocketChannel`                               | TCP 客户端通信        | 网络套接字 | ✓     |
| [ServerSocketChannel](ServerSocketChannel.md) | TCP 服务端通信（监听新连接） | 网络套接字 | ✓     |
| `DatagramChannel`                             | UDP 数据报通信        | 网络套接字 | ✓     |

---

**与传统 I/O 的对比要点**

- 传统 BIO 面向流（Stream），单向、阻塞；NIO 通道面向通道，大多支持双向且可非阻塞。
- 通道与 Buffer 解耦，更灵活地管理内存和数据传输；原生流直接读写字节数组，无 Buffer 抽象。
- 通道通过 Selector 可实现单个线程处理海量并发连接，这是传统阻塞 I/O 难以做到的。

---
