## 1  定义

Buffer 是 Java NIO 中专门用于存储数据的**容器对象**，其内部通过一个**原始类型数组**存储实际数据。`Buffer` 本身是一个抽象类，位于 `java.nio` 包，除 `boolean` 外所有基本数据类型都提供了对应的具体实现。

## 2  内部状态参量  

每个 Buffer 都维护了四个核心属性，用于精确控制数据读写边界：

- **容量（capacity）**：缓冲区能够容纳元素的最大数量，创建时确定且不可修改。
- **界限（limit）**：可读或可写元素的第一个不可访问索引。在写模式下，limit 通常等于 capacity；在读模式下，limit 表示已写入数据的末尾。
- **位置（position）**：下一个待读取或待写入元素的索引。随 `get()`/`put()` 自动推进。
- **标记（mark）**：用于临时保存某个位置，可通过 `reset()` 将 position 恢复到此标记。初始时未定义。

## 3  分配缓冲区 -allocate

所有具体实现类均通过静态工厂方法 `allocate(int capacity)` 创建基于 JVM 堆内存的缓冲区对象。

```java
// 创建 ByteBuffer
ByteBuffer byteBuffer = ByteBuffer.allocate(int capacity);

// 创建 intBuffer
IntBuffer intBuffer = IntBuffer.allocate(1024);
```

- **功能**：在 JVM 堆内存中创建一个 `Buffer`，容量为 `capacity` 字节。
- **状态**：
    - `position = 0`，`limit = capacity`，`mark = undefined`
    - 此状态表示缓冲区已准备好**被写入**（写模式）。

### 3.1  直接缓冲区 -allocateDirect

ByteBuffer 可以通过 `allocateDirect` 方法创建直接缓冲区。直接缓冲区的内存分配在操作系统直接内存区域，而非 JVM 堆中。

```java
ByteBuffer byteBuffer = ByteBuffer.allocateDirect(int capacity);
```

- **优势**：在进行 I/O 操作时，内核可直接访问该内存，**避免了数据在 JVM 堆与本地内存间的额外拷贝**，显著提升大规模数据传输效率。
- **代价**：分配和回收成本较高，且内存不受 JVM 垃圾回收直接管理。
- **仅 `ByteBuffer` 可用**

## 4  数据存取  

通过 `get()` 和 `put()` 系列方法读写内部数据。(这里以 ByteBuffer 举例)

### 4.1  相对操作和绝对操作

相对操作不指定 index，绝对操作指定 index

| 操作类型                          | 是否影响 `position`   | 是否受 `limit` 约束            | 常见用途        |
| ----------------------------- | ----------------- | ------------------------- | ----------- |
| **相对操作** (如 `get()`)          | 是，读/写后自动增加相应单位    | 是，可操作空间由 `remaining()` 决定 | 流式读写、顺序处理   |
| **绝对操作** (如 `get(int index)`) | 否，`position` 保持不变 | 否，只受 `index` 范围约束         | 随机访问、固定位置修改 |

### 4.2  get 系列方法

#### 4.2.1  单元素读

```java
// 单元素读
byte get()
```

- 读取当前 `position` 处的字节，然后 `position++`。
- 若 `position >= limit` 则抛出 `BufferUnderflowException`

```java
byte get(int index)
```

- 作用：读取索引 `index` 处的元素，不改变 `position`。
- 若 `index < 0` 或 `>= limit` 则抛出 `IndexOutOfBoundsException`
- 参数：
	- index：要读取的位置的索引

#### 4.2.2  多元素读

```java
ByteBuffer get(byte[] dst, int offset, int length)
```

- 作用：将缓冲区中自 `position` 开始的 `length` 个字节读入数组 `dst` 的 `offset` 处。
- 要求 `length <= remaining()`，否则抛出异常。操作后 `position += length`。

```java
ByteBuffer get(byte[] dst)

// 等价于
get(dst, 0, dst.length)
```

- 作用：将缓冲区中剩余数据尽可能读入数组。
- 行为
	- 读取字节数为 `min(remaining(), dst.length)`
	- 操作后 `position` 增加相应数量。
	- 若剩余数据不足数组长度，抛出 `BufferUnderflowException`

#### 4.2.3  基本类型读取

`ByteBuffer` 支持直接读出整型、浮点等数据，默认按大端字节序（BIG_ENDIAN）。

```java
char getChar()
char getChar(int index)

int getInt()
int getInt(int index)

// 略
```

- 相对操作要求 `remaining()` >= 对应字节数，否则抛出 `BufferUnderflowException`；
- 绝对操作要求 `index + 字节数 <= limit`，否则抛出 `IndexOutOfBoundsException`。
- 字节序可通过 `order(ByteOrder)` 修改，影响这些多字节类型的读取结果。

### 4.3  put 系列方法

`put` 系列方法与 `get` 对称，但方向相反。相对写入同样受 `limit` 约束并推进 `position`。

#### 4.3.1  单元素写

```java
ByteBuffer put(byte b)

```

- 在当前 `position` 处写入字节，`position++`。若 `position >= limit` 则抛出 `BufferOverflowException`

```java
ByteBuffer put(int index, byte b)
```

- 在索引 `index` 处写入字节，不改变 `position`。若 `index` 越界则抛出 `IndexOutOfBoundsException`

#### 4.3.2  多元素写

```java
ByteBuffer put(byte[] src)
```

- 将数组 `src` 中全部内容写入缓冲区，等价于 `put(src, 0, src.length)`。要求 `remaining() >= src.length`，否则抛异常

```java
ByteBuffer put(byte[] src, int offset, int length)
```

- 将数组从 `offset` 开始的 `length` 个字节写入。`position` 增加 `length`

```java
ByteBuffer put(ByteBuffer src)
```

- 将源缓冲区 `src` 的**剩余内容**（`src.remaining()` 个字节）写入当前缓冲区。操作后源缓冲区的 `position` 增加对应数量，相当于 " 搬移 " 数据

#### 4.3.3  基本类型写入

```java
putChar(char value)
putChar(int index, char value)

putInt(char value)
putInt(int index, int value)
```

- 相对操作会按给定字节序将值转换为字节序列放入缓冲区；
- 绝对操作则在指定偏移写入，不改变 `position`。

## 5  工作模式

Buffer 在两种核心状态间切换，配合了 Channel 的数据传输：

- **写模式**：准备写入数据到缓冲区内。`position` 指向下一个写入位置，`limit` 等于 `capacity`。
- **读模式**：准备从缓冲区中读出数据。调用 `flip()` 后，`limit` 被设置为原 `position`，`position` 回置为 0。

## 6  工作模式切换  

### 6.1  flip

```java
Buffer.flip()
```

- **功能**：将 Buffer 从写模式切换到读模式，为将数据从 Buffer 传递到通道（或用户程序）做准备。
- **内部操作**：
    - `limit = position` （将已写入数据的末尾边界锁定）
    - `position = 0` （重置到开头，准备读取）
    - `mark = -1` （若有标记则丢弃）

> [!important]
> 翻转后的 Buffer 可读范围是 `[0, limit)`，即之前写入的所有字节。若不调用 `flip`，直接对 Buffer 执行 `write` 会从当前 `position` 写入，可能写出未初始化数据。

### 6.2  clear

```java
Buffer.clear()
```

- **功能**：将 Buffer 重置为 " 准备接收新数据 " 的写模式。
- **内部操作**：
    - `position = 0`
    - `limit = capacity`
    - `mark = -1`

> [!note] 本质
> - **不清除数组内容**，仅移动指针。这意味着新写入的数据会直接覆盖旧数据；如果在写模式下直接读取，可能看到残留数据，因此必须先 `flip`。

### 6.3  rewind

```java
Buffer.rewind()
```

- 保持读模式，仅将 `position` 置 0，可重新读取已写入的全部数据。

## 7  与 Channel 的交互模式

### 7.1  read

```java
channel.read(buffer)
```

- 数据从 Channel 读取到 Buffer
- 通常 Buffer 处于写模式

### 7.2  write

```java
channel.write(buffer)
```

- 数据从 Buffer 写入到 Channel
- Buffer 需先 `flip()` 切换到读模式。

## 8  数据类型转换

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);

// 转换为 int 类型的 Buffer
IntBuffer intBuffer = buffer.asIntBuffer();

// 转换为 double 类型的 Buffer
DoubleBuffer doubleBuffer = buffer.asDoubleBuffer();
```

- 通过 `asXXXBuffer()` 方法将某个类型 Buffer 转换为 `XXX` 类型的 Buffer

### 8.1  只读视图

可通过 `asReadOnlyBuffer()` 获得只读视图，底层共享数据，仅屏蔽写操作。

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);

// 通过 asReadOnlyBuffer 获取只读视图
ByteBuffer readOnlyBuffer = buffer.asReadOnlyBuffer();
```
