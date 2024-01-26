# SpEL

## 使用 SpEL 
```Java
// 方式一
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); 
String message = (String) exp.getValue();

// 方式二
@Value("#{ systemProperties['user.region'] }")
private String defaultLocale;
```

SpEL 支持两种 `EvaluationContext` 用于计算表达式（解析属性、方法、字段，执行类型转换）：
- `SimpleEvaluationContext` 
- `StandardEvaluationContext`

`SimpleEvaluationContext` 是 SpEL 语言语法的子集，不包括 Java 类型引用、构造函数和 bean 引用，
还需要显式选择表达式中属性和方法的支持级别。

可以使用解析器配置对象 ( `SpelParserConfiguration` ) 来配置 SpEL 表达式解析器。可以实现：
- 编译模式
- ClassLoader
- 是否自动为 null 引用创建对象
- 是否自动增加长集合大小
- 集合自动增加的极限

表达式长度最大为 10,000 个字符，可以通过参数 `spring.context.expression.maxLength` 修改。

## 编译表达式

可以通过 `spring.expression.compiler.mode` 配置设置模式

```Java
public enum SpelCompilerMode {

	/**
	 * The compiler is switched off; this is the default.
	 */
	OFF,

	/**
	 * In immediate mode, expressions are compiled as soon as possible (usually after 1 interpreted run).
	 * If a compiled expression fails it will throw an exception to the caller.
	 */
	IMMEDIATE,

	/**
	 * In mixed mode, expression evaluation silently switches between interpreted and compiled over time.
	 * After a number of runs the expression gets compiled. If it later fails (possibly due to inferred
	 * type information changing) then that will be caught internally and the system switches back to
	 * interpreted mode. It may subsequently compile it again later.
	 */
	MIXED

}
```

IMMEDIATE 模式之所以存在，是因为 MIXED 模式可能会导致具有副作用的表达式出现问题。如果编译表达式在部分成功后崩溃，则它可能已经做了一些影响系统状态的事情。如果发生这种情况，调用者可能不希望它以解释模式静默地重新运行，因为表达式的一部分可能会运行两次。

无法编译以下类型的表达式：
- 包含赋值
- 依赖转换服务（conversion）
- 使用自定义的 resolvers 或 accessors
- 使用集合选择器或投影（ selection or projection）

## SpEL语法

- 对象属性： `user.name.firstName`
- 数组、列表访问： `array[3]`
- map 访问： `map['key1']`
- 创建内部列表： `{1,2,3,4}`, `{{'a','b'},{'x','y'}}`
- 创建内部映射： `{name:'Nikola',dob:'10-July-1856'}`，`{:}` 代表空映射
- 方法调用：`'abc'.substring(1, 3)`，`isMember('Mihajlo Pupin')`
- 赋值：
  - `parser.parseExpression("name").setValue(context, inventor, "Aleksandar Seovic")`
  - `parser.parseExpression("name = 'Aleksandar Seovic'").getValue(context, inventor, String.class)`

- 变量：`newPrimes = #primes`，内置变量`#this`和`#root`
  - `#primes.?[#this > 10]`
  - `#root.primes`
  - 需要先设置`context.setVariable("primes", primes);`

- 函数：`#reverseString('hello')`
  - 需要先注册，`context.setVariable("reverseString",StringUtils.class.getDeclaredMethod("reverseString", String.class));`

- 引用 Bean：`@serviceImplA`
  - 需要设置 `context.setBeanResolver(new BeanFactoryResolver(beanFactory));`
- Elvis 操作符：`name?:'Unknown'`
  - 等价于 `name != null ? name : "Unknown"`（像猫王的发型，所以叫这个名字，🤯）
- 安全访问符： `placeOfBirth?.city`
- 集合选择器： `members.?[nationality == 'Serbian']`， `map.?[value<27]`（key 和 value）
- 集合投影： `"members.![placeOfBirth.city]`（等价于 Java 的 stream().map(...).collect()）
- 表达式模板： `random number is #{T(java.lang.Math).random()}`
        


