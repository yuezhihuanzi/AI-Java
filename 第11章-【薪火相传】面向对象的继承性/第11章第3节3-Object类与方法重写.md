# 第11章 第3节 Object类与方法重写

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1534)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 每个类都有一个看不见的共同祖先——Object。这一节先认识它，看看toString()、equals()、hashCode()这几个方法为什么重要，以及它们默认的实现为什么常常不满足我们的需求；接着讲方法重写：子类可以重新定义父类的方法，但签名、访问权限都有规则，重写也是后面理解多态的起点。

本节配套视频请到 [原文](https://www.logamee.com/course-learning/77/1534) 观看。

## 3.1 一切类的根：Object

在Java中，`java.lang.Object`是所有类的根类。也就是说，Java中所有的类都直接或间接继承自`Object`类。

如果一个类没有显式声明继承自其他类，那么它默认继承自`Object`:

```java
// 没有继承任何父类
class Cat { }

// 相当于
class Cat extends Object { }
```

为什么需要让所有类都有一个共同的根类？

如果所有Java类都继承自`Object`,那么`Object`中所定义的任何方法都会被子类继承，从而保证所有的Java对象都有共有的行为。

`Object`类定义了11个方法(JDK 17),其中7个为非`final`方法(可被子类重写),4个为`final`方法(不可重写)。这些方法为所有Java对象提供了通用功能。以下是最常用的几个方法。

## 3.2 toString()方法

该方法的作用是返回对象的字符串表示形式。

`toString()`方法就像对象的"传记"一样，可以用文字(字符串)的形式来表达对象。由于它是根类`Object`里的`public`方法，所以所有Java类都可以继承这个方法。

```java
package com;
class Animal { }

Animal animal = new Animal();
String str = animal.toString(); // str = com.Animal@4fca772d
```

`Animal`里并没有`toString()`方法，但是它的根类`Object`有。它默认将返回"完全限定类名@哈希码的十六进制"表示，如`com.Animal@4fca772d`。

默认返回值并不能很好地描述类的特征。所以`toString()`方法通常都会被<font color="red">**重写**</font>。

## 3.3 equals(Object obj)方法

```java
public boolean equals(Object obj)
```

其作用是判断对象逻辑上是否相等(非地址相等)。

`equals()`方法默认情况下判断的是两个对象的地址是否相等：

```java
class Cat { }

Cat cat1 = new Cat();
Cat cat2 = new Cat();
boolean isEqual = cat1.equals(cat2); // false,默认比较的是地址
```

但有时候，我们期望的相等可能不是地址相等，而是对象"内容"相等：

- 可能是两个人的名字和年龄相同，就认为是相等
- 可能是两碗米饭的分量和大米品种相同，就认为是相等

也就是说，两个对象是否相等，取决于我们自己设定的判断条件：

```java
class Cat {
    public String name;
    public int age;
    public Cat(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Cat cat1 = new Cat("姜磺", 3);
Cat cat2 = new Cat("姜磺", 3);
boolean isEqual = cat1.equals(cat2); // false,默认比较的是地址
```

目前学习到的两个判等方式：

- `==`符号。通常用来比较两个基本类型，很少用来比较两个引用类型
- `equals()`可以用来比较两个引用类型，但默认情况下比较的也是两个对象的地址，和`==`符号一样的作用

`equals()`大多数情况下都需要被<font color="red">**重写**</font>,才能实现按内容比较的需求。

## 3.4 hashCode()

```java
public int hashCode()
```

`hashCode()`方法返回的哈希码(一个32位整数)主要用于提高哈希表(如`HashMap`、`HashSet`)的性能。

Java要求同时重写`hashCode()`和`equals()`,并遵守以下规则：

<font color="red">**规则1**</font>: 如果`a.equals(b) == true`,则`a.hashCode() == b.hashCode()`必须成立。否则会导致相同的对象在哈希表中存储到不同位置。

<font color="red">**规则2**</font>: 如果`a.equals(b) == false`,可以允许`a.hashCode() == b.hashCode()`(被称为哈希冲突)。虽然允许，但还是应该尽量让不同对象返回不同哈希码以减少冲突。

## 3.5 clone()——浅拷贝

该方法的作用是创建对象的浅拷贝副本。简单来说，就是将1个对象再复制(拷贝)1份。

复制分为"浅拷贝"和"深拷贝"。

<font color="red">**浅拷贝**</font>(shallow copy)是指它会创建一个新对象，并将原对象中每一个字段的值直接复制到新对象：

- 对于基本类型(如`int`、`double`),直接复制值
- 对于引用类型，复制的是引用地址，而不是引用指向的对象本身。这意味着原对象和克隆对象会共享这个引用类型对象

```java
class House {
    String tv;
    House(String tv) {
        this.tv = tv;
    }
}

// 必须实现Cloneable的接口
class Person implements Cloneable {
    String name;       // 基本类型,可以复制出两份独立的数据
    House house;       // 引用类型,只复制引用地址

    public Person(String name, House house) {
        this.name = name;
        this.house = house;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone(); // 浅拷贝
    }
}

// 使用示例
Person original = new Person("Lily", new House("Netflix"));
Person cloned = (Person) original.clone();

cloned.name = "Lucy";
cloned.house.tv = "Apple";

System.out.println(original.name);       // 打印"Lily"
System.out.println(original.house.tv);   // 打印"Apple"!
```

修改拷贝对象`cloned`的`name`不会影响源对象`original`的`name`,因为基本类型是独立拷贝。但修改`cloned.house.tv`会影响`original.house.tv`,因为引用类型只是复制了"钥匙",房子还是同一个。

<font color="red">**深拷贝**</font>(deep copy)则是递归地复制所有引用类型字段所指向的对象，让原对象和拷贝对象完全独立。这需要重写`clone()`方法并手动复制每个引用类型成员。

---

---

子类可以使用`@Override`注解，重新定义父类的方法，实现子类自己的行为。

怎么理解"重写"?有两种情况父类的方法需要被重写。

## 3.6 第一种：父类的方法只定义了"空"方法

父类只实现一个"空"的方法(没有内容),这种做法有意义吗？太有意义了。

很多时候父类的方法只是一种"契约"或者说"表明能做什么事儿",但父类并不负责具体怎么去做这件事儿。

- 父类：我能干这件事儿，但我不管怎么去做这件事儿
- 子类1:我来负责具体干这件事儿，我知道怎么做
- 子类2:我也负责具体干这件事儿，我也知道怎么做

举个现实里的例子：

- 父类：交通工具
- 子类1:车
- 子类2:飞机

```
交通工具(父类):我能从北京到上海,但我只是说我有这个能力
车(子类1):从北京到上海,我可以走高速公路
飞机(子类2):我直接飞过去
```

```java
class Vehicle {
    // 空方法,没有任何实现,子类需要重写
    public void move() { }
}

class Car extends Vehicle {
    @Override
    public void move() {
        System.out.println("汽车在公路上行驶");
    }
}

class Airplane extends Vehicle {
    @Override
    public void move() {
        System.out.println("飞机在空中飞行");
    }
}
```

重写方法上的`@Override`注解虽然不是必须的，但推荐使用，可以帮助编译器检查是否正确重写。

## 3.7 第二种：父类已实现，但子类不满意

父类虽然具体实现了某种功能，但子类决定自己重新实现：

```java
class Plane {
    public void move() {
        System.out.println("飞行");
        System.out.println("普通飞机需要3小时");
    }
}

class JetPlane extends Plane {
    @Override
    public void move() {
        System.out.println("飞行");
        System.out.println("喷气式飞机需要1小时");
    }
}
```

## 3.8 重写需要遵循的规则

<font color="red">**方法名、参数列表必须完全相同**</font>

```java
class Animal {
    void makeSound(String text) {
        System.out.println("Animal makes a sound" + text);
    }
}

class Cat extends Animal {
    @Override
    void makeSound(String text) {  // 参数列表必须同父类方法相同
        System.out.println("Meow" + text);
    }
}
```

<font color="red">**访问修饰符不能比父类更严格**</font>

```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Cat extends Animal {
    // 错误!父类是public,子类的protected范围更小,不能重写
    @Override
    protected void makeSound() {
        System.out.println("Meow");
    }
}
```

这里有一个原则——<font color="red">**里氏替换原则**</font>(Liskov Substitution Principle, LSP):

> 子类必须能够完全替代父类，而不破坏程序的正确性。

如果父类方法是`public`的，子类改成了`protected`,那么子类的方法缩小了使用范围，就有可能导致子类不能无缝替换父类。所以子类方法的访问范围不能比父类方法更严格。

<font color="red">**private、final和static的方法不能被重写**</font>

- `private`方法仅在父类内部可见，子类无法继承，自然谈不上"重写"
- `final`是Java保留关键字，用于声明方法禁止重写
- `static`修饰的方法是静态的，静态方法不能被重写

```java
class Animal {
    protected final void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Cat extends Animal {
    // 错误!重写失败,因为父类方法被final禁止重写
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}
```

<font color="red">**返回类型可以是父类方法返回类型的子类(协变)**</font>

```java
class Fruit { }
class Apple extends Fruit { }

class Basket {
    Fruit getFruit() { return new Fruit(); }
}

class AppleBasket extends Basket {
    @Override
    Apple getFruit() { // 合法,返回Fruit的子类Apple
        return new Apple();
    }
}
```

这种特性称为"协变"。反之"逆变"不可以——不能返回父类方法返回类型的父类。

| 场景 | 是否合法 | 说明 |
|------|----------|------|
| 相同基本类型 | 合法 | int → int |
| 不同基本类型 | 不合法 | int → double |
| 相同引用类型 | 合法 | String → String |
| void | 合法 | void → void 必须都是void |
| 返回子类(协变) | 合法 | Object → String |
| 返回父类(逆变) | 不合法 | String → Object |

## 3.9 重写toString()

`toString()` 是Java中最重要的方法之一。看一个例子：

```java
public class User {
    private String id;
    private String name;
    private String email;
    private int age;
    private String address;

    public User() {
    }

    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
}

User user = new User();
System.out.println(user);
```

`println`会调用`user`的`toString()`方法，但是如果不重写，默认将打印对象的完全限定名+哈希码，这个信息对我们没有用。我们希望打印出用户的简要信息。

重写`toString()`即可：

```java
@Override
public String toString() {
    return "User{" +
            "name='" + this.name + '\'' +
            ", email='" + this.email + '\'' +
            ", age=" + this.age +
            '}';
}

// 使用
User user = new User("Lily", "xxx@qq.com", 18);
System.out.println(user);
// 打印:User{name='Lily', email='xxx@qq.com', age=18}
```

## 3.10 重写equals()

`equals()`方法是Java对象身份识别的核心方法。默认逻辑是用`==`号进行比较，即比较两个对象的引用地址，并不是比较内容。

典型的需要重写`equals`的示例：

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

Point a = new Point(1, 1);
Point b = new Point(1, 1);
a.equals(b); // false!默认比较的是对象地址
```

很明显，(1,1)和(1,1)应该是同一个点，期望`equals()`能返回`true`。重写：

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;           // 自己和自己比,肯定相等
    if (o == null || getClass() != o.getClass()) return false;

    Point point = (Point) o;
    return x == point.x && y == point.y;  // 比较内容
}
```

<font color="red">**equals()重写规范**</font>:

- <font color="red">**自反性**</font>:`x.equals(x)`必须返回`true`
- <font color="red">**对称性**</font>:`x.equals(y)`与`y.equals(x)`结果必须一致
- <font color="red">**传递性**</font>:如果`x.equals(y)`且`y.equals(z)`,则`x.equals(z)`必须为`true`
- <font color="red">**一致性**</font>:多次调用结果必须一致(不依赖可变状态)
- <font color="red">**非空性**</font>:`x.equals(null)`必须返回`false`

<font color="red">**当对象作为集合元素时，不要修改影响equals()或hashCode()的字段**</font>,这会导致HashSet、HashMap等集合判断出错。

## 3.11 重写hashCode()

如果重写`equals()`,就必须重写`hashCode()`:

```java
@Override
public int hashCode() {
    return Objects.hash(name, email, age);
}

@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return age == user.age &&
           Objects.equals(name, user.name) &&
           Objects.equals(email, user.email);
}
```

必须遵守以下契约：

- 如果`x.equals(y) == true`,那么`x.hashCode()`必须和`y.hashCode()`返回相同的值
- 如果`x.hashCode() == y.hashCode()`,`x.equals(y)`不一定返回`true`(哈希冲突)
- 同一对象多次调用`hashCode()`应返回相同值

---

## 3.12 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 根类 | Root Class | /ruːt klɑːs/ | 所有类的共同祖先，Object是Java的根类 |
| 字符串表示 | String Representation | /strɪŋ ˌreprɪzenˈteɪʃn/ | toString()返回的对象的文本描述 |
| 哈希码 | Hash Code | /hæʃ kəʊd/ | 对象的32位整数摘要，用于哈希表快速定位 |
| 浅拷贝 | Shallow Copy | /ˈʃæləʊ ˈkɒpi/ | 只复制引用地址，原对象和拷贝对象共享引用 |
| 深拷贝 | Deep Copy | /diːp ˈkɒpi/ | 递归复制所有引用，原对象和拷贝对象完全独立 |
| 克隆 | Clone | /kləʊn/ | 创建对象的副本，clone()方法 |
| 标记接口 | Marker Interface | /ˈmɑːkə(r) ˈɪntəfeɪs/ | 内部没有任何方法的接口，如Cloneable仅作标记用 |

## 3.13 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1534) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-video{…}` | 改为到课程页观看 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
