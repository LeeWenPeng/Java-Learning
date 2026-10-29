## 1  核心数据结构：红黑树（Red-Black Tree）

`TreeMap` 底层基于**红黑树**（一种自平衡的二叉查找树）实现。

- **有序性**：所有键按照自然顺序（`Comparable`）或构造时指定的**比较器**（`Comparator`）进行排序。
- **平衡性**：红黑树确保树的高度始终保持在 O(log n)，避免了二叉查找树在极端情况下退化成链表。

---

## 2  时间复杂度

| 操作 | 时间复杂度 | 说明 |
| :--- | :--- | :--- |
| `put`、`get`、`remove`、`containsKey` | **O(log n)** | 基于树的高度进行查找和旋转 |
| `firstKey`、`lastKey` | O(log n)（或 O(1) 维护指针，取决于实现，标准为 O(log n)） | 获取最小/最大键 |
| `floorEntry`、`ceilingEntry`、`higherEntry`、`lowerEntry` | **O(log n)** | 利用树结构进行边界查找 |
| 遍历（`entrySet` 迭代） | O(n) | 中序遍历输出有序序列 |

> **注**：`HashMap` 的 `get`/`put` 是 O(1)，但 `TreeMap` 的有序性和范围查询能力是 `HashMap` 无法替代的。

---

## 3  核心 API 详解（结合你的区间管理场景）

`TreeMap` 实现了 `NavigableMap` 接口，提供了丰富的**导航方法**

### 3.1  (1) 边界查找方法

| 方法                             | 描述                           | 你的代码应用                                                      |
| :----------------------------- | :--------------------------- | :---------------------------------------------------------- |
| `floorEntry(K key)`            | 返回 ≤ 指定键的**最大**键值对（若无则 null） | `startToEnd.floorEntry(index)`：找到包含 `index` 的区间（起点 ≤ index） |
| `ceilingEntry(K key)`          | 返回 ≥ 指定键的**最小**键值对（若无则 null） | `startToEnd.ceilingEntry(end+1)`：找到右侧相邻区间的起点（起点 ≥ end+1）    |
| `lowerEntry(K key)`            | 返回 < 指定键的**最大**键值对（严格小于）     | 用于找严格左侧邻居                                                   |
| `higherEntry(K key)`           | 返回 > 指定键的**最小**键值对（严格大于）     | 用于找严格右侧邻居                                                   |
| `firstEntry()` / `lastEntry()` | 返回最小/最大的键值对                  | 获取全局最小/最大区间起点                                               |

### 3.2  (2) 范围视图

- `subMap(fromKey, toKey)`：返回子映射（左闭右开）
- `headMap(toKey)`：返回小于 `toKey` 的部分
- `tailMap(fromKey)`：返回大于等于 `fromKey` 的部分

这些方法在需要按区间批量查询时非常有用。

### 3.3  (3) 其他常用方法

- `put(K key, V value)`：插入键值对，若键已存在则覆盖。
- `remove(Object key)`：删除键。
- `containsKey(Object key)`：判断键是否存在。
- `size()` / `isEmpty()`：集合大小和判空。

---

## 4  排序规则

### 4.1  (1) 自然排序（默认）

键必须实现 `Comparable` 接口，`TreeMap` 调用 `compareTo` 方法排序。

```java
TreeMap<Integer, String> map = new TreeMap<>(); // 键为 Integer，自然升序
```

### 4.2  (2) 自定义排序（传入 Comparator）

在构造时传入比较器，适用于键本身未实现 `Comparable` 或需要定制顺序。

```java
TreeMap<String, Integer> map = new TreeMap<>((a, b) -> b.compareTo(a)); // 降序
```

> **注意**：如果键为 `null`，`TreeMap` 会抛出 `NullPointerException`（因为比较时需要调用 `compareTo` 或 `compare`），而 `HashMap` 允许一个 `null` 键。

---

## 5  `TreeMap` vs `HashMap` vs `LinkedHashMap`

| 特性 | `HashMap` | `LinkedHashMap` | `TreeMap` |
| :--- | :--- | :--- | :--- |
| 底层结构 | 哈希表 + 链表（解决冲突） | 哈希表 + 双向链表 | 红黑树 |
| 元素顺序 | 无序 | **插入顺序** 或 **访问顺序** | **键的自然顺序** 或 **比较器顺序** |
| 时间复杂度 | O(1)（平均） | O(1)（平均） | O(log n) |
| 是否允许 null 键 | 允许一个 | 允许一个 | **不允许** |
| 适用场景 | 快速查找、无顺序要求 | 需要保持插入/访问顺序 | 需要排序、范围查询、导航 |

---

## 6  在区间合并问题中的优势（回顾你的代码）

为什么你的代码选择 `TreeMap` 而不是 `HashMap`？

1. **区间起点有序**：`startToEnd` 的键是起点，需要按从小到大排列，便于查找 `floorEntry(index)` 定位包含 `index` 的区间。
2. **高效导航**：`ceilingEntry(end+1)` 可以 O(log n) 找到右侧第一个区间，无需遍历。
3. **动态更新**：每次修改后，区间起点的顺序依然保持，TreeMap 自动维护树平衡。

如果使用 `HashMap`，你需要遍历所有键来查找包含 `index` 的区间，时间复杂度退化为 O(n)，无法通过 LeetCode 的大数据测试。

---

## 7  常见陷阱与注意事项

1. **可变键的陷阱**：如果键是可变对象，修改键的内容后，`TreeMap` 不会自动调整位置，导致无法找到该键。**推荐使用不可变键**（如 `String`、`Integer`）。
2. **比较器一致性**：比较器必须与 `equals` 一致，否则 `containsKey` 等操作可能失效。
3. **并发问题**：`TreeMap` 非线程安全。在多线程环境下，可使用 `Collections.synchronizedSortedMap(new TreeMap<>())` 或 `ConcurrentSkipListMap`（并发版有序映射）。
4. **内存开销**：由于红黑树需要存储父节点、左右子节点和颜色标记，内存占用比 `HashMap` 大。

---

## 8  扩展：与 `ConcurrentSkipListMap` 的对比

`ConcurrentSkipListMap` 是 Java 并发包中的有序映射，底层基于**跳跃表**，具有类似的 O(log n) 操作，且支持高并发。如果你的区间管理场景涉及多线程，可以选用它，否则 `TreeMap` 更轻量。

---

## 9  总结

`TreeMap` 的核心价值在于**有序性**和**高效的导航能力**。当你需要：

- 维护一个动态有序集合；
- 快速查找 " 小于等于 x 的最大元素 " 或 " 大于等于 x 的最小元素 "；
- 按区间分割、合并数据；

`TreeMap` 是你的不二之选。正因如此，你刚才的 `longestRepeating` 解法才能从 O(k·n) 优化到 O((n+k) log n)，这是算法竞赛和工程实践中的经典模式。如果还有深入问题，欢迎继续探讨。😊
