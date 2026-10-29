## 1 获取 FileChannel

```java
// 与 FileInputStream 相关联的 FileChannel
FileChannel inChannel = FileInputStream.getChannel();
// 与 FileOutputStream 相关联的 FileChannel
FileChannel outChannel = FileOutputStream.getChannel();
```

- **返回**：返回一个与当前流关联的 `FileChannel`。
    - 从 `FileInputStream` 获取的通道是**只读**的。
    - 从 `FileOutputStream` 获取的通道是**只写**的。
- **特点**：该通道与流的内部文件描述符绑定，共享同一文件位置指针。这种方式为原有 IO 代码迁移到 NIO 提供了便利。

> [!note]
> ==该通道不能工作在非阻塞模式==。若需非阻塞文件操作，需使用 `FileChannel.open()` 并配合 `AsynchronousFileChannel` 等。

## 2 和 Buffer 的交互

### 2.1 read

Buffer <- Channel

```java
FileChannel.read(ByteBuffer dst）
```

- **功能**：从通道当前文件位置开始，读取字节序列并写入 ByteBuffer `dst` 中。
- **行为**：从 `dst` 的当前 `position` 开始写入，每写入一个字节，`position` 增加 1，直到 `position == limit`（缓冲区满）或读到文件末尾。
- 返回值：
	- 返回**实际读取的字节数**（可能小于缓冲区剩余空间）；
	- 如果已到达文件末尾且无更多数据，返回 `-1`。
- **阻塞**：`FileChannel` 的 read 是阻塞操作，直到至少一个字节被读取、或到达文件末尾、或发生 I/O 异常。
- **注意**：必须保证 Buffer 有剩余空间（`position < limit`），否则会抛出 `BufferOverflowException`。

### 2.2 write

Buffer -> Channel

```java
FileChannel.write(ByteBuffer src)
```

- 功能：将 `ByteBuffer src` 中自 position 到 limit 之间的字节写入通道，从当前文件位置开始写入。
- 行为：
	- 写入后，src.position 增加实际写入的字节数。
	- 返回写入的字节数，可能小于请求的数量（内部循环会尝试写入所有剩余字节，FileChannel 实现通常一次写完，除非被异步关闭或中断）。
- 调用前提：Buffer 必须处于读模式（即已通过 flip() 设置好有效数据区间），否则可能写入错误区域或触发缓冲区状态异常。

### 2.3 示例：文件复制

```java
package org.IO;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @Description: 文件复制
 */
public class NIO5 {
    public static void main(String[] args) {
        String filePath = "/home/leewenpeng/IdeaProjects/untitled/resources/";
        String fileName = "test";
        String nameAf = ".txt";
        
	    // 1. 创建文件输入输出流
        try(FileInputStream fis = new FileInputStream(filePath + fileName + nameAf);
            FileOutputStream fos = new FileOutputStream(filePath + fileName + "_copy5" + nameAf);
            // 2. 根据文件输入输出流获取 FileChannel
            FileChannel in = fis.getChannel();
            FileChannel out = fos.getChannel();
        ){
	        // 3. 创建 ByteBuffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 4. 从 FileInputStream Channel 中读取数据，写入到 buffer
            // 由于一次不一定能读取干净，因此通过 while 循环包裹进行多次读取
            // 这个被成为“多次提取”
            while(in.read(buffer) != -1){
	            // 5. 将 buffer 中数据写入到 Channel
                buffer.flip(); // 设置 buffer 为可读，且设置 Buffer 的内部参数
                out.write(buffer); // 写入整个 buffer 到 channel
                buffer.clear(); // 设置 buffer 为可写，且置 position = 0
            }
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 3 内存映射文件 -map

`FileChannel` 提供 `map()` 方法，它能将文件的一个区域直接映射到内存中，返回一个 [MappedByteBuffer](MappedByteBuffer.md) 对象。其核心优势在于，程序可以像操作内存数组一样直接读写文件，而无需显式进行 `read()`/`write()` 等系统调用，极大地简化了编程模型 。数据最终由操作系统在适当时候自动刷新到磁盘。

```java
// 方法签名
public abstract MappedByteBuffer map(FileChannel.MapMode mode, long position, long size) throws IOException {}

```

- 作用：将文件 (FileChannel) 的一个区域直接映射到内存 (MappedByteBuffer) 中
- 参数
	- mode：映射模式，决定内存区域的访问权限。必须和文件的打开权限匹配。
	- position：字偏移量，表示从文件哪里开始映射，$position ≥ 0$
	- size：需要映射的区域大小，$size \in [ 0, \text({Integer.MAX\_VALUE}) ]$
- 返回值：一个 [MappedByteBuffer](MappedByteBuffer.md) 对象

### 3.1 映射模式

`FileChannel.MapMode` 定义了三种映射模式，决定了映射后内存的访问权限和修改行为。

| 模式       | 常量                   | 行为                                   | 适用场景             |
| -------- | -------------------- | ------------------------------------ | ---------------- |
| **只读**   | `MapMode.READ_ONLY`  | 只能读取，写入会抛出 `ReadOnlyBufferException` | 文件分发、只读数据分析      |
| **读写**   | `MapMode.READ_WRITE` | 可读写，修改最终会传播到磁盘文件                     | 数据库存储、需要持久化修改的场景 |
| **写时复制** | `MapMode.PRIVATE`    | 私有映射，修改在内存副本上进行，不会写回磁盘，其他进程不可见       | 临时修改、私有分析副本      |

### 3.2 示例：文件复制 - 映射

```java
package org.IO;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @Description: 文件复制, 映射
 */
public class NIO5_1 {
    public static void main(String[] args) {
        String filePath = "/home/leewenpeng/IdeaProjects/untitled/resources/";
        String fileName = "test";
        String nameAf = ".txt";
        
        File file = new File();
        
        // 1. 创建字节流
        try(FileInputStream fis = new FileInputStream(file);
            FileOutputStream fos = new FileOutputStream(filePath + fileName + "_copy6" + nameAf);
            // 2. 获取字节流对应的 channel
            FileChannel in = fis.getChannel();
            FileChannel out = fos.getChannel();
        ){
	        MappedByteBuffer byteBuffer = in.map(
                    FileChannel.MapMode.READ_ONLY,  // 映射模式
                    0,                              // 字偏移量，0 表示从文件开头开始映射
                    in.size());                     // 映射的内容大小，in.size() 表示映射全部文件
            int len = out.write(byteBuffer);
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 4 零拷贝

它们可以**绕过用户空间缓冲区 (Buffer)，直接在两个通道 (Channel) 之间传输数据**，利用操作系统的零拷贝（Zero-Copy）机制大幅提升性能，尤其适合大文件或高频传输场景。

### 4.1 工作原理

#### 4.1.1 传统数据拷贝过程

![传统数据传输过程(ByteBuffer、Channel)](img/FileChannel/传统数据传输过程(ByteBuffer、Channel).md)

#### 4.1.2 transferT/transferFrom 数据传输过程

![数据传输过程（TransferTo、TransferFrom）](img/FileChannel/数据传输过程（TransferTo、TransferFrom）.md)

#### 4.1.3 对比

- **减少 CPU 占用**：无需 CPU 执行数据在用户态/内核态之间的拷贝，CPU 只需要管理 DMA 传输，负载显著降低。
- **减少上下文切换**：消除了用户态和内核态之间的多次切换。
- **吞吐量提升**：尤其在大量并发传输或大文件时，零拷贝可以成倍提高吞吐量。

### 4.2 TransferTo

```java
// 方法签名
public abstract long transferTo(long position, long count, WritableByteChannel targetChannel)
        throws IOException{}

// 使用
// 数据从 sourceChannel 传输到 targetChannel
long count = in.size();
long position = 0;
while (count > 0) {
	// 实际传输的字节数
	long realCount = sourceChannel.transferTo(position, count, targetChannel); 
    position += realCount;
    count -= realCount;
}
```

- **作用**：将当前文件通道中从指定 `position` 开始的 `count` 个字节直接传输到**目标可写通道** `target` 中（例如另一个 `FileChannel`、`SocketChannel` 等）。
- **参数**：
	- position：起始索引
	- count: 要传输的字节数。实际传输字节数可能小于 count。
	- targetChannel：要写入的目标 Channel。
- **返回值**：实际传输的字节数，可能小于 `count`（因此通常需要循环调用以确保全部传输）。
- **特点**：
	- **不修改当前通道的文件位置指针**，即传输后 `FileChannel` 的 `position()` 仍保持原值不变。
	- **通道限制**：`transferTo` 的源通道必须为 `FileChannel`，目标通道可以是任何 `WritableByteChannel`，如 `SocketChannel`、另一个 `FileChannel` 等。

#### 4.2.1 示例：文件复制 - 零拷贝 - TransferTo

```java
package org.IO;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @Description: 文件复制 零拷贝 tranferTo
 */
public class NIO5_3 {
    public static void main(String[] args) throws IOException {
        String filePath = "/home/leewenpeng/IdeaProjects/untitled/resources/";
        String fileName = "test";
        String nameAf = ".txt";
        
        // 1. 创建输入输出流
        FileInputStream fis = new FileInputStream(filePath + fileName + nameAf);
        FileOutputStream fos = new FileOutputStream(filePath + fileName + "_copy8" + nameAf);
        
        // 2. 获取输入输出的 channel
        FileChannel in = fis.getChannel();
        FileChannel out = fos.getChannel();
        
        // 3. transferTo 进行数据传输
        // 由于实际传输的字节数可能小于预计要传输的字节数
        long count = in.size();
        long position = 0;
        while (count > 0) {
            long realCount = in.transferTo(position, count, out); // 实际传输的字节数
            position += realCount;
            count -= realCount;
        }

		// 4. 关闭 Channel 和流 
        out.close();
        in.close();
        fis.close();
        fos.close();
    }
}
```

### 4.3 transferFrom

```java
// 方法签名
public abstract long transferFrom(ReadableByteChannel src, long position, long count) throws IOException {}

// 使用
// 数据从 sourceChannel 传输到 targetChannel
targetChannel.transferFrom(sourceChannel, position, count);
```

- **用途**：从**源可读通道** `src` 中读取最多 `count` 个字节，并写入到当前文件通道的指定 `position` 处。
- **返回值**：实际传输的字节数。
- **特点**：
	- **不修改当前通道的 `position` 指针**，写入位置由参数 `position` 绝对指定。
	- **通道限制**：`transferFrom` 的目标通道必须是 `FileChannel`，源通道可以是任何 `ReadableByteChannel`。

#### 4.3.1 示例：文件复制 - 零拷贝 - transferFrom

```java
package org.IO;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @Description: 文件复制 零拷贝 transferFrom
 */
public class NIO5_2 {
    public static void main(String[] args) throws IOException {
        String filePath = "/home/leewenpeng/IdeaProjects/untitled/resources/";
        String fileName = "test";
        String nameAf = ".txt";
        
        // 1. 创建输入输出流
        FileInputStream fis = new FileInputStream(filePath + fileName + nameAf);
        FileOutputStream fos = new FileOutputStream(filePath + fileName + "_copy7" + nameAf);
        
        // 2. 获取输入输出的 channel
        FileChannel in = fis.getChannel();
        FileChannel out = fos.getChannel();
        
        // 3. transferFrom
        out.transferFrom(in, 0, in.size());
        
        // 4. 关闭所有的流
        // 关闭流的顺序和开启流的顺序相反，尤其是存在依赖关系的流
        out.close();
        in.close();
        fis.close();
        fos.close();
    }
}
```
