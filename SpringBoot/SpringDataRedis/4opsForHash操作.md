## 1 使用

```java
redisTemplate.opsForHash().
```

## 2 操作

### 2.1 set

```java
void set(K key, V value);

// 设置 <hashKey, value> 到哈希表(key)中
void put(H key, HK hashKey, HV value);

// 设置多个 <hashKey, value> 到哈希表(key)中
void putAll(H key, Map<? extends HK, ? extends HV> m);
```

- 参数
	- key：哈希表的键
	- value：
	- m：`<hashKey, value>` 的字典

### 2.2 REMOVE

从哈希表中删除键值

```java
Long delete(H key, Object… hashKeys);
```

- 参数
	- key
	- hashKeys：可变参数，可以传入一个或多个 hashKey

### 2.3 hasKey

判断哈希表 `key` 中是否具有该值 `hashKey`

```java
Boolean hasKey(H key, Object hashKey);
```

- 参数
	- key：哈希表对应的键
	- hashKey：哈希表中的键
