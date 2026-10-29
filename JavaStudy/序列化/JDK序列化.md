## 1  概念

### 1.1  定义

- **序列化**：将对象的状态信息及类型元数据转换为可存储或传输的**有序字节序列**的过程。
- **反序列化**：将字节序列依据其内部结构描述重新还原为内存中的**对象实例**的过程。

### 1.2  典型应用场景

- **持久化存储**：将对象状态保存至本地磁盘、数据库或分布式缓存。
- **网络传输**：在分布式系统的不同节点间传递对象副本（如 RPC 参数与返回值）。
- **深拷贝实现**：通过序列化与反序列化生成一个与原对象状态完全一致但无引用关系的新对象。
- **Java RMI**：远程方法调用中参数与结果的跨 JVM 传递。

## 2  JDK 原生序列化机制

### 2.1  序列化

JDK 通过 `java.io.ObjectOutputStream` 与 `java.io.ObjectInputStream` 提供内置序列化支持。

序列化示例

```java
private static void serialize(String filename) throws IOException {
	// 1. 设置保存路径
	File f = new File(filename);
	// 2. 文件输出流
    OutputStream out = new FileOutputStream(f);
    // 3. 对象输出流
    ObjectOutputStream oos = new ObjectOutputStream(out);
    // 4. 序列化
    oos.writeObject(new Person("Jack", 30, Person.Sex.MALE));
    // 5. 关闭对象输出流、文件输出流
    oos.close();
    out.close();
}
```

反序列化示例

```java
private static Object deserialize(String filename) throws IOException, ClassNotFoundException {
	// 1. 设置保存路径
    File f = new File(filename);
    // 2. 文件输入流
    InputStream in = new FileInputStream(f);
    // 3. 对象输入流
    ObjectInputStream ois = new ObjectInputStream(in);
    // 4. 反序列化
    Object obj = ois.readObject();
    // 6. 关闭输入流
    ois.close();
    in.close();
    return obj;
}
```

### 2.2  序列化过程

序列化是**将内存中的对象状态转换为可存储或传输的二进制字节流的过程**。为保证反序列化时能够精确、完整地重建对象，序列化过程会在输出字节流中嵌入结构标记与元数据。

#### 2.2.1  序列化机制概述

序列化过程的核心操作包括：

- **对象数据提取**：读取对象的类结构、字段类型及字段值。
- **二进制编码转换**：将上述信息按特定协议转换为二进制格式。
- **结构标记插入**：在字节流中植入用于界定数据边界与类型的**特殊分隔符**或长度前缀。

特殊分隔符在具体协议中表现形式不同。

- **Java 原生序列化**：使用 `TC_OBJECT`、`TC_STRING` 等魔术字节（Magic Bytes）作为控制字。
- **Protocol Buffers / Thrift**：使用字段编号（Field Number）与 Wire Type 进行**标签 - 长度 - 值**编码，不依赖纯粹的分隔符截断。

#### 2.2.2  序列化二进制数据

序列化生成的二进制字节流通常由以下三部分逻辑层构成：

| 结构部分         | 核心内容与作用    | 专业表述细节                                                                                                                            |
| :----------- | :--------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| **头部数据**     | 协议标识与版本控制  | 包含**魔术数字**（用于识别协议类型，如 `0xACED` 标识 Java 序列化流）及**版本号**。用于确保反序列化器能够识别流格式，实现向后或向前兼容。                                                  |
| **对象元数据与载荷** | 类描述信息与实例状态 | 具体包括：  <br>- **类全限定名**  <br>- **序列化版本唯一标识符**（`serialVersionUID` 或类似签名）  <br>- **字段元信息**（名称、类型、修饰符）  <br>- **字段实际值**（按类型编码后的二进制数据） |
| **递归处理逻辑**   | 引用管理与继承链解析 | 当对象图中存在**循环引用**或**多态继承**时，序列化算法将递归遍历：  <br>- **对象引用处理**：使用句柄机制避免重复序列化同一对象。  <br>- **父类状态处理**：自顶向下遍历继承链，依次序列化各级父类定义的数据。            |

## 3  Serializable 接口

### 3.1  可序列化类

被序列化的类必须直接或间接实现 `java.io.Serializable` 接口，或是 `Enum` 或数组类型，否则将抛出 `NotSerializableException`。

- 若父类实现了 `Serializable`，则所有子类均自动获得序列化能力。
- 子类可序列化而父类不可序列化时，反序列化将调用父类的**无参构造器**初始化父类状态。

### 3.2  serialVersionUID

- `serialVersionUID` 是序列化运行时用于验证类定义一致性的流唯一标识符。
- 该字段必须显式声明为 `static final long` 字段。
- 它可以用来保证在反序列时，发送方发送的和接受方接收的是可兼容的对象。如果接收方接收的类的 `serialVersionUID` 与发送方发送的 `serialVersionUID` 不一致，会抛出 `InvalidClassException`。

> [!suggestion] 显式声明 serialVersionUID
> - 如果在可序列化类中没有显式声明 `serialVersionUID`，在进行序列化和反序列化时，JDK 也会计算默认值。
> - 但是，建议显式指定，避免因不同编译器生成的默认值差异导致反序列化失败。

**兼容策略**：
- **保持向后兼容**：类结构发生非破坏性变更（如添加非 `transient` 字段、新增方法）时，维持 `serialVersionUID` 不变。
- **强制版本隔离**：当修改导致序列化布局不兼容（如删除字段、修改继承层级）时，应更新版本号以触发 `InvalidClassException`，防止数据错乱。

### 3.3  默认序列化

- 当可序列化类实现了 `Serializable` 接口后，如果没有其他额外的处理，那么就会使用默认序列化机制。
- 默认的序列化机制将递归遍历对象的整个可达对象图，包括所有没有被 `transient` 关键字修饰的字段引用的对象。（父类以及成员中的对象引用，然后递归）

> [!warning]
> 注意：若对象图包含深层嵌套容器（如 List<Map<String, List<…>>>），序列化将产生巨大的 CPU 与 I/O 开销。

### 3.4  transient 关键字

- 被 `transient` 修饰的实例字段将**被默认序列化机制完全忽略**，反序列化后该字段被置为类型默认值
- **适用场景**：派生数据、缓存句柄、安全敏感信息（如明文密码）。

```java
transient private Integer age = null; // transient
```

## 4  Externalizable 接口

- Externalizable 接口是另一个可序列化接口，该接口继承了 `Serializable` 接口。
- **默认序列化机制失效**：如果可序列化类实现了 Externalizable 接口，那么该类将不再适用默认序列化机制（基于 Serializable 接口）。
- 可序列化类必须提供一个 `public` 无参构造方法。因为反序列化流程会先通过该构造器创建**空实例**，再调用 `readExternal` 回填字段值。
- 实现该类需要重写 writeExternal 和 readExternal 方法。这两个方法会在序列化和反序列化的过程中被自动调用。

```java
@Override
public void writeExternal(ObjectOutput objectOutput) throws IOException {
    objectOutput.writeObject(name);
    objectOutput.writeInt(age);
    objectOutput.writeObject(sex);
}

@Override
public void readExternal(ObjectInput objectInput) throws IOException, ClassNotFoundException {
    name = (String) objectInput.readObject();
    age = objectInput.readInt();
    sex = (Person.Sex) objectInput.readObject();
}
```

## 5  writeObject / readObject 方法

- 在可序列化类（实现 `Serializable` 接口）内，添加 `writeObject` + `readObject` 方法，是实现 `Externalizable` 的等价方案。这两个方法会在序列化和反序列化的过程中被自动调用。
- 两个方法必须严格遵循以下的格式
	- `private void writeObject(ObjectOutputStream out)`
	- `private void readObject(ObjectInputStream in)`
- 在两个方法内，通常先调用 `out.defaultWriteObject()` 或 `in.defaultReadObject()` 以执行默认序列化,再对 `transient` 字段进行自定义读写。

```java
@Serial
private void writeObject(ObjectOutputStream out) throws IOException {
    out.defaultWriteObject(); // 1. 执行默认序列化
    out.writeInt(age); // 2. 对 transient 字段进行读入
}

@Serial
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject();
    age = in.readInt();
}
```

> [!question] 为什么这两个函数是 `private` ，但是仍然能被自动调用？
> ObjectOutputStream 调用的 `writeSerialData` 和 ObjectInputStream 调用的 `readSerialData` 方法通过反射查找并调用这两个私有方法，因此访问修饰符不影响调用。

## 6  readResolve / writeReplace 方法

- **`writeReplace`**：序列化前由 JVM 调用，允许将当前对象替换为另一个**代理对象**写入流中，常用于保护单例或实现序列化代理模式。
- **`readResolve`**：反序列化后调用，返回的对象将**取代**刚刚从流中读取的对象，用于确保反序列化后仍能维持单例、枚举常量等不变量。

为了使得反序列化仍能维持单例，在可序列化类中添加 readResolve 方法

```java
class Person implements Serializable{

	// 略
	
	static final Person instance = new Person("Tom", 31, SerializeSingleton.Sex.MALE);
	
    public static Person getInstance() {
        return instance;
    }

	// 如果不添加该函数，那么在序列化后，获得的新的对象和序列化前的对象其实是两个对象
	// 也就是说，打破了单例模式
	@Serial
    private Object readResolve(){
        return instance;
    }
}

```

## 7  JDK 序列化的问题

- **无法跨语言**：JDK 序列化目前只适用基于 Java 语言实现的框架，其它语言大部分都没有使用 Java 的序列化框架，也没有实现 JDK 序列化这套协议。因此，如果是两个基于不同语言编写的应用程序相互通信，则无法实现两个应用服务之间传输对象的序列化与反序列化。
- **容易被攻击**：
	- 对象是通过在 `ObjectInputStream` 上调用 `readObject()` 方法进行反序列化的，它可以**将类路径上几乎所有实现了 `Serializable` 接口的对象都实例化**。这意味着，在反序列化字节流的过程中，该方法可以执行任意类型的代码，这是非常危险的。
	- 对于需要长时间进行反序列化的对象，不需要执行任何代码，也可以发起一次攻击。攻击者可以**创建循环对象链**，然后将序列化后的对象传输到程序中反序列化，这种情况会导致 `hashCode` 方法被调用次数呈次方爆发式增长, 从而引发栈溢出异常。例如下面这个案例就可以很好地说明。
- **序列化后的流太大**：JDK 序列化中使用了 `ObjectOutputStream` 来实现对象转二进制编码，编码后的数组很大，非常影响存储和传输效率。
- **序列化性能太差**：Java 的序列化耗时比较大。序列化的速度也是体现序列化性能的重要指标，如果序列化的速度慢，就会影响网络通信的效率，从而增加系统的响应时间。
- **序列化编程限制**：
    - JDK 序列化**一定要实现 `Serializable` 接口**。
    - JDK 序列化**需要关注 `serialVersionUID`**。
