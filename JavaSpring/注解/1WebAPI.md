## 1  核心原理：本质都是 `@RequestMapping`

从 Spring 4.3 开始，引入了 `@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`、`@PatchMapping`。

查看 `@PostMapping` 的源码，你会发现它本身被 `@RequestMapping` 标注，并预设了 `method = RequestMethod.POST`：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = RequestMethod.POST) // 核心就在这里
public @interface PostMapping {
	// … 继承了 @RequestMapping 的属性（别名）
}
```

**结论**：`@RequestMapping` 是 " 万能地图 "，而这些 Mapping 是 " 专用导航 "。**官方强烈建议**：在明确 HTTP 方法时，优先使用专用注解，因为它们更语义化，且能利用框架进行更精准的匹配校验。

---

## 2  五大 Mapping 全家桶对比

| 注解                   | 对应 HTTP 方法 | 主要业务场景                 | 是否幂等  | 请求体（Body）常见用法                              |
| -------------------- | ---------- | ---------------------- | ----- | ------------------------------------------ |
| **`@GetMapping`**    | GET        | 查询、获取资源（列表/详情）         | **是** | 极少带 Body，通常使用 Query String 或 Path Variable |
| **`@PostMapping`**   | POST       | 创建新资源、表单提交、复杂的查询（安全考虑） | **否** | 常配合 `@RequestBody` 传递 JSON/XML，或表单数据       |
| **`@PutMapping`**    | PUT        | **全量**更新资源（替换）         | **是** | 常携带完整的资源 DTO                               |
| **`@PatchMapping`**  | PATCH      | **部分**更新资源（修改个别字段）     | **否** | 携带差异字段（如 JSON Patch 或只传修改的字段）              |
| **`@DeleteMapping`** | DELETE     | 删除资源                   | **是** | 通常无需 Body，通过路径或参数传递 ID                     |

> **幂等性小贴士**：幂等指一次请求和多次请求的效果相同。GET/PUT/DELETE 在规范上是幂等的；POST 创建数据多次会生成多条记录，PATCH 可能涉及复杂逻辑，通常也不保证幂等。

## 3  HTTP 方法与 CRUD 的严格对应（最重要的一张表）

| 操作       | HTTP 方法 | URL 示例         | Spring 注解        | 返回值建议                               |
| -------- | ------- | -------------- | ---------------- | ----------------------------------- |
| **查询列表** | GET     | `/templates`   | `@GetMapping`    | 200 OK + JSON 数组                    |
| **查询单个** | GET     | `/templates/1` | `@GetMapping`    | 200 OK + JSON 对象 / 404              |
| **创建资源** | POST    | `/templates`   | `@PostMapping`   | **201 Created** + 新资源路径（Location 头） |
| **全量更新** | PUT     | `/templates/1` | `@PutMapping`    | 200 OK / 204 No Content             |
| **部分更新** | PATCH   | `/templates/1` | `@PatchMapping`  | 200 OK / 204 No Content             |
| **删除资源** | DELETE  | `/templates/1` | `@DeleteMapping` | **204 No Content**（不返回数据）           |
