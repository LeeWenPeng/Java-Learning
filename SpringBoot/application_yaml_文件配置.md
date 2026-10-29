## 1 YAML 基本语法规则

- **缩进**：使用**空格**（禁止 Tab），同一层级必须对齐。
- **键值对**：`key: value`，冒号后必须有一个空格。
- **列表**：使用 `-` 表示数组元素。
- **多文档**：使用 `---` 分隔不同的配置段（常用于多环境）。

---

## 2 顶级配置结构概览

一个典型的 `application.yaml` 文件包含以下顶级节点：

```yaml
server:           # Web 服务器配置（Tomcat）
spring:           # Spring 生态核心配置（数据源、Redis、JPA、MVC等）
logging:          # 日志配置
management:       # Actuator 监控端点配置
mybatis-plus:     # 第三方框架自定义配置（如 MyBatis-Plus）
自定义节点:        # 业务自定义配置
```

---

## 3 数据源（DataSource）配置

用于连接 MySQL / MariaDB：

```yaml
spring:
  datasource:
    url: jdbc:mysql://主机名:3306/数据库名?参数
    username: lee
    password: ${MYSQL_PASSWORD:root}   # 支持环境变量，默认 root
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:                            # HikariCP 连接池专有配置
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
```

- `url`：`jdbc:mysql://主机名:3306/数据库名?参数`
	- `jdbc:mysql://`：JDBC 驱动协议头，告诉驱动管理器使用 MySQL 驱动。
	- `url`：数据库服务器的主机名。对应你的 `'root'@'localhost'` 用户权限（这正是你之前创建的 `root` 用户只能本地登录的配置）。
	- `3306`：MySQL/MariaDB 的默认端口。
	- `数据库名`：在 `端口号/` 后面指定具体的数据库名，在连接数据库服务器后，将选中该数据库
	- 参数
		- `useSSL=false`：禁用 SSL（安全套接层）加密传输。注意：数据以明文在网络中传输，容易被抓包窃听。开发环境可接受，**生产环境必须改为 `true` 并配置证书**。
		- `serverTimezone=Asia/Shanghai`：指定时区为东八区（北京时间）。如果不加，MySQL 驱动（8.0+）会默认使用 UTC 时区，导致 `TIMESTAMP` 字段存入和读出时发生时间偏移（差 8 小时）。这是最常见的坑之一。
		- `characterEncoding=`：指定字符集编码。最好和数据库的编码一致。
			- `UTF-8`: 在 MySQL 中实际是**3 字节** UTF-8，不支持 Emoji 和部分生僻字。
			- `utf8mb4`：
		- `allowPublicKeyRetrieval=true`：**这是 MySQL 8.0+ 特有的救急参数**。
			- **背景**：MySQL 8.0 默认使用 `caching_sha2_password` 加密插件。当客户端连接时，需要获取服务端的公钥来加密密码。
			- **作用**：如果客户端缓存中没有公钥，允许客户端自动从服务端请求公钥。如果不加，且未正确配置 SSL，会报错 `Public Key Retrieval is not allowed`。
- driver-class-name：指定加载哪个 Java 类作为 JDBC 驱动入口。

### 3.1 `allowPublicKeyRetrieval` 和 `useSSL` 的组合搭配规则

这两个参数经常组合出现，以下是常见的避坑对照表：

| useSSL  | allowPublicKeyRetrieval | 现象                                                      |
| :------ | :---------------------- | :------------------------------------------------------ |
| `true`  | `false`                 | 使用 SSL 加密传输，安全但不配置证书可能失败。                               |
| `false` | `true`                  | **最典型的开发配置**。明文传输，但强制允许获取公钥，连接最顺畅。                      |
| `false` | `false`                 | **几乎必报错**（除非 MySQL 配置改回了 `mysql_native_password`），连接不上。 |

## 4 JPA / Hibernate 配置

```yaml
spring:
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL8Dialect
    hibernate:
      ddl-auto: update   # 开发用，生产请改为 validate 或 none
    properties:
      hibernate:
        format_sql: true
        use_sql_comments: true
```

## 5 Redis 配置

Spring Boot 整合 Redis 的配置前缀已从 `spring.redis` 改为 **`spring.data.redis`**（Spring Boot 2.x 之后，4.x 亦同）。

```yaml
spring:
  data:
    redis:
      # 基础连接信息
      host: localhost               # Redis 服务器地址
      port: 6379                    # 端口
      password:                     # 密码（若无留空）
      database: 0                   # 使用的数据库索引（默认0）
      timeout: 2000ms               # 连接超时时间（毫秒）
      client-type: lettuce          # 客户端类型（默认 lettuce，也可选 jedis）
      
      # Lettuce 连接池配置（需引入 commons-pool2）
      lettuce:
        pool:
          max-active: 8             # 最大活跃连接数
          max-idle: 8               # 最大空闲连接
          min-idle: 0               # 最小空闲连接
          max-wait: -1ms            # 获取连接的最大等待时间（-1表示无限等待）
        shutdown-timeout: 100ms     # 关闭超时
      
      # Sentinel 模式（集群高可用）配置示例（注释）
      # sentinel:
      #   master: mymaster
      #   nodes: 192.168.1.1:26379,192.168.1.2:26379
      
      # Cluster 模式配置示例（注释）
      # cluster:
      #   nodes: 192.168.1.1:6379,192.168.1.2:6379
      #   max-redirects: 3
```

> **注意**：`spring.data.redis` 的配置项会被 Spring Boot 自动绑定到 `RedisProperties` 类，并在 `RedisAutoConfiguration` 中用于创建 `LettuceConnectionFactory` 等 Bean。

---

## 6 服务器（Server）配置

```yaml
server:
  port: 8080                     # 监听端口
  servlet:
    context-path: /api           # 应用上下文路径（如 localhost:8080/api/xxx）
  tomcat:
    max-threads: 200             # 最大工作线程数
    connection-timeout: 20000ms  # 连接超时
```

---

## 7 日志（Logging）配置

```yaml
logging:
  level:
    root: info                    # 全局日志级别
    org.lee.demo: debug           # 自定义包 debug 级别
    org.springframework.web: info
  pattern:
    console: "%d{HH:mm:ss.SSS} - %msg%n"
  file:
    path: /var/log/myapp          # 日志文件存放目录
    name: /var/log/myapp/root.log # 具体文件名
```

---

## 8 多环境配置（Profiles）

可以在同一个 YAML 文件中用 `---` 分隔不同环境：

```yaml
# 默认配置（通用）
spring:
  profiles:
    active: dev   # 默认激活 dev

---
# 开发环境（dev）
spring:
  config:
    activate:
      on-profile: dev
  data:
    redis:
      host: localhost

---
# 生产环境（prod）
spring:
  config:
    activate:
      on-profile: prod
  data:
    redis:
      host: redis-prod.example.com
      password: ${REDIS_PROD_PASSWORD}   # 从环境变量读取
server:
  port: 80
```

启动时通过 `--spring.profiles.active=prod` 切换。

---

## 9 占位符与属性引用

- **引用环境变量**：`${ENV_VAR_NAME:default_value}`
- **引用其他配置项**：`${spring.application.name}`
- **随机值**：`${random.uuid}`、`${random.int(100)}`

示例：

```yaml
app:
  name: MyApp
  description: This is ${app.name} application.
  secret: ${RANDOM_SECRET:${random.uuid}}
```

---

## 10 MyBatis-Plus 自定义配置（示例）

```yaml
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  type-aliases-package: org.lee.demo.entity
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

---

## 11 注意事项与最佳实践

1. **敏感信息不硬编码**：密码、密钥等应通过环境变量（`${VAR}`）或配置中心注入，绝对不要明文提交到 Git。
2. **缩进必须一致**：使用 IDE 自动格式化（如 IDEA 的 `Ctrl+Alt+L`）可避免 YAML 语法错误。
3. **优先使用 `application-{profile}.yaml` 拆分文件**：当配置项很多时，可以创建 `application-dev.yaml`、`application-prod.yaml` 等文件，便于管理。
4. **Spring Boot 4 的变化**：注意 `spring.data.redis` 前缀已取代旧的 `spring.redis`，且默认使用 Lettuce 客户端。
5. **测试连接**：配置完成后，启动 Spring Boot 应用，观察日志是否有 Redis 连接成功的信息，或执行简单的 `redisTemplate.opsForValue().set("test","ok")` 验证。

---

## 12 总结：你可能遇到的报错与对应关系

- **报错 `Public Key Retrieval is not allowed`**：解决方案就是在 URL 末尾加上 `&allowPublicKeyRetrieval=true`。
- **报错 `Unknown database 'msgcenter'`**：说明你忘了执行 `CREATE DATABASE msgcenter`，或者 URL 中的库名拼错了。
- **报错 `Access denied for user 'lee'@'localhost'`**：说明你的 `GRANT` 权限没配好，或者密码输入错误。请回到 MySQL 执行 `ALTER USER 'lee'@'localhost' IDENTIFIED BY 'root';` 重置密码。
