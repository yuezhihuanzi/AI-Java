# 第12章 第3节 static关键字

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1540)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> `static`(静态)关键字让成员属于类本身，而不是某个实例对象。这一节先讲清楚这个根本区别，再看静态变量如何被所有对象共享、静态方法有什么使用限制，最后总结类内部访问实例成员和静态成员的正确写法。

## 3.1 认识static关键字

`static`(静态)关键字用于修饰类的成员变量、方法、代码块和嵌套类，使其属于<font color="red">**类本身**</font>而非类的实例。被`static`修饰的元素被称为静态变量、静态方法等。

记住一条非常重要的信息：<font color="red">**静态元素属于类本身，而不属于实例对象**</font>。能理解这一点才算真正理解了"静态"的意思。

## 3.2 静态成员变量

假设有一家连锁咖啡店，它有很多店面，但所有店面的营业时间都是相同的：

```java
class CoffeeShop {
    // 实例变量(每个对象都不一样)
    int id;          // 咖啡店编号
    String address;  // 地址

    // 静态变量(所有实例对象共有)
    static String openingHours = "8:00-22:00";

    public CoffeeShop(int id, String address) {
        this.id = id;
        this.address = address;
    }
}

// 每个对象都有自己的数据
CoffeeShop shop1 = new CoffeeShop(1, "xx路xx号");
CoffeeShop shop2 = new CoffeeShop(3, "kk路kk号");
```

不同于实例变量，静态变量通常是所有对象共享的成员变量。所有咖啡店因为是连锁的，所以它们的营业时间都是统一的，不会因为店铺不同而有所区别。

但实例变量不一样。实例变量通常是每个对象都不一样，比如咖啡店的编号和地址就会因为不同的咖啡店而有所不同。

静态变量的访问和修改无需通过实例对象，直接在类上修改即可：

```java
CoffeeShop shop2 = new CoffeeShop(3, "kk路kk号");
shop2.id = 5;                                // 实例变量需要通过实例对象来修改
CoffeeShop.openingHours = "9:00-24:00";      // 静态变量直接在类上修改
```

如果将"营业时间"设置为实例变量，无论是初始化还是修改数据都很麻烦——明明都是一样的数据，却要给每个对象都单独赋值。所以，如果所有对象都共享一个成员变量，它就应该被设置为`static`静态的。

## 3.3 静态方法

`static`除了能够修饰成员变量，还可以修饰方法，称为静态方法：

```java
class CoffeeShop {
    int id;
    String address;
    static String openingHours = "8:00-22:00";

    // 静态方法:修改营业时间
    public static void changeHours(String newHours) {
        CoffeeShop.openingHours = newHours;  // 只能访问静态变量
        System.out.println("营业时间已改为:" + newHours);
    }

    public CoffeeShop(int id, String address) {
        this.id = id;
        this.address = address;
    }
}

CoffeeShop.changeHours("9:00-24:00");
```

静态方法也是通过类来调用。

**注意**:静态方法内部只能访问静态成员变量，不能访问实例成员变量：

```java
public static void changeHours(String newHours) {
    CoffeeShop.openingHours = newHours;
    this.address = "xxxxx";  // 错误!静态方法不能使用this,不能访问实例变量
}
```

在实例方法中，可以访问类的静态变量：

```java
class CoffeeShop {
    static String openingHours = "8:00-22:00";

    public String getOpeningHours() {
        // 推荐带上类名
        return CoffeeShop.openingHours;
    }
}
```

<font color="red">**总结**</font>,在类的内部：

- 如果要访问实例变量，用`this.实例变量`
- 如果要访问静态变量，用`类名.静态变量`

---

## 3.4 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 静态 | Static | /ˈstætɪk/ | 属于类本身而非实例对象的修饰符 |
| 静态变量 | Static Variable | /ˈstætɪk ˈveəriəbl/ | 所有实例对象共享的成员变量 |
| 静态方法 | Static Method | /ˈstætɪk ˈmeθəd/ | 属于类本身的方法，只能访问静态成员 |
| 实例变量 | Instance Variable | /ˈɪnstəns ˈveəriəbl/ | 每个对象各自持有的成员变量 |

## 3.5 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1540) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
