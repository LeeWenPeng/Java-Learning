## 1  @Bean 是什么？

`@Bean` 是一个方法级别的注解。其核心作用是告诉 Spring 容器：调用该方法，并将返回的对象托管到 Spring IoC 容器中。

换句话说，`@Bean` 就是告诉 Spring 容器，执行该方法，并且注册该方法的执行结果为 Bean。

## 2  执行流程

1. Spring 容器启动时，扫描到 `@Configuration` 类
2. 执行被 `@Bean` 注解的方法
3. 获取方法返回的对象，将其放入 Spring 容器。默认的 Bean 的名称就是方法名
4. 其他类可以通过 `@Autowired` 直接注入该 Bean。

## 3  核心属性

| 属性                      | 作用                                | 示例                                                              |
| ----------------------- | --------------------------------- | --------------------------------------------------------------- |
| **`name` / `value`**    | 指定 Bean 在容器中的名称（别名）。不写则默认是方法名。    | `@Bean("myRedis")`  <br>注入时用 `@Qualifier("myRedis")`            |
| **`initMethod`**        | Bean 创建后，自动调用的初始化方法名。             | `@Bean(initMethod="start")`  <br>对应类中的 `public void start()`    |
| **`destroyMethod`**     | 容器销毁前，自动调用的销毁方法名（释放资源）。           | `@Bean(destroyMethod="close")`  <br>对应类中的 `public void close()` |
| **`autowireCandidate`** | 是否可作为其他 Bean 的自动装配候选者。默认为 `true`。 | `@Bean(autowireCandidate=false)` 暂不参与装配                         |

## 4  特性

### 4.1  方法间调用（代理逻辑）

在 `@Configuration` 类中，直接调用另一个 `@Bean` 方法，默认会从容器中**获取单例对象**，而不是重新 new 一个。

```java
@Configuration
public class AppConfig {
	@Bean
	public UserService userService() {
		// 注意：这里调用 userRepository()，不会重新创建，而是从容器拿已有的！
		return new UserService(userRepository());
	}

	@Bean
	public UserRepository userRepository() {
		return new UserRepository();
	}
}
```

这是因为 `@Configuration` 类会被 Spring 通过 **CGLIB 动态代理**增强，确保 Bean 的单例特性。

### 4.2  结合 `@Scope` 指定作用域

默认是单例（`singleton`）。如果想每次获取都新建一个，可以加上 `@Scope`：

```java
@Bean
@Scope("prototype") // 每次获取都 new 一个
public PrototypeClass prototypeClass() {
	return new PrototypeClass();
}
```

### 4.3  条件化加载 (`@Conditional`)

结合 Spring Boot 的自动配置，你可以让 `@Bean` 在特定条件下才生效（比如类路径有某个 jar，或配置文件中存在某个属性）：

```java
@Bean
@ConditionalOnMissingBean // 只有当容器中没有 RedisTemplate 时才创建（Spring Boot 常用）
public RedisTemplate<String, Object> redisTemplate() {
	// …
}
```
