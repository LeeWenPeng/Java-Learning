## 1  什么是 AtomicInteger

- 位于 `import java.util.concurrent.atomic` 包
- 提供原子操作的整数
- 底层基于 CAS 实现，利用 Unsafe 类的 `compareAndSwapInt` 方法，由 CPU 硬件保证原子性

## 2  关键方法

### 2.1  `incrementAndGet`

- 原子地将当前值加一，并返回新值
- 相当于 `++i`

### 2.2  `getAndIncrement`

- 原子地将当前值加一，并返回旧值
- 相当于 `i++`

### 2.3  `get`

- 返回当前值

### 2.4  `compareAndSet`

- 如果当前值等于期望值，则原子更新为新值

## 3  CAS 机制

- Compare-And-Swap
- 是一种无锁的乐观并发策略
- 操作数：内存位置 `V`，预期旧值 `A`，新值 `B`
- 只有当 `V==A` 时，才原子更新 `V=B`，否则失败并重试
- 底层由 CPU 的 `cmpxchg` 指令支持，在硬件层面上保证原子性
