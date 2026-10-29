## 1  创建

```java
ServerSocketChannel server = ServerSocketChannel.open() // 创建
server.configureBlocking(false); // 设置非阻塞
```

## 2  绑定 Socket Address

```java
// 设置 serverSocket 信息
InetSocketAddress address = new InetSocketAddress(String IPAddress, int PORT); // Socket Address
server.bind(address);```
