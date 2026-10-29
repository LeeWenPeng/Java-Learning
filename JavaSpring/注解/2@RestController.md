## 1  @RestController 是组合注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller          // 继承了 Controller 的 Bean 注册能力
@ResponseBody        // 核心！类注解，表示该类下所有的注解都应用此行为
public @interface RestController {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 * @since 4.0.1
	 */
	@AliasFor(annotation = Controller.class)
	String value() default "";

}
```

**结论**：`@RestController` = `@Controller` + `@ResponseBody`。

- `@Controller` 负责将该类注册为 Spring IoC 容器中的 Bean，并启用 MVC 的请求映射扫描。
- `@ResponseBody` 加在**类级别**上，意味着该类下的**所有处理方法的返回值**，都会自动写入 HTTP 响应体（Response Body）中。
``
