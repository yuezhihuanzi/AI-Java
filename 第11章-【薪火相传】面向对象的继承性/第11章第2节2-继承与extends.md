# 第11章 第2节 继承与extends

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1533)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 继承是面向对象里最容易理解错的概念——它不是生物学里"父生子"的关系，而是"一般到特殊"的is-a关系：狗是一种动物，经理是一种员工。这一节先用这个视角理解继承的本质，再学extends的写法，看看子类能从父类继承哪些成员、访问范围有什么限制，以及Java为什么不让一个类同时继承多个父类。

## 2.1 extends关键字

理解继承特性后，下一个问题是Java用什么语法来支持继承？

使用`extends`关键字可以让子类继承父类：

```java
// 父类
class ParentClass {
    // 父类成员
}

// 子类通过extends继承父类
class ChildClass extends ParentClass {
    // 子类成员
}
```

## 2.2 一个完整的继承案例

类的继承可以非常复杂。以现实世界里的"动物"关系为例：

```
动物(Animal)
    │
    ├── 脊椎动物(Vertebrate)
    │   │
    │   ├── 哺乳类(Mammal)
    │   │   │
    │   │   ├── 猫(Cat)
    │   │   │   │
    │   │   │   └── 家猫(DomesticCat)
    │   │   │
    │   │   └── 狗(Dog)
    │   │
    │   └── 鸟类(Bird)
    │       │
    │       └── 鹰(Eagle)
    │
    └── 无脊椎动物(Invertebrate)
        │
        └── 昆虫(Insect)
            │
            └── 蝴蝶(Butterfly)
            │
            └── 蚕(Bombyx)
```

上述共有5条继承链：

- Animal → Vertebrate → Mammal → Cat → DomesticCat
- Animal → Vertebrate → Mammal → Dog
- Animal → Vertebrate → Bird → Eagle
- Animal → Invertebrate → Insect → Butterfly
- Animal → Invertebrate → Insect → Bombyx

一个父类下面可以有多个子类。

## 2.3 子类的继承

子类自动获得父类的非私有成员(成员变量和方法),无需重复编写。

```java
class Animal {
    String name = "动物的名字";
    void eat() {
        System.out.println("eating");
    }
}

class Cat extends Animal {
    // 自动拥有name字段和eat()方法
    // Cat除了继承的方法,还可以有自己的方法
    void meow() {
        System.out.println(name + " says: Meow!");
    }
}
```

虽然子类`Cat`并没有显式写出`name`成员变量和`eat()`方法，但调用`cat.name`和`cat.eat()`依然可以生效。这是因为`Cat`继承了`Animal`的非私有成员变量和方法。

必须是非私有，即`public`、`default`和`protected`才有可能被继承。`private`的元素只能在类内部访问，不能被子类继承。

同时，继承也要遵循访问修饰符的访问范围：

```java
package com;
class Animal {
    String name = "动物的名字";
    void eat() {
        System.out.println("eating");
    }
}

package im;
import com.Animal;
// 错误!Cat和Animal不在一个包,且Animal并非public,所以无法继承
class Cat extends Animal {
    void meow() {
        System.out.println(" says: Meow!");
    }
}
```

再看一个案例：

```java
package com;
// public允许其他包的类继承Animal
public class Animal {
    private String name = "动物的名字";
    void eat() {
        System.out.println("eating");
    }
}

// Cat和Animal不在同一个包
package im;
// 可以继承Animal,因为Animal是public的
class Cat extends Animal {
    // 没有继承name,因为父类的name是private的
    // 没有继承eat()方法,因为eat()是default的,仅同包可继承
    void meow() {
        System.out.println(" says: Meow!");
    }
}
```

子类可以无限地继承下去——子类下面还有子子类：

```java
// 动物,父类
class Animal { }

// 猫类,子类通过extends继承动物类
class Cat extends Animal { }

// 英短,子子类继承自猫类
class ShortHair extends Cat { }
```

## 2.4 多重继承

一个子类可以有多个父类吗？

如果一个子类可以拥有多个父类，在面向对象里称为<font color="red">**多重继承**</font>(Multiple Inheritance)。

现实世界里，一个子类拥有多个父类是非常正常的现象。比如鸭嘴兽同时具备哺乳动物、鸟类和爬行动物的特征：

```
鸭嘴兽(Platypus)
    ├── 继承自 哺乳动物(Mammal)
    │   └── 特征:有毛发、哺乳
    ├── 继承自 鸟类(Bird)
    │   └── 特征:产卵、鸭嘴(类似喙)
    └── 继承自 爬行动物(Reptile)
        └── 特征:后肢有毒刺(类似某些蛇类)
```

不同编程语言对多重继承的态度差异较大：

| 语言 | 是否支持多继承 | 替代方案 |
|------|---------------|----------|
| C++ | 完全支持 | 虚继承 |
| Python | 完全支持 | MRO机制 |
| Java | 仅接口支持 | 接口继承 |
| C# | 仅接口支持 | 接口继承 |
| Go | 不支持 | 组合 |

主流语言里完全支持多重继承的只有C++和Python。Java不完全支持多重继承：在Java中，<font color="red">**类不支持多重继承**</font>,只有`interface`(接口)支持多重继承。

上述鸭嘴兽无法在Java里直接通过继承实现，但可以通过"多个类组合"或者"多重接口"的方式来实现。

> <font color="red">**为什么Java不支持多重继承？**</font>  
> 多重继承让类的继承变得非常复杂，容易出现"菱形继承问题"(Diamond Problem)。Java的前辈C++支持多重继承，但被普遍认为弊大于利。Java吸取这些经验，没有支持类的多重继承。

多重继承的菱形问题示例：

```java
// 猫类
class Cat { }

// 英短蓝猫
class BlueShorthair extends Cat { }

// 金吉拉
class ChinchillaGolden extends Cat { }

// 金渐层,由英短蓝猫和金吉拉繁育而成
// 构成菱形继承:
//            Cat
//       /             \
// BlueShorthair   ChinchillaGolden
//      \               /
//       GoldenShortHair
// 错误!Java不能同时继承2个父类
class GoldenShortHair extends BlueShorthair, ChinchillaGolden { }
```

---

## 2.5 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 继承 | Inheritance | /ɪnˈherɪtəns/ | 基于现有类创建新类，实现代码复用的机制 |
| 父类 | Parent Class / Super Class | /ˈpeərənt klɑːs/ | 被继承的类，也称超类、基类 |
| 子类 | Child Class / Sub Class | /tʃaɪld klɑːs/ | 继承其他类的类，也称派生类 |
| 基类 | Base Class | /beɪs klɑːs/ | 父类的另一种称谓，C++中使用较多 |
| 派生类 | Derived Class | /dɪˈraɪvd klɑːs/ | 子类的另一种称谓，C++中使用较多 |
| is-a关系 | is-a Relationship | /ɪz ə rɪˈleɪʃnʃɪp/ | 表示"是一种"的关系，是继承的核心概念 |

## 2.6 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1533) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
