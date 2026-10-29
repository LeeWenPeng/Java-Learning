## 1 `opsForValue()`

是一个通用操作器，每次操作都需要显式传入 key

其对应的是 Redis 中的 `String` 数据类型

## 2 使用

```java
redisTemplate.opsForHash().xxx()
```

## 3 操作

### 3.1 SET

将键值对放入 Redis

```java
// 设置键值对，如果 key 已经存在，则覆盖旧值
void set(K key, V value);

// 设置键值对，并设置过期时间
void set(K key, V value, long timeout, TimeUnit unit);
```

- 参数
	- key：键
	- value：值
	- timeout：过期时间
	- unit：单位
		- `TimeUnit.SECONDS`

### 3.2 GET

根据 key 获取对应的值

```java
void get(Object key);
```

- 参数
	- key：键

### 3.3 SETNX/SETEX

条件性 SET

- 常用于实现分布式锁等场景

```java
// 当 key 不存在时，设置值
Boolean setIfAbsent(K key, V value);

// 当 key 存在时，设置值
Boolean setIfPresent(K key, V value);
```

- 参数
	- key：键
	- value：值
- 返回值
	- 成功：true
	- 失败：false

### 3.4 INCR/DECR

为 key 对应的值增加/减少指定的 delta 值

- 常用于实现计数器、限流器等

```java
Long increment(K key, long delta)
Long increment(K key, double delta)
Long decrement(K key, long delta)
```

- 参数
	- key：键。
		- 要求 key 必须是数字类型的字符串，否则抛出异常。
	- delta：变化的值

### 3.5 MSET/MGET

批量操作，一次设置多个键值对，或一次获取多个 key 的值

- 相对于 SET/GET，可以减少网络往返次数，提高性能

```java
void multiSet(Map<? extends K, ? extends V> map)
void multiGet(Collection<K> keys)
```

- map：`<key, value>` 键值对
- keys：多个 key

### 3.6 getAndSet

获取 key 对应的旧值，并将其更新为 value

- 先进行 get 再进行 set

```java
String getAndSet(K key, V value)
```

- 返回值
	- key 对应的旧的值

### 3.7 SETRANGE/GETRANGE

```java
// 设置键值对，从指定的 offset 开始，覆盖旧值。对应 SETRANGE 命令
void set(K key, V value, long offset)

// 获取 key 对应的 value 的子串 `[start, end]`
void get(K key, long start, long end)
```

- 参数
	- key：键
	- value：值
	- offset：偏移量
- 返回值

## 4 `boundValueOps(key)`

是一个绑定到指定 key 的操作器。

- 在创建时，就已经绑定了一个 key，后续所有操作都无需再传入该 key。
- 适用于代码中需要频繁操作同一个 key 的情况
