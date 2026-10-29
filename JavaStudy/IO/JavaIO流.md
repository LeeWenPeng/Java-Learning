![IO关系](img/JavaIO/IO关系.md)

Java IO 体系的设计遵循**装饰器模式**，核心分为两大类：

- **字节流**：以 `InputStream`/`OutputStream` 为根基，操作的最小单位是 `byte`（8 位），适用于所有类型的文件（图片、视频、文本等）。
- **字符流**：以 `Reader`/`Writer` 为根基，操作的最小单位是 `char`（16 位 Unicode），内置了字符编码/解码过程，能正确处理文本数据，避免乱码。
- `BufferedInputStream/BufferedOutputStream` 或 `BufferedReader/BufferedWriter` 是对基础流的包装（装饰），增加内部缓冲区，以减少实际 I/O 的系统调用次数，大幅提升读写效率。

## 1  BIO

[0-BIO](BIO/0-BIO.md)

## 2  NIO


