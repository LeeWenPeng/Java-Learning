## 本质

`@SpringBootApplication` 是 Spring Boot 的**组合注解**，其本质是三个核心注解的聚合，并在此基础上通过 Spring 的 `@AliasFor` 机制，将自身的属性 " 透传 " 给底层的元注解。

- `@SpringBootConfiguration`：本质是 `@Configuration` 的派生，作用是将当前类标记为一个**配置类**，允许在其中通过 `@Bean` 方法定义 Bean。这是 Spring Boot 对 " 主配置类 " 的专用语义标记。
- `@EnableAutoConfiguration`：是 Spring Boot " 约定优于配置 " 理念的核心载体。它通过 `@Import(AutoConfigurationImportSelector.class)` 引入了一个**导入选择器**，在应用启动时**触发自动配置类的加载流程**。
- `@ComponentScan`：负责扫描指定包及其子包下的 `@Component`、`@Service`、`@Controller` 等注解，并将它们注册为 Bean。
	- **关键在于**：这个 `@ComponentScan` 并非 " 裸 " 注解，它**固定携带了两个排除过滤器**： TypeExcludeFilter 和 AutoConfigurationExcludeFilter。
	- 这是 Spring Boot 刻意设计的——自动配置类由 `@EnableAutoConfiguration` 的独立机制加载，组件扫描必须把它们排除掉，避免重复注册。
	- 这也是为什么 " 额外显式添加 `@ComponentScan`" 会引入隐患。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

### `@AliasFor`：属性的跨注解 " 透传 " 机制

`@SpringBootApplication` 自身声明的属性（如 `exclude`、`scanBasePackages`、`proxyBeanMethods`）并非独立存在，它们都通过 `@AliasFor` **映射到了元注解的同名属性上**。这意味着你在 `@SpringBootApplication` 上写 `scanBasePackages`，最终生效的其实是元注解 `@ComponentScan` 的 `basePackages` 属性。

#### 指向 `ComponentScan` 的别名

```java
@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
String[] scanBasePackages() default {};

@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
Class<?>[] scanBasePackageClasses() default {};

@AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")
Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
```

- `scanBasePackages` **不是独立属性**，它是元注解里那个 `@ComponentScan` 的 `basePackages` 的**别名**。
- 这意味着，写 `@SpringBootApplication(scanBasePackages = {"a", "b"})` 时，Spring 通过 `@AliasFor` 把值**合并进那一个唯一的 `@ComponentScan` 实例**。扫描器只有一个，`excludeFilters` 原封不动保留。

## 语法

将该注解写到客户端上，可以指定 `scanBasePackages`

```java
@SpringBootApplication(scanBasePackages = {"com.demo.xTimerSystem", "cn.bitoffer"})
public class DemoApplication {

    public static void main(String[] args) {
	    //todo...
    }

}
```
