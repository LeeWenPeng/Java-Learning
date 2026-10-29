`Scanner` 是 Java 提供的一个非常实用的**文本扫描器**。它的主要设计目标是让你能方便地从各种输入源（如键盘、文件、字符串）中读取并解析出基本类型（如 `int`, `double`）和字符串。

## 1  底层机制：它是如何工作的？

理解 `Scanner` 的工作流程能帮我们规避很多问题。

1. **分词 (Tokenization)**：`Scanner` 的核心是将输入文本按**分隔符 (Delimiter)** 切分成一连串的 " 标记 (Token)"。默认的分隔符是**空白字符 (whitespace)**，包括空格、制表符 (`\t`) 和换行符 (`\n`) 等。你可以通过 `useDelimiter()` 方法自定义分隔符。
2. **数据转换**：识别出标记后，`Scanner` 会根据你调用的 `next…()` 方法（如 `nextInt()`, `nextDouble()`），将对应标记的字符串内容 " 翻译 " 成你想要的 Java 数据类型，并返回。
3. **内部缓冲**：为了提高效率，`Scanner` 内部维护了一个字符缓冲区。它不会从输入源（如文件）中一个字符一个字符地读，而是一批批地读入缓冲区，然后再从缓冲区中解析数据。

## 2  核心方法详解

掌握了工作的原理，我们再看看 `Scanner` 提供的 " 工具箱 "——即各种方法，主要可以分为以下几类：

### 2.1  用于读取数据 (`next…()` 系列)

这是 `Scanner` 最核心的功能，但使用时**务必小心 `next()` 和 `nextLine()` 的混用陷阱**。

| 方法                                                                  | 功能描述                            | 要点与注意事项                                                                                                                          |
| :------------------------------------------------------------------ | :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------- |
| **`nextXxx()`**<br>例如: `nextInt()`, `nextDouble()`, `nextBoolean()` | **读取下一个标记**，并将其转换为对应的基本数据类型后返回。 | - **不消耗换行符**：读取完数字后，用户按下的回车键 (`\n`) 会留在缓冲区中，这通常是混用错误的根源。<br>- 如果标记无法被正确转换（如对 "abc" 调用 `nextInt()`），会抛出 `InputMismatchException`。 |
| **`next()`**                                                        | **读取下一个标记**，并以 `String` 类型返回。   | - **以空白符为界**：无法读取包含空格的字符串（如 "Hello World" 只会被识别为 "Hello"）。<br>- **同样不消耗换行符**。                                                    |
| **`nextLine()`**                                                    | **读取当前行剩余的所有内容**，直到遇见换行符 (`\n`) | - **消耗换行符**：它会读走包括换行符在内的整行内容（但返回值中不包含换行符）。<br>- **可以读取空格**：适合读取带空格的完整句子。<br>- 这也是用来**清除输入缓冲区中残留换行符**的常用方法。                       |

### 2.2  用于探测输入 (`hasNext…()` 系列)

这些方法用于在读取前进行安全检查，返回 `boolean` 值，这对于循环和异常处理至关重要。

| 方法 | 功能描述 |
| :--- | :--- |
| **`hasNext()`** | 检查输入中是否还有下一个标记。 |
| **`hasNextInt()`, `hasNextDouble()`…** | 检查下一个标记能否被解析为指定的数据类型。这是**防御性编程**的核心。 |
| **`hasNextLine()`** | 检查输入中是否还有下一行。 |

### 2.3  配置类方法

`Scanner` 允许你自定义其行为。

| 方法 | 功能描述 |
| :--- | :--- |
| **`useDelimiter(String pattern)`** | 设置新的分隔符（支持正则表达式）。例如，`useDelimiter(", ")` 会按逗号和空格来分词。 |
| **`useLocale(Locale locale)`** | 设置地区，影响数字的解析格式（例如，不同国家对小数点和千位分隔符的处理不同）。 |
| **`useRadix(int radix)`** | 设置解析整数时的进制，默认是 10 进制。例如，`useRadix(16)` 会让 `nextInt()` 尝试解析十六进制数字。 |

### 2.4  资源管理方法

`Scanner` 实现了 `Closeable` 接口，使用完毕后必须正确关闭。

| 方法 | 功能描述 |
| :--- | :--- |
| **`close()`** | 关闭扫描器。如果底层是 `System.in`，关闭后**整个程序将无法再从标准输入读取数据**，因此要格外谨慎。 |

## 3  与 I/O 流的区别与定位

在 Java I/O 体系中，`Scanner` 的定位非常清晰：

- **职责不同**：`InputStream`/`Reader` 等负责**数据的传输**（读写字节/字符），好比是 " 管道 "；而 `Scanner` 负责对传输来的 **文本数据进行解析**，好比是管道出口的 " 水龙头 "。
- **层次不同**：`Scanner` 是**更高层次**的抽象，它构建在 `InputStream` 或 `Reader` 之上，并将其复杂性封装起来。
- **能处理非字符串类型**：它可以直接调用 `nextInt()` 等方法，这是 `BufferedReader` 等基础流无法直接做到的。与 BufferedReader 相比，它的特点是方便但性能较低【8†L38-L40】。

## 4  代码实战与深度辨析

### 4.1  核心陷阱：`nextInt()` 后直接调用 `nextLine()`

这是新手最容易遇到的 "bug"：程序似乎直接跳过了 `nextLine()`。原因有二：

- `nextInt()` 在读取数字后，只留下了 `\n`。

`nextLine()` 会把残留的 `\n` 当作一个 " 空行 " 立即消费并返回。

```java
Scanner scanner = new Scanner(System.in);
System.out.print("请输入年龄：");
int age = scanner.nextInt(); // 用户输入 '25' 并按下回车，缓冲区留下 '\n'

System.out.print("请输入姓名：");
String name = scanner.nextLine(); // 此行代码会立即读取到上一步的 '\n' 并返回空字符串，不会给用户输入机会。

System.out.println("年龄：" + age + ", 姓名：" + name); // 输出: 年龄: 25, 姓名: [空]
// 产品经理怒吼：'为什么跳过姓名输入？！'
```

**解决方案**：

- **方案一（教科书式清理）**：在调用 `nextInt()` 后，**额外添加一个 `nextLine()` 来消化掉残留的换行符**。

  ```java
  int age = scanner.nextInt();
  scanner.nextLine(); // 手动消费掉换行符
  String name = scanner.nextLine(); // 现在可以正常读取
  ```

- **方案二（统一按行处理）**：**所有输入统一使用 `nextLine()` 读取**，然后手动进行类型转换，从源头避免问题。

  ```java
  String ageStr = scanner.nextLine();
  int age = Integer.parseInt(ageStr); // 手动将字符串转为 int
  
  String name = scanner.nextLine();
  ```

- **方案三（防御式探测）**：在调用 `nextLine()` **前，先用 `hasNextLine()` 判断**，确保有可读的行。

### 4.2  安全读取：使用 `hasNext` 系列方法进行防御性编程

直接将 `nextInt()` 等放在循环中，一旦用户输入了非法字符（如字母），程序就会因 `InputMismatchException` 崩溃。正确的做法是用 `hasNext…()` 进行探测：

```java
Scanner scanner = new Scanner(System.in);
System.out.println("请输入一个整数：");

while (!scanner.hasNextInt()) { // 检查输入是否为整数
    System.out.println("输入无效，请重新输入一个整数：");
    scanner.next(); // 非常重要！消费掉错误的标记，防止陷入死循环
}
int number = scanner.nextInt();
System.out.println("你输入的数字是：" + number);
```

### 4.3  从文件读取实例：以 EOF 控制循环

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class FileScannerExample {
    public static void main(String[] args) {
        File file = new File("data.txt");
        // 使用 try-with-resources 确保资源自动关闭
        try (Scanner scanner = new Scanner(file)) {
            // 当文件还有下一行时，持续循环读取
            while (scanner.hasNextLine()) {
                String line = scanner.nextLine();
                System.out.println(line);
            }
        } catch (FileNotFoundException e) {
            System.err.println("文件未找到: " + e.getMessage());
        }
    }
}
```

## 5  最佳实践与常见问题速查

| 主题 | 常见问题 | ✅ 最佳实践建议 |
| :--- | :--- | :--- |
| **资源管理** | 用完不关，资源泄漏；忘记为 `new Scanner(System.in)` 对象调用 `close()`。 | 对文件等 I/O 源，**务必在 `try-with-resources` 代码块中创建和使用**。**注意：绝不要在项目中关闭包装了 `System.in` 的 `Scanner` 全局实例**，因为它会关闭标准输入流，导致整个程序无法再接收控制台输入。 |
| **数据类型** | 用错 `nextXxx()` 方法，导致 `InputMismatchException`。 | 读取前**总是先用 `hasNextXxx()` 方法进行类型探测**，并做好异常处理。 |
| **行 vs 标记** | `next()` 后调用 `nextLine()`，读不到完整行。 | - **强烈推荐将 `nextLine()` 作为默认的读取方法，然后手动 `split()` 或 `parse`。** <br>- 混用时，记住在 `next()`/`nextInt()` 后加一个额外的 `nextLine()` 来 " 吞掉 " 换行符。 |
| **性能开销** | 在算法竞赛中因读取大量数据而超时。 | - **常规场景下，Scanner 的性能完全足够。**<br>- 对于算法竞赛等**大数据量输入场景，应使用 `BufferedReader` + `StringTokenizer` 的组合**，因为 `Scanner` 内部的正则解析相对较慢【8†L38-L40】。 |
| **字符编码** | 读取非系统默认编码的文件出现乱码。 | 使用 `new Scanner(file, charsetName)` 或 `new Scanner(inputStream, charsetName)` 构造方法，明确指定字符集，如 `StandardCharsets.UTF_8`。 |

## 6  总结与速查表

`Scanner` 是为解析文本而生，而不是为高性能二进制 I/O 而设计。掌握它虽不难，但细节决定成败。

| 特点方面 | 核心要点速查 |
| :--- | :--- |
| **它的本质** | 是一个**文本解析工具 (Text Parser)**，而非 I/O 流。 |
| **核心优势** | 将分词与类型转换融为一体，提供极高**便利性**。 |
| **主要代价** | **性能开销**较大（因大量使用正则），且内部**非线程安全**【8†L38-L40】。 |
| **最经典坑** | `nextInt()` 后直接跟 `nextLine()` 导致后者被跳过。 |

**一句话使用建议**：当你要快速编写中小型程序，实现控制台交互、配置文件解析、格式化文本处理时，首选 `Scanner`。当你要处理大文本、追求极致性能（如竞技编程）或进行字节级 I/O 时，应选择 `BufferedReader` 或 NIO。
