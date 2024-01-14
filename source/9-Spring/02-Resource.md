| 前缀         | 示例                               | 说明                           |
|------------|----------------------------------|------------------------------|
| classpath: | `classpath:com/myapp/config.xml` | 从classpath加载。                |
| file:      | `file:///data/config.xml`        | 作为 `URL` 从文件系统加载。            |
| https:     | `https://myserver/logo.png`      | 以 `URL` 形式加载。                |
| (none)     | `/data/config.xml`               | 取决于底层的 `ApplicationContext'。 |