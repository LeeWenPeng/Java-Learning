## 1  不可变性

### 1.1  不可变性的来源

```java
public final class String implements Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc {
    @Stable
    private final byte[] value;
```

- `String` 类被 `final` 修饰，表示不可继承 `String` 类
- `value` 数组用于存储数据，被 `final` 修饰，表示不可修改

### 1.2  不可变性的优势

1. 保证 `String` 对象的安全性，避免 `String` 对象被篡改，比如加载类的全限定名
2. 保证 hash 值不会频繁更改，可以作为 HashMap 的键
3. 可以实现字符串常量池。

## 2  字符串常量池

在 Java 中，`String` 对象的创建主要有两种方式

1. 通过字符串字面量（常量）直接赋值；
2. 通过 `new` 关键字显式构造。

### 2.1  通过字符串字面量创建

当使用形如 `String s = "abc";` 的字符串字面量赋值时，JVM 遵循以下流程：

1. **常量池检查**  
   类加载阶段，该字面量对应的 UTF-8 编码字符串会被解析并存放在 **运行时常量池（Runtime Constant Pool）** 中。执行赋值指令时，JVM 会先检查字符串常量池（String Pool / String Table）中是否已存在内容相等的字符串实例。

2. **返回现有引用**  
   若常量池中已存在内容相同的字符串对象，则直接返回该对象的引用，不再创建新对象。

3. **创建并驻留**  
   若常量池中不存在匹配的字符串，则在堆中创建一个新的 `String` 实例，并将其引用存入字符串常量池（即 " 驻留 " 或 " 内部化 "），随后返回该引用。
	1. `String` 实例在 Java 堆中
	2. 其引用位于常量池中
	3. 常量池位于堆中

> [!note] 字符串常量池所在位置
> 1. JDK 7 以前：永久代（PermGen）
> 2. JDK 7：Java 堆（Heap）中
> 3. JDK 8 以及以后：元空间（Metaspace）之外的堆区

### 2.2  通过 `new` 关键字创建

当使用 `String s = new String("abc");` 显式构造时，对象创建过程更为复杂，且涉及多个内存区域：

1. **编译期常量处理**  
   编译阶段：代码中的字符串字面量参数（`"abc"`）被解析并存入 `.class` 文件的常量池（Constant Pool）结构。
2. **类加载期驻留**  
   类加载过程中，JVM 会在运行时常量池（字符串常量池）中检查该字面量是否已存在。若不存在，则先在堆中创建一个对应的 `String` 对象并驻留到常量池；若已存在，则直接沿用该驻留引用。
3. **堆对象构造**  
   执行 `new` 指令时，JVM 会在堆内存中独立创建一个全新的 `String` 实例，其内部 `value` 字符数组将引用常量池中已存在的那个字符串对象的 `value` 数组（即字符数据共享，但外层的 `String` 容器是独立的）。
4. **返回新对象引用**  
   最终将堆中新构造的 `String` 对象的引用返回给变量。

由此可见，`new String("abc")` 在常量池已包含 `"abc"` 的情况下，**仍会在堆中额外创建一个 `String` 对象**，而字面量赋值则直接复用常量池中的同一对象。这也是为何通常推荐使用字面量创建字符串以节省内存开销的原因。

![String常量池](img/String的不可变性/String常量池.md)

### 2.3  字符串驻留（`intern()`）方法

`String` 类提供了 `intern()` 方法，允许开发者主动将一个堆中的 `String` 对象 " 手动 " 添加到常量池（若池中无等价内容）或返回池中已存在的等价对象引用：

```java
String s1 = new String("hello");
String s2 = s1.intern();   // 返回常量池中的 "hello" 引用
String s3 = "hello";       // 直接复用常量池中的同一引用
System.out.println(s2 == s3); // true
```

- **行为**：调用 `intern()` 时，若常量池中已存在内容相同的字符串，则直接返回该驻留引用；否则，将当前 `String` 对象自身（或其副本）的引用存入常量池并返回该引用。
- **使用场景**：适用于需要大量字符串内容比较且重复率较高的场景，通过驻留使内容相同的字符串共享同一对象，从而减少内存占用并加速 `==` 比较。

> [!note] 别轻易使用 intern()
> 因为常量池内是类似于 hashmap 的结构，如果其中存储的数据越大，遍历的时间复杂度就会变大。

## 3  String 的性能

### 3.1  字符串的拼接

- 字符串常量的拼接，编译器会将其优化成一个常量字符串

```java
package Object.StringObject;
public class Test {
    public static void main(String[] args) {
        // 本行代码会被优化为
        // String str = "abc";
        String str = "a" + "b" + "c";
        System.out.println("str = " + str);
    }
}
```

- 字符串变量的拼接，编译器会将其优化为 `StringBuilder` 的方式进行字符串拼接。如果这个拼接操作在循环内，那么每一次，都需要 new 一个新的 `StringBuilder` 对象进行字符串的拼接

```java
package Object.StringObject;
public class Test {
    public static void main(String[] args) {
        str = "";
        for (int i = 0; i < 10; i++) {
            // 本行代码会被优化为
            // str = (new StringBuilder(String.valueOf(str))).append(i).toString();
            str += i;
            // 每次循环都会 new 一个新的 StringBuilder
        }
    }
}
```

字符串拼接的最佳方案

- 单行拼接直接使用 `+` 号，编译器会帮助优化。
- 循环内的字符拼接，使用 `StringBuilder`，避免多次创建 `StringBuilder` 和 `String` 对象。
- 在并发编程时，考虑到线程安全，使用 `StringBuffer`。

### 3.2  字符串分割

字符串分割有两个方式

- `split()` 方法。依靠正则表达式进行分割，但是正则表达式的性能不稳定，使用不恰当时，会引起回溯问题，导致 CPU 占有率居高不下。
- 应该优先考虑使用 `String.indexOf()` + `subString()` 进行字符串分割。

#### 3.2.1  为什么 `split()` 的正则回溯会有问题？

`String.split(String regex)` 内部使用 `Pattern.compile(regex)`，如果正则表达式包含量词（`*`、`+`、`?`、`{n,m}`）且组合不当，匹配失败时会进行**指数级回溯**。

**典型高风险例子：**

```java
// 恶意长字符串 + 缺陷正则

String bad = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaab";

bad.split("(a+)+b"); // 回溯次数爆炸，CPU 100%
```

即使简单如 `split("\\|")`，因为每次调用都会重新编译正则对象，在循环中频繁使用也会产生不必要的开销。

## 4  StringBuffer

StringBuffer 是为了解决字符串拼接产生类太多中间对象的问题而提供的一个类。 `StringBuffer` 是一个线程安全的可修改字符序列，其线程安全性是通过在各种修改数据的方法上添加 `synchronized` 关键字实现的。

## 5  StringBuilder

StringBuilder 是 Java 1.5 中新增的 API，其和 StringBuffer 二者 API 几乎一致。唯一的区别在于修改数据的方法上，**StringBuffer 加了 `synchronized` 关键字，而 StringBuilder 没有**。

> [!suggestion] 使用建议
> 除非明确要求线程安全性，否则应一律使用 `StringBuilder`，因为它去除了不必要的锁开销，性能更高。

## 6  StringBuilder 的底层实现与扩容机制

StringBuilder 和 StringBuffer 的底层是通过可修改的字符数组 (JDK 9 以后为 `byte[]`, 配合编码标识 `coder`) 对字符串进行保存，可以通过 append 或 insert 方法把字符串添加到已有序列的末尾或指定位置。

初始容量

- 无参构造：初始容量为 16
- 有参构造 `StringBuilder(String str)`：初始容量为 `str.length() + 16`
- 有参构造 `StringBuilder(int capacity)`：初始容量为 `capacity`

扩容算法：

1. 计算新容量尝试值：$新容量 = 旧容量 * 2 + 2$
2. 若计算出的新容量仍然小于所需的最小容量，那么以所需的最小容量作为新容量：$新容量 = max(新容量, 所需最小容量)$
3. 创建新数组，并将原数组中数据复制到新数组 (底层调用 `System.arraycopy`)，丢弃原数组。

> [!Suggestion] 性能优化建议
> 如果能提前预估最终字符串的长度，**务必使用带初始容量的构造器**，以避免多次扩容带来的内存分配与复制开销。这种优化在拼接大量字符串（如生成报表、拼接长 JSON）时效果极为显著。

## 7  String 的题目

```java
public class Test {
    public static void main(String[] args) {
        String str1 = "A";
        String str2 = null;
        System.out.println(str1);
        // null 无法打印，因为 null 可能是 char[] null，也可能是 String null
//        System.out.println(null);
		// null 无法调用 toString
//        System.out.println(str2.toString());
		// 这里的 str2 被优化为 (String) null，这里 null 为 String 类型字符串
        System.out.println(str2); // 打印结果： null
        // 这里被优化为 StringBuilder 调用 append
        // 其中,StringBuilder 对象 append(str2)，由于 str2 == null
        // 因此会调用 appendNull，等价于 append("null")
        System.out.println(str1 + str2); // 打印结果 Anull
    }
}
```
