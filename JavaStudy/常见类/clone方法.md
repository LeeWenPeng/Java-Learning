## 1  暴论

所谓浅拷贝和深拷贝，本质上只针对引用类型的数据

- 浅拷贝：拷贝引用地址
- 深拷贝：将引用地址指向的内容拷贝到新地址中，创建一个新引用指向该地址

## 2  浅拷贝

### 2.1  概念

- 创建一个新对象，但该新对象中的**引用类型字段**仍然指向原始对象中对应字段所指向的同一内存地址。
	- **针对基本数据类型**：直接复制其**字面量值**，修改副本的基本类型字段不会影响原对象。
	- **针对引用数据类型**：仅复制其**内存地址引用**。因此，通过副本修改该引用指向的对象内容时，原对象也会同步受到影响（反之亦然）。
- **Java 语言特性**：`Object` 类提供的默认 `clone()` 方法执行的就是浅拷贝。此外，数组的 `clone()` 方法、集合构造器（如 `new ArrayList(originalList)`）默认均属于浅拷贝范畴。

### 2.2  String 对象

String 对象很特殊，其是不可变对象。

- 浅拷贝：String 的拷贝为浅拷贝。拷贝完成后，原对象的 `String` 引用和副本对象的 `String` 引用**指向堆内存中的同一个 `String` 实例**。
- 引用重新绑定：由于 `String` 是**不可变对象**，堆内存中的 `char[]` 内容无法被修改。当修改拷贝得到的 String 对象的值时，其实没有修改原对象内容，而是将引用绑定到了一个新的字符串地址。

## 3  深拷贝

### 3.1  概念

- 创建一个新对象，并递归地将原始对象中所有**引用类型字段**所指向的对象也进行完整复制，直至基本类型或不可变对象。
- 复制整个**对象图**。拷贝后的对象与原对象在内存中**完全隔离**。

## 4  案例

拷贝函数位于 [3.1.2  Student](clone方法.md#3.1.2%20Student)

### 4.1  Bean 类

#### 4.1.1  Subject

```java
package Object.CloneTest;

public class Subject {
    private String name;

    public Subject(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Subject{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

#### 4.1.2  Student

```java
package Object.CloneTest;
public class Student implements Cloneable{
    private Subject subject;
    private String name;

    public Student(Subject subject, String name) {
        this.subject = subject;
        this.name = name;
    }

    public Subject getSubject() {
        return subject;
    }

    public void setSubject(Subject subject) {
        this.subject = subject;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

//    @Override
//    protected Object clone() throws CloneNotSupportedException {
//        // 浅拷贝
//        return super.clone();
//    }
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return new Student(new Subject(this.getSubject().getName()), this.getName());
    }

    @Override
    public String toString() {
        return "Student{" +
                "subject=" + subject +
                ", name='" + name + '\'' +
                '}';
    }
}
```

### 4.2  Test 类

```java
package Object.CloneTest;

public class Test {
    public static void main(String[] args) throws CloneNotSupportedException {
        // 原始对象
        Student student = new Student(new Subject("Math"), "Lee");
        System.out.println("原始对象: " + student);

        // 浅拷贝
        Student copyStudent = (Student) student.clone();
        System.out.println("拷贝对象: " + copyStudent);

        // 对象是否为同一个
        System.out.println("Object: " + (student == copyStudent));
        System.out.println("Name： " + (student.getName() == copyStudent.getName()));
        System.out.println("Subject：" + (student.getSubject() == copyStudent.getSubject()));

        // 修改拷贝对象
        copyStudent.setName("Wang"); // copyStudent 指向新的 String 对象
        copyStudent.getSubject().setName("yu wen"); // 没有修改 Subject 的引用指向，而是修改类内 Name 指向新的 String 对象
        System.out.println("修改了拷贝对象后的原始对象: " + student);
        System.out.println("修改了拷贝对象后的拷贝对象: " + copyStudent);
    }
}
```
