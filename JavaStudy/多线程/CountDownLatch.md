`CountDownLatch` 是 `java.uitl.concurrent` 包下的同步工具类，允许一个或多个线程等待其他线程完成。

## 1  构造

```java
CountDownLatch latch = new CountDownLatch(int count);
```

- `count` 表示需要等待的 " 计数 " 次数。

## 2  核心方法

### 2.1  countDown

```java
latch.countDown();
```

- 作用：将 `count` 减 1，表示一个任务已完成

### 2.2  await

```java
latch.await();
```

- 作用：阻塞当前线程，直到计数变为 0
