# Resource

## 内置的一些实现

- UrlResource
- ClassPathResource
- FileSystemResource
- PathResource
- ServletContextResource
- InputStreamResource
- ByteArrayResource

## 使用 `ResourceLoader` 的实现获取资源

```Java
public interface ResourceLoader {
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```
所有的 ApplicationContext 都实现了 ResourceLoader 接口，因此所有的 ApplicationContext 都可以用来获取资源实例。在一个特定的 ApplicationContext 上调用 getResource()，并且指定的位置路径没有特定的前缀，就会得到一个适合该特定 ApplicationContext 的 Resource （如 ClassPathXmlApplicationContext -> ClassPathResource）。也可以指定前缀将 String 对象转换为特定 Resource ，具体策略如下：

| 前缀         | 示例                               | 说明                           |
|------------|----------------------------------|------------------------------|
| classpath: | `classpath:com/myapp/config.xml` | 从classpath加载。                |
| file:      | `file:///data/config.xml`        | 作为 `URL` 从文件系统加载。            |
| https:     | `https://myserver/logo.png`      | 以 `URL` 形式加载。                |
| (none)     | `/data/config.xml`               | 取决于底层的 `ApplicationContext'。 |

## `ResourcePatternResolver` 处理 `location pattern`

```Java
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
	Resource[] getResources(String locationPattern) throws IOException;
}
```
ResourcePatternResolver 支持解析位置模式，如 Ant-style path pattern。标准 ApplicationContext 中的默认 ResourceLoader 实际上是实现了 ResourcePatternResolver 接口的 PathMatchingResourcePatternResolver。

## 获取 `ResourceLoader`

可以通过多种方式获取 `ResourceLoader`，继而获取资源，如
- 自动注入 `ResourceLoader`
- 实现 `ResourceLoaderAware` 接口
- 实现 `ApplicationContextAware` 接口

```Java
public interface ResourceLoaderAware {
	void setResourceLoader(ResourceLoader resourceLoader);
}
```

```{note}
- 可以通过实现 `ApplicationContextAware`，通过`ApplicationContext`来获取资源（因为`ApplicationContext`都实现了`ResourceLoader`接口）。
- 如需通过通配符或特殊的`classpath*`加载多个资源，请使用`ResourcePatternResolver`。
```

## Resource 作为依赖

如果资源时动态的，那么适合使用 `ResourceLoader` 来加载，但是，如果资源是静态的，那么可以借助`@Value`来获取资源，特殊的 `PropertyEditor` 会将字符串转换为 `Resource` 对象，如：
```Java
@Component
public class MyBean {
	private final Resource template;
	public MyBean(@Value("${template.path}") Resource template) {
		this.template = template;
	}
}
```

需通过通配符或特殊的 `classpath*` 加载多个资源，如 `templates.path=classpath*:/config/templates/*.txt`
```Java
@Component
public class MyBean {
	private final Resource[] templates;
	public MyBean(@Value("${templates.path}") Resource[] templates) {
		this.templates = templates;
	}
}
```



