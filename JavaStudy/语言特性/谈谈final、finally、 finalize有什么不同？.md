## final

final 用于声明“不可变性”的关键字，通过限制变量、方法、类的修改能力，为代码提供安全性、线程安全基础。

### final 修饰变量——保证变量本身的值“不可变”

- 基础类型：值本身不可变

- 引用类型：引用地址不可变，也就是引用的指向不能变，但对象的内部状态可以修改

  ```java
  final List list = new ArrayList(); // 定义 final 修饰的引用对象
  // list = new ArrayList();  // 编译报错，引用地址不可变
  list.add("x"); // 成功
  ```

#### 关键约束

- 被 final 修饰的常量，必须显式初始化

  - 局部变量：在声明时或在首次使用前必须赋值，且只能赋值一次
  - 成员变量：必须在声明时、或构造方法中、或实例初始化块中完成赋值
- 与常量区别
  - `static final` 是类级常量
  - `final` 单独修饰是实例级常量



### final 修饰方法——防止逻辑被篡改

- 禁止子类重写：确保核心逻辑不被覆盖，常用于：
  - 安全校验方法
  - 模板方法模式中的骨架方法

- 明确设计意图：向其他开发者声明“此方法不应被修改”

### final 修饰类——彻底封死继承链



使用场景

- 工具类：无需扩展
- 安全敏感类：防止篡改
- 核心业务实体：防止破坏

特性：

- final 类中的所有方法默认 final，但是可以显示声明非 final 方法
- 与单例模式无关：单例模式通过私有构造器实现
  - 单例模式要求限制的是实例化过程
  - 而 final 关键字限制的是类的继承
- 线程可见性：若对象所有的字段都为 final ，且构造器未发生 this 溢出，则该对象对其他线程天然可见
  - 但 final 不保证原子性

#### final 实现 immutable 类

final 修饰引用型对象时，只保证引用的不可变性。因此，如果希望对象本身不可变，则需要保证

- 将 class 声明为  final，防止子类继承并覆盖父类方法，破坏不可变性
- 将所有成员变量定义为 private 和 final，并且不要实现 setter 方法
- 构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值
  - 如果直接赋值，比如引用类型成员，外部可以通过原引用来修改内部状态
- 如果确实需要实现 getter 方法，或者其他可能会返回内部状态的方法，使用 copy-on-write 原则，创建私有 copy。
  - 防止外部通过 getter 方法获取引用直接修改内部状态

## finally

finally 是Java异常处理机制中用于确保关键代码（如资源清理）无论是否发生异常都会执行的代码块



设计目的：主要用于资源处理（比如关闭文件、释放锁、重置状态等），确保程序在任何情况下都能正确释放资源，防止资源泄露



执行时机：在 try/catch 之后，方法返回之前



注意：需要避免在其中使用 return 或抛出异常，以防覆盖原返回值或异常



**执行失败场景**：在`System.exit()`、JVM崩溃、线程强制中断、死循环等极端情况下，finally块可能不执行2。



```java
try{
    System.exit(1);
}finally{
    // todo...
}
```

- 上述 finally 中的代码将不会被执行



### 现代替代方案 try-with-resources

Java 7 引入了 try-with-resources 语法，它可以自动关闭实现了 `AutoCloseable` 或 `Closeable` 的资源

#### 传统资源清理方式

```java
FileInputStream fis = null;
try{
    fis = new FileInputStream("data.txt");
    // 读取文件操作 todo ... 
}catch(IOException e){
    // 处理异常 todo ...
}finally{
    if(fis != null){
        try{
            fis.close();
        }catch(IOException ignored){
            // 关闭异常
            // 通常忽略或记录为 warn
        }
    }
}
```

#### 现代替代方案：try-with-resources

```java
try(FileInputStream fis = new FileInputStream("data.txt")){
    // 读取文件操作 todo ...
}catch(IOException e){
    // 处理异常
}

// fis.close() 自动在 finally 中调用
```

- 资源需要实现 AutoCloseable 或 Closeable 接口
- 上述代码中 FileInputStream 继承自 InputStream，而 InputStream 实现了 Closeable 接口

## finalize

finalize 是基础类 java.lang.Object 中的一个方法，设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制已经不被推荐使用，在 Java 9 中将其标注为 deprecated。



> 为什么不推荐使用 finalize 机制
>
> 因为无法保证finalize 什么时候执行，执行是否符合预期。使用不当会影响性能，导致程序死锁、挂起等。