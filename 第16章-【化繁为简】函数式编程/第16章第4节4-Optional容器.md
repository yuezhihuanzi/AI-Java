# 第16章 第4节 Optional容器

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1564)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 空值问题(NullPointerException)几乎是Java程序员每天都会撞上的墙。Optional是Java 8给出的答案：它用容器把"可能有值也可能没有"的状态显式表达出来，逼你在取值前先想清楚"没有值怎么办"。这一节会讲Optional的常用操作，以及它真正擅长和并不擅长的地方——别把它当成万能药。

## 4.1 为什么NullPointerException可怕

相信不少同学编写代码时，经常遇到空指针异常（NullPointerException，简称NPE）。

NPE最大的问题是：程序频繁地抛出NPE，但你却不知道为什么会产生NPE，直到通过大量调试才发现产生NPE的原因。

null值的可怕之处在于，它可以作为一个正常的结果在程序里到处传递。随着传递路径的增加，要找出错误的难度逐步增高。比如经过十几个方法的调用，终于某个方法抛出了NPE，但为了解决这个问题，可能需要回溯这十几个方法，最终才能发现null值产生的源头。

## 4.2 Optional是什么

`java.util.Optional<T>`是一个容器对象。你可以把它想象成一个盒子，里面装着"礼物"（数据）。但这个盒子不一定装有礼物，它也有可能是空的。

```
    Optional = 盒子
    数据 = 礼物（可能有，也可能没有）
```

Optional的主要目的是提供一种更优雅、更明确的方式来处理可能为null的情况，从而避免显式的null检查。

Optional强制你思考值可能不存在的情况，并让你显式地处理这种可能性，而不是隐式地假设值永远存在。

## 4.3 创建Optional

<font color="red">**确定里面一定有东西**</font>：
```java
Optional<String> box = Optional.of("一份真实的礼物");
// 如果传入null，会立即抛出NullPointerException
```

<font color="red">**确定里面是空的**</font>：
```java
Optional<String> box = Optional.empty();
```

<font color="red">**可能空，也可能有礼物**</font>（最常用、最安全）：
```java
Optional<String> box = Optional.ofNullable(someString);
// 如果someString为null，返回Optional.empty()
```

## 4.4 从Optional提取值

<font color="red">**不推荐：直接get()**</font>
```java
if (box.isPresent()) {
    String gift = box.get();
    System.out.println(gift);
}
```
这样和传统方法没有区别，毫无意义。

<font color="red">**推荐：ifPresent()**</font>
```java
Optional<String> opt = Optional.of("Alice");
opt.ifPresent(name -> System.out.println("Hello, " + name));
// 输出: Hello, Alice

Optional<String> emptyOpt = Optional.empty();
emptyOpt.ifPresent(name -> System.out.println("Hello, " + name));
// 什么也不做，不会NPE
```

<font color="red">**推荐：orElse()**</font>
```java
Optional<String> emptyOpt = Optional.empty();
String name = emptyOpt.orElse("Unknown");
System.out.println(name); // 输出: Unknown
```

无论Optional是否为空，都可以返回一个值，避免后续出现NPE。

<font color="red">**推荐：orElseGet()**</font>
```java
String name = emptyOpt.orElseGet(() -> getDefaultName());
```

与`orElse`的区别：`orElse`中的参数是立即求值的，即使Optional有值，默认值也会被创建。而`orElseGet`中的Supplier只在需要时（值为空时）才被调用。如果创建默认值的代价很高，应优先使用`orElseGet`。

<font color="red">**推荐：orElseThrow()**</font>
```java
Optional<User> user = userRepository.findById(id);
user.orElseThrow(
    () -> new EntityNotFoundException("User not found with id: " + id)
);
```

完美替代了"如果为null就抛异常"的模式。

## 4.5 Optional的本质

Optional不仅是对操作的简化手段，更是一种思考模式：你必须考虑空值的处理。

```java
// 传统方式：可能忘记判空
User user = userRepository.findById(id);
// if(user == null) { ... }  ← 经常忘记写

// Optional方式：被迫思考
Optional<User> user = userRepository.findById(id);
user.orElseThrow(() -> new RuntimeException("Not found"));
```

通过正确使用Optional，可以大大减少代码中的null检查样板代码，使逻辑更清晰，并有效降低NPE的发生风险。

## 4.6 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 容器 | Container | /kənˈteɪnər/ | 包装数据的对象 |
| 可选值 | Optional | /ˈɒpʃənəl/ | 可能为空的容器 |
| 样板代码 | Boilerplate Code | /ˈbɔɪlərpleɪt koʊd/ | 重复且冗余的代码 |

## 4.7 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1564) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-appreciate` | 赞赏入口略去（飞书无法渲染） |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
