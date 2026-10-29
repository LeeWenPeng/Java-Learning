## 1  核心组件

### 1.1  实体类

Customer

```java
package org.Reflection;

public class Customer {
    private String name;
    private String email;
    public Customer(String name, String email){
        this.name = name;
        this.email = email;
    }
    public Customer(){}

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    @Printable
    public void printName(){
        System.out.println(name);
    }

    @Printable
    public void printEmail(){
        System.out.println(email);
    }
}
```

Address

```java
package org.Reflection;

public class Address {
    private String street;
    private String postcode;

    public Address(){}
    public Address(String street, String postcode){
        this.street = street;
        this.postcode = postcode;
    }

    public void printStreet(){
        System.out.println("Address: " + street);
    }

    public void printPostcode(){
        System.out.println("Postcode: " + postcode);
    }
}
```

Order

```java
package org.Reflection;

public class Order {
    private Customer customer;
    private Address address;

    public Order(){}

    @Autowired
    public Order(Address address, Customer customer){
        this.customer = customer;
        this.address = address;
    }

    public Address getAddress(){
        return address;
    }

    public Customer getCustomer() {
        return customer;
    }

}
```

Message

```java
package org.Reflection;

public class Message {
    private String content;

    public Message(){}

    @Autowired
    public Message(String content){
        this.content = content;
    }

    @Printable
    public  void printMessage(){
        System.out.println("Message: " + this.content);
    }
}
```

User

```java
package org.Reflection;

import java.util.List;

public class User extends Person{

    @MyAnnotation
    public String name;
    private final int age;
    private String email;
    private Message message;
    private List<String> comments;
    public static int publicStaticField = 1;
    private static int privateStaticField = 10;

    static {
        System.out.println("User Class is initialized");
    }

    public User(String name, int age){
        this.name = name;
        this.age = age;
    }

    @Autowired
    public User(Message message){
        this.message = message;
        this.age = 18;
    }

    public String getName(){
        return name;
    }

    public int getAge(){
        return age;
    }

    public String getEmail(){
        return email;
    }

    public void myPublicMethod(){
        System.out.println("This is a public method");
    }

    private void myPrivateMethod(){
        System.out.println("This is a private method");
    }

    private void myPrivateMethod(String content, String mark){
        System.out.println("This is a private method with two parameters. " );
        System.out.println("content: " + content);
        System.out.println("mark: " + mark);
    }

    public static void myPublicStaticMethd(){
        System.out.println("This is a public static method. ");
    }
    private static void myPrivateStaticMethd(){
        System.out.println("This is a private static method. ");
    }
    private static void myPrivateStaticMethd(String content){
        System.out.println("This is a private static method with parameters. ");
        System.out.println("Content: " + content);
    }

    public String toString(){
        return name + " : " + age;
    }
}
```

### 1.2  配置类 Config

```java
package org.Reflection;

/*
    定义服务，实例化并返回对象
 */
public class Config {

    @Bean
    public Customer customer(){
        return new Customer("Lee", "xxx@gmail.com");
    }

    @Bean
    public Address address(){
        return new Address("this is a address", "10010");
    }

    @Bean
    public Message message(){
        return new Message("Hi this is a message!");
    }
}
```

## 2  容器

```java
package org.Reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

/*
    注册服务，获得实例
 */
public class Container {

    private Map<Class<?>, Method> methods; // 服务创建方法的映射，也就是，存储的是创建实例的方法
    private Map<Class<?>, Object> services; // 服务单例缓存（单例池），存储的是服务的实例
    private Object config; // config 实例，用于调用方法获取服务

    /*
        初始化
     */
    public void init() throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {

        this.methods = new HashMap<>(); // 初始化方法列表
        this.services = new HashMap<>(); // 初始化服务缓存

        // 获取方法
        Class<?> clazz = Class.forName("org.Reflection.Config");
        Method[] methods = clazz.getDeclaredMethods();
        for(Method method : methods){
            if(method.getDeclaredAnnotation(Bean.class) != null){
                this.methods.put(method.getReturnType(), method);
            }
        }

        this.config = clazz.getConstructor().newInstance(); // 获取 config 实例化对象
    }

    /*
        通过类的 Class 对象和 config 获取服务实例

     */
    public Object getServiceInstanceByClass(Class<?> clazz) throws InvocationTargetException, IllegalAccessException {
        // 1. 如果 services 中不存在该对象，则新建对象，否则直接返回
        if(services.containsKey(clazz))
            return services.get(clazz);

        // 2. 新建对象。
            // 通过 Class 对象为键，从 methods 中获取服务方法并调用，将获得的实例返回。
            // 服务方法配置在 config 对象中
        if(this.methods.containsKey(clazz)){
            Method method = this.methods.get(clazz);
            Object obj = method.invoke(this.config);
            services.put(clazz, obj); // 将新创建的服务加入缓存
            return obj;
        }
        return null;
    }

    /*
        通过 Class 对象创建普通实例，并且自动注入
     */
    public Object createInstance(Class<?> clazz) throws InvocationTargetException, IllegalAccessException, InstantiationException, NoSuchMethodException {
        Constructor<?>[] constructors = clazz.getDeclaredConstructors();
        for(Constructor<?> constructor : constructors){
            if (constructor.getDeclaredAnnotation(Autowired.class)!=null){
                // 解析并获取服务
                Class<?>[] parameterTypes = constructor.getParameterTypes(); // 通过构造器的参数列表，获取服务类型
                Object[] arguments = new Object[parameterTypes.length]; // 用于存储服务
                for(int i = 0; i < parameterTypes.length; ++i){
                    arguments[i] = getServiceInstanceByClass(parameterTypes[i]); // 获取服务
                }
                return constructor.newInstance(arguments); // 将服务注入，创建并返回完整的对象
            }
        }
        return clazz.getConstructor().newInstance(); // 返回无参构造的实例
    }
}
```

## 3  Main

```java
package org.Reflection;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main3 {
    public static void main(String[] args) throws ClassNotFoundException, InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException, NoSuchFieldException {

        Container container = new Container();
        container.init();
//        String className = "org.Reflection.Order"; 
        String className = "org.Reflection.User";
        String fieldName = "message";
        Class<?> clazz = Class.forName(className);
        Object obj = container.createInstance(clazz); // 创建实例对象
//        System.out.println(obj);
        Field field = clazz.getDeclaredField(fieldName); // 获取字段
        field.setAccessible(true); // 设置字段访问权
        Object fieldVal = field.get(obj); // 获取字段属性值，也就是成员的实例对象
        Method [] methods = fieldVal.getClass().getDeclaredMethods(); // 获取成员对象中的方法。其中，.getClass() 获取该实例对象对应的 Class 对象
        for(Method method : methods){ // 遍历成员的方法列表，找到相应注解并执行
            if(method.getDeclaredAnnotation(Printable.class) != null){
                method.invoke(fieldVal);
            }
        }
    }
}
```
