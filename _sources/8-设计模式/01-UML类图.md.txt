# UML 类图

## 介绍

类封装了数据和行为, 其中类属性表达数据，类方法表达行为。类图主要用来展示类之间的关系，当然也支持描述类属性和方法。其中，类之间的关系主要包含以下四种：
- 关联关系
- 依赖关系  
- 泛化关系
- 接口与实现关系

```{uml}
@startuml
class Company {
    - name： String
}

class  Game {
    - name: String
    - creater：Company
    + run(): void
}

class Boy {
    + name: String
    # play(Game): int
}
Game <.. Boy::play
Company <-- Game::creater
@enduml
```

## 类定义
  
以 [plantuml](https://plantuml.com/zh/class-diagram) 为例子，说明类定义方式。

```
@startuml
abstract        abstract
abstract class  "abstract class"
annotation      annotation
circle          circle
()              circle_short_form
class           class
class           class_stereo  <<stereotype>>
diamond         diamond
<>              diamond_short_form
entity          entity
enum            enum
exception       exception
interface       interface
metaclass       metaclass
protocol        protocol
stereotype      stereotype
struct          struct
@enduml
```

```{uml}
@startuml
abstract        abstract
abstract class  "abstract class"
annotation      annotation
circle          circle
()              circle_short_form
class           class
class           class_stereo  <<stereotype>>
diamond         diamond
<>              diamond_short_form
entity          entity
enum            enum
exception       exception
interface       interface
metaclass       metaclass
protocol        protocol
stereotype      stereotype
struct          struct
@enduml
```

| **字符** | **图标(属性)** | **图标(方法)** | **可访问性** |
| --- | --- | --- | --- |
| `-` | ![](https://plantuml.com/img/private-field.png) | ![](https://plantuml.com/img/private-method.png) | `private` 私有 |
| `#` | ![](https://plantuml.com/img/protected-field.png) | ![](https://plantuml.com/img/protected-method.png) | `protected` 受保护 |
| `~` | ![](https://plantuml.com/img/package-private-field.png) | ![](https://plantuml.com/img/package-private-method.png) | `package private` 包内可见 |
| `+` | ![](https://plantuml.com/img/public-field.png) | ![](https://plantuml.com/img/public-method.png) | `public` 公有 |

## 关联关系
关联（Association）关系是类与类之间最常用的一种关系，它是一种结构化关系，用于表示一类对象与
另一类对象之间有联系，如汽车和轮胎、师傅和徒弟、班级和学生等。

`Strudent <-- Class::students`
```{uml}
class Class {
   - students: Strudent[]
}

class Strudent {
   - name: String
}

Strudent <-- Class::students
```

## 依赖关系
依赖（Dependency）关系是一种使用关系，特定事物的改变有可能会影响到使用该事物的其他事物，
在需要表示一个事物使用另一个事物时使用依赖关系。大多数情况下，依赖关系体现在某个类的方法
使用另一个类的对象作为参数

`Car <.. Driver::driver`
```{uml}
class Driver {
    + dirver(Car car): void
}

class Car {
    + move(): void
}

Car <.. Driver::driver
```

## 泛化关系
泛化（Generalization）关系也就是继承关系，用于描述父类与子类之间的关系.

`Person <|-- Student`
```{uml}
class Person {
    # name: String
    # age : int
    + move() : void
    + say()  : void 
}

class Student {
    - studentNo : String
    + study() : void
}

class Teacher {
    - teacherNo : String
    + teach() : void
}

Person <|-- Student
Person <|-- Teacher
```

## 接口与实现关系
类实现了接口，类中的操作实现了接口中所声明的操作。

`Animal <|.. Panda::eat`
```{uml}
Interface Animal {
    + eat(): void
}

Class Human {
    + eat: void
}

Class Panda {
    + eat: void
}

Animal <|.. Human::eat
Animal <|.. Panda::eat
```