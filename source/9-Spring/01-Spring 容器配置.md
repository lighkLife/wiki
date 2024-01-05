# Spring 容器配置

## 基于注解的容器配置
- `@Autowired` ：自动注入，可设置在字段（支持数组和集合）、构造函数、setter 方法、任意名称的多个参数的方法上
- `@Primary`： 指定优先注入候选者
- `@Qualifier`：明确指定注入候选者，与 `@Autowired` 一起使用
- `@Resource`：通过 `name` 指定注入候选者，可设置在字段（支持数组和集合）、单参数的 setter 方法上
- `@Value`
- `@PostConstruct` 和 `@PreDestroy`

本质上都是基于容器扩展点实现，如：
- `ConfigurationClassPostProcessor` （BeanFactoryPostProcessor）
- `AutowiredAnnotationBeanPostProcessor` （BeanPostProcessor）
- `CommonAnnotationBeanPostProcessor` （BeanPostProcessor）
- `PersistenceAnnotationBeanPostProcessor`（BeanPostProcessor）
- `EventListenerMethodProcessor`（BeanFactoryPostProcessor）
  
## 基于`Java`的容器配置

使用`AnnotationConfigApplicationContext`即可支持`@Bean` 和 `@Configuration` 进行容器配置。
```{caution}
默认情况下，使用 Java 配置定义的具有公共 close 或 shutdown 方法的 bean 会自动注册销毁回调。如果您有公共 close 或 shutdown 方法，并且您不希望在容器关闭时调用它，则可以将 @Bean(destroyMethod = "") 添加到您的 bean定义禁用默认的 (inferred) 模式。
```

## Classpath 扫描和 Components 管理

`@Component` 是元注解，@Repository 、 @Service 和 @Controller 是使用 @Component 标记的注解。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

	// ...
}
```

常规 Spring 组件中的 @Bean 方法的处理方式与 Spring @Configuration 类中对应方法的处理方式不同。
@Configuration 标记的类会使用 CGLIB 进行增强，来拦截方法和字段的调用。
CGLIB 代理调用 @Configuration 类中的 @Bean 方法中的方法或字段创建对协作对象。
```java
// @Component
@Configuration
public class FactoryMethodComponent {

	@Bean @Scope("prototype")
	public TestBean prototypeInstance(InjectionPoint injectionPoint) {
		return new TestBean("prototypeInstance for " + injectionPoint.getMember());
	}
}
```

