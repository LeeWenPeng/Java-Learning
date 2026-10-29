## 1  BIO 服务器

```java
package org.IO;

import java.io.IOException;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Scanner;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class BIO{
    private static volatile boolean stop = false;
    private static final int MAX_CONNECTIONS =  2;

    public static void main(String[] args) throws IOException, InterruptedException {
        int connectionNum = 0;
        int port = 8888;

        ExecutorService service = Executors.newCachedThreadPool();
        try (ServerSocket serverSocket = new ServerSocket(port)){
            System.out.println("Server started on port " + port);

            while(!stop && connectionNum < MAX_CONNECTIONS){
                // 接受新的链接
                Socket socket = serverSocket.accept();
                connectionNum++;
                System.out.println("Accepted connection #" + connectionNum);

                if(connectionNum == MAX_CONNECTIONS){
                    stop = true;
                    System.out.println("Reached max connections, no further accepts.");
                }

                // 提交处理任务
                service.execute(
                        () -> handleClient(socket)
                );
            }
            System.out.println("Server stop accepting new connections. Waiting for active tasks to complete…");
        }finally {
            service.shutdown();
            if(!service.awaitTermination(5, TimeUnit.SECONDS)){
                service.shutdownNow();
            }
            System.out.println("Server shut down.");
        }
    }
    private static void handleClient(Socket socket){
        try(socket;
            Scanner scanner = new Scanner(socket.getInputStream());
            PrintStream printStream = new PrintStream(socket.getOutputStream());
        ) {
            // 读取客户端输入直到链接断开或服务器主动停止
            while(!stop && scanner.hasNextLine()){
                String line = scanner.nextLine().trim();
                if(line.isEmpty()){
                    continue;
                }
                printStream.println("PONG: " + line);
                if(printStream.checkError()){
                    break;
                }
            }

        } catch (IOException e) {
            System.out.println("Client handler error: " + e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```

### 1.1  测试

```bash
nc -v localhost 8888
```
