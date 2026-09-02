# 第10章 第4节 getters 与 setters

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1529)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 把成员变量设为 private 只是封装的第一步，真正的问题是外部怎么安全地读写它。getters 和 setters 就是为此存在的：它们像门卫一样站在字段前面，让你能控制「读」和「写」的规则——比如写入前校验数据是否合法，或者只读不写。这一节就讲清楚这两个方法的写法，以及什么场景下值得为字段配它们。

---

## 4.1 为什么需要 getters 与 setters

封装性的另一个特点体现在「数据抽象」。

首先，类是对现实世界中某个事物的数据和行为的封装。数据和方法必然会分为两类：

- 可以公开被外部访问的
- 不允许被外部访问的（保密的）

学习了访问修饰符后，我们很容易就能控制哪些是私有的，哪些是公开的：

```java
class Person {
    // 公开的,任何类都可以访问和修改
    public String name;
    // 私有的,任何外部类都不能访问和修改
    private String id;
}
```

上述代码看似没有问题，但其实有很大的隐患：

`name` 被设置为 `public`，意味着可以被读取和修改。但任何名字都有一定的规范，比如中国人的名字不可能是英文字母，名字的长度也不能太长。其他类可以轻易地将 `name` 修改为 `"abcdefg"` 这种不合法的字符串。

身份证号理论上不能修改，所以可以设置为 `private`。但在极特殊情况下，比如身份证登记有误需要勘正时，那就需要能够更改身份证号。但设置为 `private` 后没有办法再修改了。

有时候，某个数据可以被读取，但我们并不希望原始数据直接暴露给外部。比如身份证号，可能只希望截取前 6 位提供给其他类访问，而不是完整的 18 位。

Java 除了需要灵活控制每个成员变量的可读、可写，还需要能够控制<font color="orange">**怎样修改**</font>以及<font color="orange">**怎样读取**</font>这个成员变量。

那 Java 怎么做到？

## 4.2 getters 与 setters

Java 提倡一个思想：<font color="red">**所有数据（成员变量）都应该是私有的**</font>。如果需要从外部修改数据或者读取数据，都应该通过「方法」来完成，不应该直接读取或者修改成员变量。

```java
// 错误写法(语法正确但不符合OO思想)
class Person {
    public String name;
    public String id;
}
```

成员变量是 `public` 的，其他类可以随意修改，这不符合面向对象的思想。应该修改成 `private`：

```java
class Person {
    private String name;
    private String id;
}
```

但如果是 `private` 的成员变量，就无法从类的外部修改数据了。Java 通过 <font color="blue">**getters**</font> 和 <font color="blue">**setters**</font> 来解决这个问题。看一段完整的示例：

```java
/**
 * 简单的Person类,展示基本属性和封装
 */
public class Person {
    // 成员变量 - 使用private保护数据
    private String name;    // 姓名
    private int age;        // 年龄
    private String gender;  // 存储英文性别("male" 或 "female")

    /**
     * 构造函数 - 创建Person对象
     */
    public Person(String name, int age, String gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    // ===== 公共访问方法(getters) =====
    // 用来读取成员变量的方法被称为getters
    // Java规定所有读取成员变量的方法都必须以get开头

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // 读取数据,并转换成中文
    public String getGender() {
        return switch (gender) {
            case "male" -> "男";
            case "female" -> "女";
            default -> "未知";
        };
    }

    // ===== 公共修改方法(setters) =====
    // 用来修改成员变量的方法被称为setters
    // Java规定所有修改成员变量的方法都必须以set开头

    public void setName(String name) {
        // 简单验证:姓名不能为空
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("姓名不能为空");
        }
        this.name = name;
    }

    public void setAge(int age) {
        // 简单验证:年龄在合理范围内
        if (age < 0 || age > 120) {
            throw new IllegalArgumentException("年龄必须在0-120之间");
        }
        this.age = age;
    }

    public void setGender(String gender) {
        // 简单验证:性别只能是"男"或"女"
        if (!"男".equals(gender) && !"女".equals(gender)) {
            throw new IllegalArgumentException("性别必须是'男'或'女'");
        }
        this.gender = gender;
    }

    // 普通方法
    public void celebrateBirthday() {
        this.age++; // 年龄增加1岁
        System.out.println(this.name + " 过生日啦!现在" + age + "岁了!");
    }
}
```

上述代码是一个比较完整的 Java 类，包含以下几个部分：

- **成员变量**
- **构造函数**
- **getters**（用以读取成员变量）
- **setters**（用以修改成员变量）
- **普通方法**

## 4.3 getters 和 setters 的意义

- **getters**：特殊的方法，方法名以 `get` 开头，专用来读取成员变量
- **setters**：特殊的方法，方法名以 `set` 开头，专用来修改成员变量

对于某个成员变量，如果配备有 getter 或 setter，我们称为<font color="blue">**属性**</font>。成员变量通常都是私有的（`private`），而属性通常意味着可以读或者可以修改。

无论是 getters 还是 setters，都是让我们在读或者写成员变量前有一次处理机会：

```java
public void setAge(int age) {
    // 简单验证:年龄在合理范围内
    if (age < 0 || age > 120) {
        throw new IllegalArgumentException("年龄必须在0-120之间");
    }
    this.age = age;
}
```

上述 setter 最终目的是修改 `age`，但年龄必须是合法数字，不能是负数、也不能太大。所以首先用 `if` 检查输入的参数，如果不在正常范围内就抛出异常，从而停止修改成员变量。

对于 getter 也是一样：

```java
public String getGender() {
    return switch (gender) {
        case "male" -> "男";
        case "female" -> "女";
        default -> "未知";
    };
}
```

在返回数据之前，也有一次「机会」来处理数据。上述代码将英文的 `male` 和 `female` 转换成中文再输出。

相对于成员变量，getters 和 setters 都是方法，方法具有更灵活的机制——可以防止非法数据被写入、可以过滤掉机密数据、可以转换数据的输出格式。它们像成员变量的一道屏障，比只能简单读写成员变量要强大很多。

## 4.4 数据抽象

所谓<font color="red">**数据抽象**</font>，就是将原始数据存储在成员变量中，但是对外不直接暴露原始数据，而是通过 getters 和 setters 提供数据，这样更安全。

记住：永远将成员变量设置为 `private`，这是一个很好的做法。

---

## 4.5 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 访问器 | Getter | /ˈɡetə(r)/ | 以 get 开头的方法，用于读取私有成员变量 |
| 修改器 | Setter | /ˈsetə(r)/ | 以 set 开头的方法，用于修改私有成员变量 |
| 属性 | Property | /ˈprɒpəti/ | 配有 getter 或 setter 的成员变量 |
| 数据抽象 | Data Abstraction | /ˈdeɪtə æbˈstrækʃn/ | 通过公共接口提供数据，隐藏内部存储细节 |
| 验证 | Validation | /ˌvælɪˈdeɪʃn/ | 检查输入数据是否合法的过程 |
| 异常 | Exception | /ɪkˈsepʃn/ | 程序运行时的错误情况，如 IllegalArgumentException |

## 4.6 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1529) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` 阅读指南 | 普通引用块 |
| 阅读指南中英文逗号 | 改为中文逗号 |
| `@logicframe-appreciate` | 赞赏入口略去（飞书无法渲染） |
| `@logicframe-question{…}` | 改为到课程页完成 |
