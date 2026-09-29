# 第11章 第4节 引用类型转换与instanceof

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1535)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 引用类型之间也能"转换",但它和int转double不是一回事——对象本身不会变，变的只是编译器看待这个引用的方式。这一节讲清楚向上转型为什么总是安全、向下转型为什么可能出错，以及如何用instanceof在转换前做检查，避免ClassCastException。

本节配套视频请到 [原文](https://www.logamee.com/course-learning/77/1535) 观看。

## 4.1 什么类型之间可以转换

```java
class Dog { }
class Cat { }

Cat cat = new Cat();
```

`Cat`类型的对象能否转换成`Dog`类型？答案是不能。

从现实来理解，一只狗无论如何也不可能变成一只猫。所以把猫转换成狗这种类型，很荒诞也没有意义。

猫类和动物类，即`Cat`类和`Animal`类之间就可以转换。我们可以说"猫是动物",这是没问题的。

<font color="red">**只有两个类型处于同一条继承链上，它们才能进行转换**</font>。两个无关的类不能进行转型(比如狗和猫)。

```java
class Animal { }
class Cat extends Animal { }

Animal animal = new Cat();  // 向上转型,自动转型,无需特别语法
```

`Cat`是子类，`Animal`是父类，子类→父类，称为<font color="red">**向上转型**</font>。

向上转型是自动的，因为猫肯定是一种动物，这很自然，所以无需特别的语法，直接将实例化的`Cat`对象赋值给`Animal`即可。

此外，任何类型都可以向上转型为`Object`,因为`Object`是所有类的基类：

```java
Object obj = new Animal();
Object obj1 = new Cat();
```

## 4.2 向下转型

既然有向上转型(Upcasting),那么也会存在向下转型(Downcasting),即父类→子类。但向下转型不是自动转型，且有可能会出现异常。

先讨论为什么向下转型有可能会出现错误。从现实来理解：`Animal`是一种动物，但动物有很多种，未必就一定是猫，所以将动物强行转换成猫就会出错。

```java
class Animal { }
class Cat extends Animal { }
class Dog extends Animal { }

Animal animal = new Animal();
Cat cat = (Cat)animal; // 错误!Animal实际上并不是猫
```

向下转型需要<font color="red">**父类的实际类型必须是子类才能转换**</font>:

```java
class Animal { }
class Cat extends Animal { }
class Dog extends Animal { }

Animal animal = new Cat();  // Animal实际类型是Cat
Cat cat = (Cat)animal;      // 向下转换成功,将Animal还原成Cat
Dog dog = (Dog)animal;      // 失败!animal实际类型本身就不是Dog类
```

## 4.3 编译时错误和运行时错误

Java程序的错误主要分为编译时错误和运行时错误。

Java程序从写完代码到成功运行需要分为两步：

- 先编译。源代码编译成字节码(`.class`文件)
- 只有字节码才能被运行

<font color="red">**编译时错误**</font>:在编译阶段产生的错误。在IDE源码里出现红色错误提示的大多数都是编译时错误：

```java
int a = 3      // 编译时错误,语句没有以;号结尾
int a = 3.3;   // 编译时错误,将double类型赋值给了整型
```

<font color="red">**运行时错误**</font>:程序在编译成功后，进入运行阶段才发现的错误。典型的是`NullPointerException`(空指针异常):

```java
String str = null;
int length = str.length(); // 试图在null引用上调用方法
```

编译时错误不可怕，真正可怕的是运行时错误。因为运行时错误隐藏比较深，往往是由许多行代码连续运行导致的。

## 4.4 类型检测：instanceof操作符

向上转型是安全的，比较麻烦的是向下转型——向下转型如果失败，会发生运行时错误。为了避免运行时错误，有必要在进行向下转型前，检查父类的真实类型到底是什么。

`instanceof`是Java中的二元操作符，用于在运行时检查一个对象是否属于特定类型。它返回一个布尔值(`true`或`false`):

```java
String text = "Hello";
boolean isString = text instanceof String;   // true
boolean isObj = text instanceof Object;      // true(所有类都继承自Object)
boolean isInt = text instanceof Integer;     // false
```

若对象为`null`,始终返回`false`:

```java
String nullStr = null;
boolean isString = nullStr instanceof String;  // false
```

`instanceof`的典型使用场景是在向下转型时进行安全检查：

```java
class Animal { }
class Cat extends Animal { }
class Dog extends Animal { }

Animal animal = new Cat();

boolean isDog = animal instanceof Dog; // false,animal真实类型是Cat
boolean isCat = animal instanceof Cat; // true

if (animal instanceof Cat) {    // 先判断
    Cat cat = (Cat) animal;     // 确定后再向下转型
    cat.toString();
}
```

## 4.5 Java 16+的模式匹配instanceof

Java 16引入了模式匹配`instanceof`,允许在检查类型的同时直接声明类型转换后的变量：

```java
Object obj = "Pattern Matching";

// 传统写法
if (obj instanceof String) {
    String s = (String)obj;
    s.toUpperCase();
}

// 简化写法(Java 16+)
if (obj instanceof String s) { // 若检查结果为true,声明s并转型
    s.toUpperCase();            // 直接使用s
}
```

## 4.6 向上转型与向下转型对比

| 特性 | 向上转型 | 向下转型 |
|------|----------|----------|
| 方向 | 子类→父类 | 父类→子类 |
| 安全性 | 总是安全 | 需要检查 |
| 语法 | 自动隐式 | 显式，语法：(type) |
| 信息丢失 | 丢失子类特有成员 | 恢复子类成员 |
| 检查方式 | 不需要 | instanceof |

向上转型后，子类特有的成员变量和方法会"丢失":

```java
class Animal { }
class Cat extends Animal {
    public String name = "名字";
}

Cat cat = new Cat();
String name = cat.name;          // 没问题,Cat有name成员变量

Animal animal = new Cat();
String name1 = animal.name;      // 错误!向上转型后,name将丢失

Cat cat2 = (Cat)animal;
String name2 = cat2.name;        // 没问题,向下转型后,name成员变量又恢复了
```

## 4.7 类型转换的实质

在Java中，引用类型转换的实质是<font color="red">**改变Java编译器对引用变量的类型认知，而不改变堆内存中的实际对象**</font>。

可以把它看作一个识别过程。程序里有一个对象，它在堆内存中始终是原来的实际类型。你用一个父类引用去接收它，编译器就按父类类型看待这个引用；你显式转回子类引用，编译器就恢复对子类成员的认知。对象本身没有变化，变化的是编译器能够看到哪些成员。

向上转型后，子类特有的成员暂时不可见，这就是前面说的"信息丢失"。向下转型恢复子类引用后，这些成员又能被访问，信息也随之恢复。

---

---

## 4.8 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 向上转型 | Upcasting | /ʌpˈkɑːstɪŋ/ | 子类自动转换为父类，总是安全 |
| 向下转型 | Downcasting | /daʊnˈkɑːstɪŋ/ | 父类显式转换为子类，需要instanceof检查 |
| 类型转换 | Type Casting | /taɪp ˈkɑːstɪŋ/ | 改变变量类型的认知，不改变实际对象 |
| 运行时检查 | Runtime Check | /ˈrʌntaɪm tʃek/ | 在程序运行时进行的类型安全检查 |
| 模式匹配 | Pattern Matching | /ˈpætən ˈmætʃɪŋ/ | Java 16+引入的instanceof增强特性 |
| 编译时错误 | Compile-time Error | /kəmˈpaɪl taɪm ˈerə(r)/ | 编译阶段就能发现的错误 |
| 运行时错误 | Runtime Error | /ˈrʌntaɪm ˈerə(r)/ | 程序运行阶段才出现的错误 |

## 4.9 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1535) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-video{…}` | 改为到课程页观看 |
| `@logicframe-appreciate` | 赞赏入口略去（飞书无法渲染） |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
