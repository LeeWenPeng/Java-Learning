## 1  Object：所有类的超类

- Object 是Java中所有类的祖先类，无论是否显式继承。

- 在Java中，只有基础类型不是对象，其他所有数据类型都是对象。因此，他们也都是 Object 的子类

- 可以使用 Object 类型的变量引用任何类型的对象

  ```java
  Object obj = new ClassName(args);
  ```

- 但如果想使用，则需要清楚对象的原始类型，并将 Object 类型变量向下转型为具体的原始类型。

  ```java
  ClassName cla = (ClassName) obj;
  ```

## 2  Equals 方法

### 2.1  默认实现

Object 类中默认的 equals 方法是通过 `==` 运算符比较两个对象的引用

```java
public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }

```



### 2.2  特性

- 自反性，对于任何非空引用 x，`x.equals(x)` 都应该返回 `true`
- 对称性，对于任何非空引用 x 和 y，当且仅当 `x.equals(y)` 返回 `true` 时，`y.equals(x)` 也应该返回 `true`
- 传递性，对于任何非空引用 x、y 和 z，如果 `x.equals(y)` 返回 `true`，且 `y.equals(z)` 返回 `true` 时，`x.equals(z)` 也应该返回 `true`
- 一致性，如果 x 和 y 引用的对象没有发生变化，反复调用 `x.equals(y)` 应该返回同样的结果
- 对于任何非空引用 x，`x.equals(null)` 应该返回 `false`

### 2.3  getClass() or instanceof

- 子类的相等性由各自重写的方法决定，且彼此不可互换时，使用 `getClass()`
- 子类的相等性由父类决定时，使用 `instanceof` 
- 当实现类似于 `Set`、`List`等通用数据结构时，其相等性通常使用 `instanceof InterfaceType`，并考虑添加 `final`

### 2.4  重写 equals() 方法的建议

1. 检测 this 与 otherObject 是否引用同一个对象

   ```java
   if(this == otherObject) return true;
   ```

2. 检测 otherObject 是否为 null

   ```java
   if(otherObject == null) return false;
   ```

3. 比较 this 和 otherObject 是否属于相同的类

   1. 如果equals 语义在每个子类中有所改变，那么使用 `getClass()`

      ```java
      if(getClass() != otherObject.getClass()) return false;
      ```

      

   2. 如果所有子类具有统一的语义，就使用 `instanceof` 来检测

      ```java
      if(!(otherObject instanceof ClassName)) return false;
      ```

4. 将 otherObject 转换为对应类的类变量

   ```java
   ClassName other = (ClassName) otherObject;
   ```

5. 对所有需要比较的域进行比较

   1. 通过 `==` 比较基本类型域
   2. 通过 `equals` 比较对象域

   如果所有域都匹配，就返回 `true`。否则，返回 `false`。

   ```java
   return feild1 == other.feild1 && feild2 == other.field2 && ...;
   ```


### 2.5  注意

重写 Object 方法时，参数列表不能改变，否则不是重写，而是重载

#### 2.5.1  典型错误

```java
public class Employee {
//    @Override
    public boolean equals(Employee other) {
        return this.name == other.name &&
                this.salary == other.salary &&
                this.hireday == other.hireday;
    }

    private String name;
    private int salary;
    private int hireday;
}
```

#### 2.5.2  解决方法

使用 `@Override` 注解对重写的方法进行标记

## 3  hashCode 方法

- 散列码是由对象导出的一个整型值。 不同对象得到的散列码不同。

- hashCode() 方法便是对象导出散列码的过程
- 当重写 equals() 方法时，就必须重写 hashCode() 以便用户可以将对象插入到散列表中
- equals 和 hashCode 的定义必须一致
  - 如果 `x.equals(y) == true`，那么 `x.hashCode() == y.hashCode()`





### 3.1  默认实现

```java
public native int hashCode();
```



### 3.2  重写hashCode



对于单独的对象

```java
int hashCode();
```

- 返回对象散列码
- 散列码可以是任意的整数，包括正数和负数
- 两个相等的对象要求返回相同的散列码



对于数组类型的对象，通过静态方法 `Array.hashCode()`

```java
static int hashCode(type[] a);
```

- 计算数组 a 的散列码





## 4  toString 方法

用于返回表示对象值的字符串



大部分 toString 的返回值样式为 `类名[属性]`

- 类名可以通过 `getClass()+getName()` 获取
- Object 类中定义的 toString 方法将返回 `类名@散列码



只要一个对象通过 `+` 运算符和任意一个字符串连接，Java编译时，就会自动调用 toString 方法。



### 4.1  数组

- 数组继承了 Object 类中的 toString 方法，而且没有重写
- 如果想要打印数组值，应调用静态方法 `Arrays.toString(数组名)`
- 多维数组，则应调用静态方法 `Arrays.deepToString(数组名)`



> [!note] 建议每一个类都拥有自己的 toString 方法

## 5  其余常用方法

- getClass()
  - 返回对象的类
- clone()
  - 返回一个对象的拷贝