在实际开发中，通常需要为**开发（dev）**、**测试（test）**、**生产（prod）** 等环境准备不同的配置。

## 1  快速参考卡片

| 操作      | 方式                              |
| :------ | :------------------------------ |
| 指定环境    | `spring.profiles.active: dev`   |
| 命令行指定环境 | `--spring.profiles.active=prod` |
| 配置文件命名  | `application-{profile}.yaml`    |
| 配置覆盖优先级 | 命令行 > 环境变量 > 环境配置文件 > 主配置文件     |

## 2  文件命名规范

Spring Boot 支持按环境拆分配置文件，命名格式为：

```java
application-{环境标识}.yaml
```

例如：

- `application-dev.yaml` — 开发环境
- `application-test.yaml` — 测试环境
- `application-prod.yaml` — 生产环境

## 3  激活指定环境

在主配置文件 `application.yaml`（或 `application.properties`）中，通过 `spring.profiles.active` 指定当前激活的环境：

```yaml
spring:
  profiles:
    active: dev    # 激活 application-dev.yaml
```

启动时，Spring Boot 会加载：

1. `application.yaml`（基础配置，所有环境共享）
2. `application-dev.yaml`（环境专属配置，**覆盖**基础配置中的同名项）

> [!note]
> 如果多个环境有共享的配置（如 Redis 的 `port` 固定为 6379），可以将其放在 `application.yaml` 中，避免在每个环境文件中重复。

### 3.1  其他激活方式

除了在 `application.yaml` 中固定激活外，还可以通过以下方式**动态指定**：

- **命令行参数**（优先级最高）：

  ```bash
  java -jar myapp.jar --spring.profiles.active=prod
  ```

- **操作系统环境变量**：

  ```bash
  export SPRING_PROFILES_ACTIVE=prod
  java -jar myapp.jar
  ```

- **IDEA 启动配置**：在 `Run/Debug Configurations` 的 `Program arguments` 中添加 `--spring.profiles.active=prod`。

## 4  项目目录结构示例

```java
src/main/resources/
├── application.yaml           # 主配置（公共部分）
├── application-dev.yaml       # 开发环境
├── application-test.yaml      # 测试环境
└── application-prod.yaml      # 生产环境
```

## 5  各环境配置示例

**application.yaml（公共）**

```yaml
server:
  port: 8080
spring:
  profiles:
    active: dev
```

**application-dev.yaml（开发）**

```yaml
server:
  port: 8080                    # 保持默认
spring:
  data:
    redis:
      host: localhost
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db
```

**application-prod.yaml（生产）**

```yaml
server:
  port: 80                      # 覆盖为 80
spring:
  data:
    redis:
      host: redis-prod.example.com
      password: ${REDIS_PASSWORD}
  datasource:
    url: jdbc:mysql://prod-db.example.com:3306/prod_db
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

## 6  配置加载顺序（优先级）

配置的覆盖优先级从高到低为：

1. 命令行参数（`--spring.profiles.active=prod`）
2. 操作系统环境变量（`SPRING_PROFILES_ACTIVE`）
3. `application-{profile}.yaml`（环境专属配置）
4. `application.yaml`（基础配置）

同一配置项，**高优先级覆盖低优先级**。

---

## 7  多文档 YAML

除了拆分为多个文件，也可以在**同一个 YAML 文件**中用 `---` 分隔多环境，适用于配置较少的项目：

```yaml
# application.yaml
spring:
  profiles:
    active: dev

---
spring:
  config:
    activate:
      on-profile: dev
  data:
    redis:
      host: localhost

---
spring:
  config:
    activate:
      on-profile: prod
  data:
    redis:
      host: redis-prod.example.com
```

> **建议**：当配置项较多时，仍推荐拆分为独立的文件，更清晰易维护。
