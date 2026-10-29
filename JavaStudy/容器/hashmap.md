在 Java 中，`HashMap` 是最常用的集合类之一，用于存储键值对（Key-Value pairs）。它基于哈希表实现，允许使用 `null` 作为键或值，且**不保证映射的顺序**。

以下是 `HashMap` 的核心用法指南，包括引入、创建、增删改查以及遍历。

### 1. 引入包
首先需要导入 `java.util.HashMap`：
```java
import java.util.HashMap;
import java.util.Map; // 推荐引用接口类型
```

### 2. 创建 HashMap
通常建议使用接口 `Map` 来声明变量，这样代码更灵活：
```java
// 语法：Map<Key类型, Value类型> map = new HashMap<>();
Map<String, Integer> scores = new HashMap<>();
```

### 3. 基本操作

#### A. 添加元素 (put)
使用 `put(key, value)` 方法。如果键已存在，新值会覆盖旧值。
```java
scores.put("Alice", 95);
scores.put("Bob", 88);
scores.put("Charlie", 92);

// 覆盖操作：如果 "Alice" 已存在，分数将更新为 100
scores.put("Alice", 100); 
```

#### B. 获取元素 (get)
使用 `get(key)` 方法。如果键不存在，返回 `null`（对于对象类型）或默认值。
```java
Integer aliceScore = scores.get("Alice"); // 返回 100
Integer unknownScore = scores.get("David"); // 返回 null
```

#### C. 检查是否存在 (containsKey / containsValue)
```java
boolean hasBob = scores.containsKey("Bob"); // true
boolean hasScore90 = scores.containsValue(90); // false (因为没有正好90分的)
```

#### D. 删除元素 (remove)
```java
scores.remove("Charlie"); // 删除键为 "Charlie" 的条目
```

#### E. 获取大小 (size)
```java
int size = scores.size(); // 返回剩余元素个数
```

### 4. 遍历 HashMap (三种常用方式)

#### 方式一：使用 entrySet (推荐，效率最高)
如果你需要同时访问键和值，这是最佳方式。
```java
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + ": " + value);
}
```

#### 方式二：使用 keySet
如果你只需要键，或者需要通过键去获取值（稍微慢一点，因为多了一次 get 操作）。
```java
for (String key : scores.keySet()) {
    System.out.println(key + ": " + scores.get(key));
}
```

#### 方式三：使用 forEach (Java 8+, 最简洁)
适合简单的打印或操作。
```java
scores.forEach((key, value) -> {
    System.out.println(key + " => " + value);
});
```

### 5. 完整示例代码

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapExample {
    public static void main(String[] args) {
        // 1. 创建
        Map<String, String> userMap = new HashMap<>();

        // 2. 添加
        userMap.put("u001", "张三");
        userMap.put("u002", "李四");
        userMap.put("u003", "王五");

        // 3. 修改 (如果键存在则覆盖)
        userMap.put("u001", "张三丰"); 

        // 4. 获取
        String name = userMap.get("u002");
        System.out.println("用户 u002 是: " + name); // 输出: 李四

        // 5. 判断
        if (userMap.containsKey("u003")) {
            System.out.println("用户 u003 存在");
        }

        // 6. 遍历
        System.out.println("--- 所有用户 ---");
        for (Map.Entry<String, String> entry : userMap.entrySet()) {
            System.out.println("ID: " + entry.getKey() + ", 姓名: " + entry.getValue());
        }

        // 7. 删除
        userMap.remove("u003");
        System.out.println("删除后大小: " + userMap.size());
    }
}
```

### 6. 重要注意事项

1.  **线程不安全**：`HashMap` 不是线程安全的。如果在多线程环境下使用，请考虑：
    *   使用 `ConcurrentHashMap`（推荐，性能好）。
    *   使用 `Collections.synchronizedMap(new HashMap<>())`。
2.  **Key 的要求**：
    *   作为 Key 的对象必须正确重写 `hashCode()` 和 `equals()` 方法，否则可能导致无法获取到存入的值或出现重复键。
    *   推荐使用不可变类（如 `String`, `Integer`）作为 Key。
3.  **顺序问题**：`HashMap` **不保证**插入顺序。如果需要保持插入顺序，请使用 `LinkedHashMap`；如果需要按 Key 排序，请使用 `TreeMap`。
4.  **Null 值**：`HashMap` 允许一个 `null` 键和多个 `null` 值。

如果你有关于特定场景（如自定义对象作为 Key）的问题，欢迎继续提问！