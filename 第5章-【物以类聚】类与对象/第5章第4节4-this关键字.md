# 第5章 第4节 this 关键字

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1499)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> <font color="blue">`this`</font> 在类内部代表「当前正在使用的那个对象」。它最常见的用途是区分同名的方法参数和成员变量，比如 `this.name = name`；也会出现在构造器链式调用里。这一节把 `this` 的作用和使用时机讲清楚——什么时候必须写、什么时候可以省略，写错地方编译器会立刻告诉你。

本节讲解视频请到 [课程页](https://www.logamee.com/course-learning/77/1499) 观看。

---

## 4.1 类内部的简化写法

上个小节，我们谈到访问对象的成员变量需要通过「对象.成员变量」的方式来访问。但来看一个例外：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        // 可以省略掉对象名,直接给成员变量赋值
        name = catName; 
        age = catAge; 
    }
}
```

上述代码第 6、第 7 两行同样是对成员变量 `name` 和 `age` 进行赋值操作。但这里并没有使用「对象.成员变量」的方式来访问。

上述代码并没有问题，实际上，这是一种简化写法。在类的内部，「对象.成员变量」的访问方式可以将对象名省略，直接使用成员变量名来访问。

## 4.2 为什么不能用 new 获取当前对象

那么，如果我们还是想通过对象名来访问应该怎么写？

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        Cat.name = catName; //错误
        Cat.age = catAge; //错误
    }
}
```

上述代码第 6、第 7 行是错误的写法，之前我们有讲到，不可以使用「类名.成员变量」的方式来访问。但如果我们想使用「对象.成员变量」的方式来访问，就需要有一个实例化的对象：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        Cat cat = new Cat(); //错误
        cat.name = catName; 
        cat.age = catAge; 
    }
}
```

上述代码有问题吗？有，而且它存在非常严重的问题。

我们的本意是实例化一个 `Cat` 对象，然后为其成员变量赋值。但这里特别的地方在于，第 6 行实际上是在 `Cat` 类的构造函数里实例化一个 `Cat` 对象。即，我们尝试在一个类的构造函数里实例化自身，这是不被 Java 允许的。

想想这样会带来什么问题：

- 当实例化 `Cat` 对象时，会调用 `Cat` 类的构造函数
- 由于在构造函数里又去实例化 `Cat` 对象，这又触发了类的构造函数
- 随后由于在构造函数里又去实例化 `Cat` 对象，这又触发了类的构造函数
- 随后由于在构造函数里又去实例化 `Cat` 对象，这又触发了类的构造函数
- ……（无限循环）

这是一个无法停止的自我复制，这就是 Java 不允许在构造函数里实例化自己的原因。

由于在构造函数里无法实例化自己，所以我们无法得到当前类的实例化对象。既然无法得到对象，那自然无法通过「对象.成员变量」的方式来访问成员变量。

## 4.3 this 指代当前对象

Java 考虑到了这个问题，所以特别提供了一个关键字：<font color="blue">`this`</font>。`this` 可以指代当前对象，无需再使用 `new` 操作符进行实例化：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        this.name = catName; //this指代当前的Cat类的实例对象。
        this.age = catAge;
    }
}
```

特别提醒同学们，<font color="red">**`this` 只能用于类的内部，不能在类的外部使用 `this`**</font>。

当然，正如我们之前看到的，`this` 是可以省略的：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        name = catName; 
        age = catAge;
    }
}
```

除了可以访问成员变量，`this` 还可以用来调用对象的方法：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        this.name = catName; 
        this.age = catAge;
        this.meow(); // 也可以用来访问方法
        meow();// this也可以省略
    }
    void meow(){
      System.out.println("meowwwwwww");
    }
}
```

如果是在类的外部可以使用以下代码来访问成员变量和方法：

```java
class Cat {
    String name; 
    int age;     

    Cat(String catName, int catAge) {
        // 内部使用this指代当前对象
        this.name = catName; 
        this.age = catAge;
        this.meow();
    }
    void meow(){
      System.out.println("meowwwwwww");
    }
}

// 在类的外部需要使用new得到对象
Cat cat = new Cat("大壮",2);
cat.age = 6;
cat.meow();
```

总结下，在类的外部可以使用 `new` 来得到类的对象；而在类的内部无需实例化，直接使用 `this` 指代当前类的对象。

## 4.4 什么时候不能省略 this

由于在类的内部，`this` 可以省略，这就涉及到是否应该省略掉 `this` 这个问题。在某些场景下，`this` 是不可以省略的：

```java
class Cat {
    String name;
    
    void setName(String name) {
        this.name = name; // 必须用 this 访问成员变量
    }
}
```

上述代码，必须使用 `this`。主要原因在于 `setName` 这个方法的形参也使用了 `name` 作为参数名，这就造成了方法形参名和 `Cat` 类的成员变量 `name` 同名。这在 Java 里是允许的。

但如果我们省略掉 `this`：

```java
class Cat {
    String name;
    
    void setName(String name) {
        name = "姜磺"; // 无法区分name到底是形参name还是成员变量name
    }
}
```

这会让 Java 无法区分 `name` 到底是方法形参 `name` 还是成员变量 `name`。正确的写法是：

```java
class Cat {
    String name;
    
    void setName(String name) {
        this.name = "姜磺"; // this.name 指向成员变量
    }
}
```

当然，其实还有不少地方是不能省略 `this` 关键字的，比如在构造函数里调用其他构造函数时，`this` 就不能省略。这里给出一个建议：

<font color="red">**尽量不要省略 `this` 关键字，因为这可以让我们一眼看出这是一个属于类的成员变量，而非方法形参或者局部变量。**</font>

---

## 4.5 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| this 关键字 | This | /ðɪs/ | 类内部指代当前对象的引用 |
| 变量遮蔽 | Shadowing | /ˈʃædoʊɪŋ/ | 形参与成员变量同名时，形参遮蔽成员变量的现象 |
| 构造器链式调用 | Constructor Chaining | /kənˈstrʌktə(r) ˈtʃeɪnɪŋ/ | 构造函数通过 `this()` 调用其他构造函数的写法 |
| 名称冲突 | Name Conflict | /neɪm ˈkɒnflɪkt/ | 多个名称指向同一作用域而产生的歧义 |

## 4.6 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1499) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` 阅读指南 | 普通引用块 |
| 「对象.成员」变量 | 「对象.成员变量」 |
| 什么什问题 | 什么问题 |
| `#6和#7` | 第 6、第 7（行号写法统一） |
| `print('meowwwwwww')` | `System.out.println("meowwwwwww");` |
| 在类的,外部 | 在类的外部 |
| 正确示例注释仍写「无法区分」 | `this.name` 指向成员变量 |
| `@logicframe-appreciate` | 赞赏入口略去（飞书无法渲染） |
| `@logicframe-video{25}` | 改为到课程页观看视频 |
| `@logicframe-question{…}` | 改为到课程页完成 |
