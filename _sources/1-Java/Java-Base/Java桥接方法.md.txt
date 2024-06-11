# Java 桥接方法

## 什么是的 Java 桥接方法

Java中的桥接方法（Bridge Method）是一种为了实现某些Java语言特性而由编译器自动生成的方法。
可以通过 Method 类的 `isBridge` 方法来判断一个方法是否是桥接方法。
在字节码文件中，桥接方法会被标记为 `ACC_BRIDGE` 和 `ACC_SYNTHETIC`.

```{note}
- `ACC_BRIDGE`： 这个方法是由编译生成的桥接方法。
- `ACC_SYNTHETIC`： 这个方法是由编译器生成，并且不会在源代码中出现。
```

桥接方法最常见的情况有：
- 协变返回值类型(子类与父类方法返回值不一致)
- 类型擦除(子类与父类方法参数或返回值不一致)
- 改变基类可见性

## 协变返回值类型
在面向对象程序设计中，实例函数的协变返回值类型指的是：
子类中的成员函数的返回值类型不必严格等同与该函数所重写的父类中的函数的返回值类型，而可以是更 "狭窄" 的类型。

例如：
```Java
public class Root {
    public Number getValue(){
        return 0;
    }
}

// 协变返回值类型，编译时**会**生成桥接方法
public class A extends Root{
    @Override
    public Integer getValue() {
        return 1;
    }
}

// 返回结果与父类相同，编译时**不会**生成桥接方法
public class B extends Root{
    @Override
    public Number getValue() {
        return 2;
    }
}
```

直接使用 `javac Root.java A.java B.clasjava` 编译上面三个类，再使用`javap -v A.class` 查看，
可以发现 `A.class` 中多一个编译生成的桥接方法 `public java.lang.Number getValue();`：

{lineno-start=1 emphasize-lines="1, 12, 18"}
```
  public java.lang.Integer getValue();
    descriptor: ()Ljava/lang/Integer;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: iconst_1
         1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         4: areturn
      LineNumberTable:
        line 6: 0

  public java.lang.Number getValue();
    descriptor: ()Ljava/lang/Number;
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokevirtual #3                  // Method getValue:()Ljava/lang/Integer;
         4: areturn
      LineNumberTable:
        line 3: 0
```

在JVM方法中，返回类型也是方法签名的一部分，而桥接方法的签名和其父类的方法签名一致，桥接方法内部调用子类的实现，
以此就实现了协变返回值类型。

经过测试 Java8 测试，下面的方法调用都会被编译为：直接调用子类的方法实现，`Number value2 = a.getValue();`
并不会调用桥接方法 `public java.lang.Number getValue();`，所以这种情况生成桥接方法，在 Java 中并没有
起到作用，推测是为了兼容其他 JVM 上的语言来实现协变返回值类型。
```Java
    A a = new A();
    Integer value1 = a.getValue(); //public java.lang.Integer getValue();
    Number value2 = a.getValue();  //public java.lang.Integer getValue();
```

## 类型擦除

### 方法参数为泛型
```Java
public class Root<T> {
    public T val;
    public T getValue(){
        return val;
    }
}

// 范型具化，编译时会进行类型擦除，父类类型擦除后与子类的方法签名不同，因此会生成桥接方法
public class A extends Root<Integer>{
    @Override
    public void set(Integer integer) {
        val = integer;
    }
}

// 类型擦除后的方法签名与父类相同，因此不会生成桥接方法
public class B<T> extends Root<T>{
    @Override
    public void set(T t) {
        val = t;
    }
}
```
查看 A.class 文件进行确认：

{lineno-start=1 emphasize-lines="1, 14, 18, 21, 22"}
```
  public void set(java.lang.Integer);
    descriptor: (Ljava/lang/Integer;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #2                  // Field val:Ljava/lang/Object;
         5: return
      LineNumberTable:
        line 6: 0
        line 7: 5

  public void set(java.lang.Object);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: checkcast     #3                  // class java/lang/Integer
         5: invokevirtual #4                  // Method set:(Ljava/lang/Integer;)V
         8: return
      LineNumberTable:
        line 3: 0
```
可以看到多了一个 `public void set(java.lang.Object);` 桥接方法，桥接方法内部会先进行类型检查
（ 2: checkcast     #3`），再调用类型具化后的方法`public void set(java.lang.Object);`
（5: invokevirtual #4）。可见，再泛型系统中，桥接方法可以进行编译时类型检查。

### 方法返回结果为泛型

```Java
public class Root<T> {
    public T val;
    public T getValue(){
        return val;
    }
}

// 范型具化，编译时会进行类型擦除，父类类型擦除后与子类的方法签名不同，因此会生成桥接方法
public class A extends Root<Integer> {

    @Override
    public Integer getValue() {
        return 1;
    }
}

// 类型擦除后的方法签名与父类相同，因此不会生成桥接方法
public class B<T> extends Root<T> {

    @Override
    public T getValue() {
        return val;
    }
}
```

查看 A.class 验证

{lineno-start=1 emphasize-lines="1, 12, 18"}
```
  public java.lang.Integer getValue();
    descriptor: ()Ljava/lang/Integer;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: iconst_1
         1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         4: areturn
      LineNumberTable:
        line 7: 0

  public java.lang.Object getValue();
    descriptor: ()Ljava/lang/Object;
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokevirtual #3                  // Method getValue:()Ljava/lang/Integer;
         4: areturn
      LineNumberTable:
        line 3: 0
```

## 改变基类可见性

父类可见新与子类的可见性不同时，为了保证父类的方法可以正常调用，会为子类生成桥接方法，在桥接方法内部调用父类方法。

```Java
class Root {
    public void hello() {
        System.out.println("hello");
    }
}

// 父类的方法可见新与子类覆盖后的可见性不同，生成桥接方法
public class A extends Root {
   
}

public class B extends Root {
    @Override
    public void hello() {
        super.hello();
    }
}
```
可以查看 A.class 确认

{lineno-start=1 emphasize-lines="7"}
```
  public void hello();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #5                  // Method org/apache/ibatis/temp/case1/Root.hello:()V
         4: return
      LineNumberTable:
        line 3: 0
```

如果有客户端代码如下，就可以确保其正常运行：
```Java
    public static void main(String[] args) {
        A a = new A();
        a.hello(); // 实际调用的是桥接方法
        Root root = a;
        root.hello(); // 实际调用的是父类方法，但因为可见性，只能在父类包中访问
    }
```

因此，桥接方法还保证了 Java 中继承与方法可见性的正常工作
