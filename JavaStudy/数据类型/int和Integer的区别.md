## 1  Java 中 `int` 与 `Integer` 的深度解析

### 1.1  一、核心区别对比

  `int` 是 Java 的**基本数据类型**，而 `Integer` 是其对应的**包装类**。二者在本质、存储方式和使用场景上存在显著差异。

| 特性         | `int` (基本类型)                            | `Integer` (包装类)                                           |
| ------------ | ------------------------------------------- | ------------------------------------------------------------ |
| **本质**     | 原始数据类型 (Primitive Type)，**不是对象** | 引用数据类型 (Reference Type)，是 `java.lang.Object` 的子类  |
| **默认值**   | `0`                                         | `null` (未初始化时)                                          |
| **存储位置** | 直接存储数值，通常位于**栈内存 (Stack)**    | 存储对象引用，对象实体位于**堆内存 (Heap)**                  |
| **内存占用** | 固定 4 字节 (32 位)                          | 对象头 + 4 字节数值 + 对齐填充 (通常 16~24 字节)，开销更大    |
| **泛型支持** | **不支持** (如 `List<int>` 非法)            | **支持** (如 `List<Integer>`)                                |
| **方法调用** | 无方法，只能进行算术运算                    | 拥有丰富的方法 (如 `parseInt()`, `toString()`, `compareTo()`) |
| **空值处理** | 不能为 `null`，必须赋值                     | 可以为 `null`，常用于区分 " 值为 0" 和 " 未赋值 "                   |

### 1.2  二、自动装箱与拆箱 (Auto-boxing / Unboxing)

  自 JDK 5 起，Java 引入了自动装箱和拆箱机制，极大地简化了基本类型与包装类之间的转换。

#### 1.2.1  机制原理

  - 自动装箱：将 `int` 自动转换为 `Integer` 对象。

	```java
    Integer a = 10; 
    // 编译器实际执行：Integer a = Integer.valueOf(10);
    ```

  - 自动拆箱：将 `Integer` 对象自动转换为 `int` 数值。

	```java
    int b = a; 
    // 编译器实际执行：int b = a.intValue();
    ```

#### 1.2.2  潜在陷阱

  虽然方便，但过度依赖或不当使用会导致性能问题或空指针异常 (NPE)。

  - **性能损耗**：在循环中频繁装箱拆箱会创建大量临时对象，增加 GC 压力。
  - 空指针风险：如果 `Integer` 对象为 null，进行拆箱操作会直接抛出 NullPointerException 异常。

	```java
    Integer value = null;
    int result = value; // ❌ 运行时抛出 NullPointerException
    ```

### 1.3  三、Integer 的值缓存机制 (Integer Cache)

  为了优化性能，减少频繁创建小整数对象的开销，`Integer` 类内部实现了一个**缓存池**。

#### 1.3.1  缓存范围

  - **默认范围**：**`[-128, 127]`**
  - 行为规则：
	- 当数值在此范围内时，通过 `Integer.valueOf()` (或自动装箱) 获取对象，**返回缓存中已存在的实例**。
- 当数值超出此范围时，**每次都会 `new` 一个新的 `Integer` 对象**。

#### 1.3.2  代码验证

  ```java
  Integer a = 127;
  Integer b = 127;
  System.out.println(a == b); // true (指向同一个缓存对象)
  
  Integer c = 128;
  Integer d = 128;
  System.out.println(c == d); // false (两个不同的新对象)
  
  // ⚠️ 注意：直接使用 new 构造器会绕过缓存，始终创建新对象
  Integer e = new Integer(127);
  Integer f = 127;
  System.out.println(e == f); // false (e 是新对象，f 是缓存对象)
  ```

#### 1.3.3  底层源码逻辑

  `Integer.valueOf(int i)` 方法的简易逻辑如下：

  ```java
  public static Integer valueOf(int i) {
      // 判断是否在缓存范围内 (-128 到 high，high 默认为 127)
      if (i >= IntegerCache.low && i <= IntegerCache.high)
          return IntegerCache.cache[i + (-IntegerCache.low)];
      // 超出范围，返回新对象
      return new Integer(i);
  }
  ```

#### 1.3.4  扩展知识：自定义缓存范围

  可以通过 JVM 启动参数 `-XX:AutoBoxCacheMax=<size>` 来调整缓存的上限（下限 -128 固定）。

  - 例如：`-XX:AutoBoxCacheMax=200` 会将缓存范围扩大到 `[-128, 200]`。
  - **应用场景**：如果业务中大量使用 128~500 之间的整数作为对象键值，调大此参数可减少对象创建，提升性能。

### 1.4  四、总结

  1. **优先使用基本类型**：

	 - 如果没有特殊需求（如需要 `null` 值、泛型集合、反射等），**始终优先使用 `int`**。它更高效、更省内存且无 NPE 风险。

  2. **慎用 `'=='` 比较包装类**：

	 - **错误**：`if (integerA == integerB)` (受缓存影响，结果不可靠)
	 - **正确**：`if (integerA.equals(integerB))` 或 `if (integerA.compareTo(integerB) == 0)`
	 - _例外_：如果你明确知道数值在 `[-128, 127]` 之间且确定使用的是缓存对象，`==` 才有效，但为了代码健壮性，**永远推荐用 `equals()`**。

  3. **警惕空指针**：

	 - 从数据库或接口获取的 `Integer` 可能为 `null`，在进行数学运算或转 `int` 前，务必进行非空检查。

	 ```java
     // 安全写法
     int value = (myInteger != null) ? myInteger : 0;
     ```

  4. **理解缓存边界**：

	 - 在涉及对象身份比较（如用作 Map 的 Key 或 Set 的元素去重）时，理解缓存机制有助于排查 " 明明值相等却找不到对象 " 的诡异 Bug（通常是因为混用了 `new Integer()` 和自动装箱）。

  > **一句话总结**：`int` 是高效的数值，`Integer` 是功能丰富的对象；利用自动装箱时要小心 `null` 和 `==` 比较，并牢记 `[-128, 127]` 的缓存魔法。
