# ThreadLcoal 概述

`ThreadLocal` 为线程提供了私有状态存放位置，可以简单的看作每个线程独享一个对象。

![thread-local](./img/thread-local.png)

## 使用方式

```Java
// ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "nobody");=
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("zhangsan");
String name = threadLocal.get();
threadLocal.remove();
```

## 使用场景

`ThreadLocal` 其实是保证线程安全的一种设计思路，让状态保持在线程自己内部，避免发生线程安全问题。

因此经常用来作为环境上下文（Context）使用，比如：
- SpringMVC 中的 HttpServletRequest、HttpServletResponse
- Spring 中的事务
- JDBC Connection

## 内部结构

操作 ThreadLocal 中的状态（变量） 本质上是操作当前线程的 `threadLocals`（`ThreadLocal.ThreadLocalMap`） 属性,  `threadLocals` 是 `ThreadLocal` 类中定义的一个哈希表，**使用线性探测**处理哈希冲突，ThreadLocal 对象作为 Key, ThreadLocal 里存储的状态（变量）作为 Value。

![threadlocalmap](./img/threadlocalmap.png)

## 内存泄漏

ThreadLocalMap Entry 的 key 是

(哈希表处理冲突)[https://zh.wikipedia.org/zh-cn/%E5%93%88%E5%B8%8C%E8%A1%A8#%E5%A4%84%E7%90%86%E5%86%B2%E7%AA%81]