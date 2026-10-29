## 1  概念

java.lang.Object 是所有类的超类，也就是说，所有的类都是 Object 的扩展类，并且都可以使用 Object 的所有方法。



当一个类被创建时，就隐式继承了 Object 类，当然你仍然可以显式继承：



```java
// 隐式继承
public class ClassA{
    
}

// 显式继承——不需要这样做
public class ClassB extends Object{
    
}
```

## 2  Object 类型的变量





