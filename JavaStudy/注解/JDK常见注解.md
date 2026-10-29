## 1  `@Override` 注解

### 1.1  注解定义与元信息

`@Override` 的源码定义如下（基于 JDK 17）：

```java
package java.lang;

import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

> [!question] 为什么生命周期是 `SOURCE`？  
> 因为 `@Override` 的唯一目的是在**编译期**进行语法校验，一旦校验通过，它的使命就完成了。将其保留至运行时毫无意义，反而会占用字节码空间。

### 1.2  核心作用：编译时重写校验

`@Override` 通知编译器：**" 被修饰的方法意图重写父类或接口中的方法声明。"** 编译器会据此执行严格的检查：

- **检查签名一致性**：方法名、参数列表、返回类型必须与被重写方法**完全一致**（返回类型允许协变）。
- **检查可见性**：子类重写方法的访问权限不能比父类方法更严格（例如父类方法是 `protected`，子类不能重写为 `private`）。
- **检查静态与非静态**：实例方法不能重写静态方法，反之亦然。
- **检查异常声明**：重写方法抛出的受检异常不能比父类方法更宽泛。

如果上述任何条件不满足，编译器会**立即报错**，阻止生成不正确的类文件。

### 1.3  为什么必须使用 `@Override`？

即使不使用 `@Override`，只要方法签名匹配，Java 仍然会正常完成重写（Override）行为。那么为什么还要显式添加？

| 对比项 | 使用 `@Override` | 不使用 `@Override` |
| :--- | :--- | :--- |
| **意图表达** | 清晰表明这是重写方法，增强可读性。 | 容易误以为是新增方法，阅读代码时需要额外判断。 |
| **编译期保护** | **严格校验签名**，签名不匹配立即报错。 | 若签名写错，编译器会将其视为**新方法**，不会报错，导致运行时 Bug 难以定位。 |
| **重构安全** | 父类方法签名变更后，子类 `@Override` 方法若未同步修改会直接编译失败，强制开发者处理。 | 父类方法改名后，子类旧方法悄然成为 " 孤儿 " 普通方法，逻辑断裂但编译通过。 |

## 2  `@Deprecated` 注解

注释一个方法，描述该方法为过时、已经被弃用

### 2.1  作用

- **标记状态**：声明某个 API 已进入 " 废弃 " 或 " 过时 " 状态。
- **非阻断性**：被标记的代码仍可正常编译和运行，但会触发编译警告。
- **演进辅助**：给 API 的使用者预留时间迁移到新的替代方案。

> [!note] 可以与 Javadoc `@deprecated` 标签配合使用
> 注解用于让编译器发出警告，Javadoc 标签用于解释废弃原因及迁移指南。
>
>```java
>/**
> * 旧版计算器
> * @deprecated 自 2.0 版本起，请使用 {@link NewCalculator} 替代。
> * */
> @Deprecated
> public class OldCalculator {
> 	// …
> }
> ```

### 2.2  语法与使用方式

#### 2.2.1  基本语法

```java
@Deprecated
要注解的元素(几乎所有 Java 元素)

/**
* @deprecated 自 2.5 版本起弃用，将在 3.0 版本中移除。请使用 {@link [#newMethod](#newMethod)()}。
*/
@Deprecated(since = "2.5", forRemoval = true) 
public void legacyMethod() { }
```

- `{@link}` 标签：指向新的 API

#### 2.2.2  修饰范围

`@Deprecated` 可以修饰几乎所有 Java 元素：

- 类、接口、枚举 (`ElementType.TYPE`)
- 方法 (`ElementType.METHOD`)
- 字段（包括枚举常量）(`ElementType.FIELD`)
- 构造方法 (`ElementType.CONSTRUCTOR`)
- 注解类型 (`ElementType.ANNOTATION_TYPE`)
- 包 (`ElementType.PACKAGE`)（通过 `package-info.java` 声明）
- 模块 (`ElementType.MODULE`)（Java 9+ 的 `module-info.java`）

#### 2.2.3  增强属性

从 Java 9 开始，`@Deprecated` 增加了两个可选属性，使其语义更加精确。

| 属性 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| `since` | `String` | `""` | 标记该 API 从哪个版本开始被弃用 |
| `forRemoval` | `boolean` | `false` | 标识该 API 是否计划在未来的版本中被**彻底移除** |

- 当 `forRemoval = true` 时，某些编译器（如 JDK 9+ 的 `javac`）会发出**更强烈的警告**（如 `removal warning`），提醒开发者该 API 即将消失，必须尽快迁移。

## 3  `@SuppressWarnings` 注解

### 3.1  作用

`@SuppressWarnings` 是 Java 内置的一个编译时注解，用于指示编译器**抑制（不报告）**指定的编译警告。

### 3.2  语法

```java
// 抑制指定警告
@SuppressWarnings(String arg)
要被注解的元素(类或方法)

// 如果需要同时抑制多种警告，可以使用花括号包裹的字符串数组
@SuppressWarnings({String arg1, arg2}) 
要被注解的元素(类或方法)
```

- 参数：单个字符串或字符串数组，代表要忽略的警告类型。
	- 不区分大小写，但建议使用小写连字符
	- 常见参数值包括：

| 参数值 | 含义 |
| :--- | :--- |
| `"deprecation"` | 忽略**使用了已过时（`@Deprecated`）API**的警告。 |
| `"unchecked"` | 忽略**未经检查的类型转换**警告（常见于泛型集合）。 |
| `"rawtypes"` | 忽略**使用了原始类型**（如 `List` 而非 `List<String>`）的警告。 |
| `"unused"` | 忽略**未使用的变量或方法**警告。 |
| `"all"` | 忽略**所有**警告。 |

### 3.3  注解的粒度与范围

`@SuppressWarnings` 可以修饰不同级别的元素：

```java
// 1. 抑制整个类的警告
@SuppressWarnings("deprecation")
public class Test { … }

// 2. 抑制某个方法的警告
@SuppressWarnings("deprecation")
public static void main(String[] args) { … }
```

> [!tip] 最佳实践
> 尽可能将注解放在**最小作用域**内（如方法级别），避免因为大范围抑制而掩盖了其他本应关注的潜在问题。

## 4  `@SafeVarargs`

`@SafeVarargs` 是 JDK 7 引入的一个注解，其核心作用是**抑制编译器关于 " 泛型可变参数 " 可能引发的堆污染（Heap Pollution）警告**。它向编译器和调用者承诺：该方法在处理泛型可变参数时，不会进行任何不安全的操作，因而是类型安全的。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
public @interface SafeVarargs {
}
```

### 4.1  `@SafeVarargs` 注解使用范围

并非所有方法都能使用此注解，其适用范围有严格限制，只能标注在**真正无法被覆盖重写的方法**上：

- **构造方法** （Constructor）
- **静态方法** （`static` method）
- **最终方法** （`final` method）
- **私有方法** （`private` method） （>= JDK9）。

## 5  `@FunctionalInterface`

JDK8 引入该注解。用于指示被修饰的接口是函数式接口。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface FunctionalInterface {
}
```

- 类似于 Override 对于重写方法的作用，@FunctionalInterface 也用于指示函数式接口

### 5.1  什么是函数式接口

**函数式接口 (Functional Interface) 就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口**。函数式接口可以被隐式转换为 lambda 表达式。

特点

- 接口有且仅有一个出现方法
- 不能在接口内重写 Object 类中的 public 方法
- 允许有 default 方法
