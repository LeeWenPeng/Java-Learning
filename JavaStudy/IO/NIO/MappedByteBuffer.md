## 1  概念

`MappedByteBuffer` 是 Java NIO 中实现内存映射文件的核心类，它继承自 `ByteBuffer`，除了拥有 `ByteBuffer` 的全部方法外，还提供了几个与内存映射特性直接相关的操作。

`MappedByteBuffer` 的继承链：

```java
java.lang.Object
  └── java.nio.Buffer
        └── java.nio.ByteBuffer
              └── java.nio.MappedByteBuffer
```

因此它天然拥有 `ByteBuffer` 的所有方法（`get()`、`put()`、`flip()`、`clear()`、`capacity()`、`position()`、`limit()` 等），同时还支持基本数据类型的读写（如 `getInt()`、`putDouble()`）。

## 2  `MappedByteBuffer` 特有方法

### 2.1  `force()`

```java
public final MappedByteBuffer force()
```

- **作用**：将映射缓冲区中**所有已修改但尚未写入磁盘的内容**强制同步到底层存储设备。
- **行为**：
	- 对 `READ_WRITE` 模式下的映射有效。在 `READ_ONLY` 或 `PRIVATE` 模式下，修改本身就不可能发生（只读会抛异常，私有修改不会写回），因此调用 `force()` 无实际意义但不会出错。
	- 此方法是**阻塞式**的，直到所有脏页（dirty pages）被刷新到存储设备后才返回。
	- 对于网络文件系统（NFS）或某些特殊文件系统，`force()` 会**尽力**保证数据持久化，但具体的持久性级别取决于操作系统实现。

> [!question] 为什么不用 `FileDescriptor.sync()` ？  
> - `force()` 作用于当前映射区域，
> - FileDescriptor 的 `sync()` 作用于整个文件。
> - 通常使用 `force()` 更精确且性能更好。

- **示例**：

  ```java
  MappedByteBuffer buffer = channel.map(MapMode.READ_WRITE, 0, 1024);
  buffer.put("Hello, Mapped!".getBytes());
  buffer.force(); // 确保写入内容永久保存
  ```

### 2.2  `load()`

```java
public final MappedByteBuffer load()
```

- **作用**：**尽力**将映射文件的内容加载到物理内存中，使得后续访问更快。
- **行为**：
	- 调用后，操作系统会尝试将文件对应的页面预加载到主存（触发缺页中断），但这只是一个**提示**，实际加载情况取决于操作系统和当前可用内存。
	- `load()` 在返回时**不一定保证所有数据都在内存中**，它只是一个性能优化建议。
- **返回值**：返回 `this` 引用，支持链式调用。

> [!important] 重要提示
不要依赖 `load()` 来确保数据立即可用；应结合 `isLoaded()` 或直接访问数据，让缺页中断自动处理。

### 2.3  `isLoaded()`

```java
public final boolean isLoaded()
```

- **作用**：判断映射缓冲区的内容是否**完全**驻留在物理内存中。
- **返回值**：如果很可能所有页面都在物理内存中，则返回 `true`；否则返回 `false`。
- **局限性**：
	- 这是一个**尽力而为**的检查，即使返回 `true`，在方法调用后页面仍可能被操作系统换出；返回 `false` 也不代表不能高效访问。
	- 主要用于诊断和性能监控，不应作为程序逻辑的关键依赖。
- **示例**：

  ```java
  MappedByteBuffer buffer = channel.map(MapMode.READ_ONLY, 0, fileSize);
  buffer.load();
  if (buffer.isLoaded()) {
      System.out.println("数据已完整载入内存，可快速访问");
  } else {
      System.out.println("部分数据可能仍需从磁盘读取");
  }
  ```
