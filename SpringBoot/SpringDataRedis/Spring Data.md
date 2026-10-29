## 1  Spring Data 是什么？

Spring Data 是 Spring 家族中的一个**顶级项目**，其目标是**统一并简化数据访问层的开发**。

- 所有 Spring Boot 整合 Spring Data 的启动器都叫 `spring-boot-starter-data-xxx`。
- 当 Spring Data 的子项目被 Spring Boot 整合后，项目内会多一个 `xxxTemplate` 实例。
	- Spring Data Redis 被整合后，多了一个 `RedisTemplate` 实例
	- `StringRedisTemplate` 是 `RedisTemplate` 的一个子类

### 1.1  命名规范与自动配置

Spring Boot 为每个 Spring Data 二级项目提供了**起步依赖（Starter）**，命名格式统一为：

```text
spring-boot-starter-data-xxx
```

例如：

- `spring-boot-starter-data-redis` → 整合 Redis
- `spring-boot-starter-data-jpa` → 整合 JPA

当你在项目中引入对应的 Starter 后，Spring Boot 的**自动配置**机制会生效，容器中会自动创建 **`xxxTemplate`** 实例作为操作入口。

对于 Redis 来说：

- 引入 `spring-boot-starter-data-redis` 后，Spring 容器中会多一个 **`RedisTemplate`** 实例。
- **`StringRedisTemplate`** 是 `RedisTemplate` 的子类，它默认使用字符串序列化器，适用于键和值都为字符串的简单场景。

## 2  Spring Data 好处

- 很方便操作对象类型（基于 POJO 模型）

## 3  Spring Data 版本与 Spring Boot 对照

|Spring Boot 版本|默认 Spring Data 版本|备注|
|---|---|---|
|2.7.x|Spring Data 2021.2.x|基于 Java 8/11|
|3.0.x - 3.3.x|Spring Data 2022.0.x - 2023.1.x|基于 Java 17，使用 Jakarta EE|
|4.1.x|Spring Data 2025.0.x|基于 Java 17+，模块化重构|

**特别注意**：Spring Boot 4 升级后，Spring Data 的某些核心 API（如 `RedisTemplate` 的序列化机制）有较大调整，迁移时需仔细阅读官方迁移指南。
