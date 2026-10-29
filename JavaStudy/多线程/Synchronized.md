- `Synchronized` 是 Java 提供的最基本、最常用的内置同步机制。
- 用于保证多线程环境下对共享资源访问的原子性、可见性和有序性。
- 基于监视锁（Monitor Lock）实现，是 JVM 层面的互斥锁

## 1  `Synchronized` 三种用法

### 1.1  修饰实例方法

```java
public synchronized void method(){
	// todo... 同步代码
}
```

- 锁对象：当前的实例对象
- 同一时刻，同一实例的多个同步方法只能被一个线程执行
- 不同实例的同步方法互不干扰

### 1.2  修饰静态方法

```java
public static synchronized void method(){
	// todo... 同步代码
}
```

- 锁对象：当前类的 `Class` 对象
- 同一时刻，所有线程无论通过哪个实例方法调用该静态方法，都只能串行执行

### 1.3  同步代码块

```java
synchronized (lockObject){
	// todo... 同步代码
}
```

- 锁对象：任意对象
- 只锁定代码块范围内的操作
- 粒度更细，性能更好

> [!note] synchronized 锁的目标
> synchronized 锁的是对象，而非代码。不同线程，如果使用不同的实例对象，则不会互斥
