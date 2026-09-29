# 第16章 第2节 Lambda与方法引用

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1562)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> Lambda表达式让"把行为当作参数传递"成为可能，写起来比匿名内部类简洁得多。这一节先从函数式编程的角度理解Lambda是什么，再掌握它的语法和函数式接口——Lambda不能单独存在，必须赋值给一个恰好有一个抽象方法的接口，Java还内置了四种核心函数式接口，覆盖最常见的场景。最后看方法引用：当Lambda体只是调用一个现成方法时，可以用更简短的写法替代。

## 2.1 行为作为参数

我们已经习惯将数据作为参数传入方法：

```java
class Calculator {
    public static int add(int a, int b) {
        return a + b;
    }
}
Calculator.add(3, 2); // 把数据3和2作为参数传入
```

但"行为"（即方法）能不能当做参数传入其他方法呢？

```python
# Python示例：函数作为参数传递
def say_hi():
    print("嗨！")

def call_function(func):
    func()  # 调用传入的函数

call_function(say_hi)  # 输出: 嗨！
```

这种把"方法"当做参数传递的做法是典型的函数式编程。在Java 8之前，Java很难做到这一点。

## 2.2 Lambda表达式

<font color="red">**Lambda表达式**</font>是一个可传递的匿名函数。从三个关键词理解：

- <font color="red">**匿名**</font>：它没有明确的名称，像一个没有名字的函数
- <font color="red">**函数**</font>：有参数列表、函数体、返回类型，但不属于某个特定的类
- <font color="red">**可传递**</font>：可以作为参数传递给方法或存储在变量中

Lambda表达式的核心价值在于：你可以将一段代码（行为）像数据一样进行传递。

## 2.3 Lambda语法

```
(parameters) -> expression
(parameters) -> { statements; }
```

- <font color="red">**参数列表**</font>：与传统方法参数类似，位于括号内
- <font color="red">**箭头符号**</font>：`->`将参数列表与Lambda主体分隔开

几个示例：

```java
() -> 42                              // 无参数，返回42
() -> { System.out.println("Hi"); }   // 无参数，多条语句需要{}
a -> a * a                            // 单参数，括号可省略
(a, b) -> a + b                       // 多参数，括号不可省略
(int a, int b) -> a > b               // 可显式指定类型
```

当Lambda主体只有一条表达式时，不需要`return`关键字，表达式的值会自动返回。当主体有多条语句时，必须用`{}`包裹，并使用`return`返回值。

## 2.4 Lambda的应用场景

### 简化线程创建

```java
// 匿名内部类写法（冗长）
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World!");
    }
};
new Thread(runnable).start();

// Lambda写法（简洁）
new Thread(() -> System.out.println("Hello from Lambda!")).start();
```

### 简化排序

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");

// Lambda方式指定排序规则
names.sort((a, b) -> a.compareTo(b));
```

`sort`方法让程序员自己定义排序规则，然后将规则传入。这个规则可以用Lambda表达式来制定。

### 遍历集合

```java
List<String> list = Arrays.asList("A", "B", "C");

// Lambda方式遍历
list.forEach(s -> System.out.println(s));
```

`forEach`会遍历List集合，每次循环时调用后面的Lambda表达式，并将当前元素作为参数传给Lambda表达式。

## 2.5 什么是函数式接口

Lambda表达式不能单独存在，它必须被赋值给一个<font color="red">**函数式接口**</font>类型的变量。

<font color="red">**函数式接口**</font>是有且仅有一个抽象方法的接口。关键点：

1. <font color="red">**单个抽象方法**</font>：最核心的规则。可以包含默认方法（default）和静态方法，它们不计入抽象方法数量
2. <font color="red">**@FunctionalInterface注解**</font>：不是必须的，但强烈推荐使用。它告诉编译器检查接口是否真的只有一个抽象方法

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t); // 单个抽象方法
}
```

## 2.6 Lambda赋值给函数式接口

```java
// 将Lambda表达式赋值给Consumer<T>接口
Consumer<String> printer = s -> System.out.println(s);
```

正确的理解是：Lambda表达式<font color="red">**实现**</font>了这个函数式接口。Lambda表达式是一种接口实现的简化写法。

## 2.7 Java内置的四种核心函数式接口

位于`java.util.function`包中。

### Consumer\<T\>

接收一个参数，无返回值。"Consumer"译为"消费者"，恰如生活中消费者的行为：接收参数，但不返回任何值。

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

// 示例
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello Consumer"); // 输出: Hello Consumer
```

### Function\<T, R\>

接收参数，返回结果。"Function"译为"功能、函数"，接收一个T类型参数，返回一个R类型结果。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// 示例：输入字符串，求长度
Function<String, Integer> stringLength = s -> s.length();
Integer len = stringLength.apply("Hello"); // 5
```

### Supplier\<T\>

不接受参数，返回一个结果。"Supplier"译为"供应者"，不需要接收参数，只提供"结果"。

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// 示例：随机数供应者
Supplier<Double> randomSupplier = () -> Math.random();
Double d = randomSupplier.get();
```

### Predicate\<T\>

接受一个输入参数，返回布尔值。"Predicate"译为"断言"，主要用于判断条件是否为真。

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

// 示例：偶数判断
Predicate<Integer> isEven = n -> n % 2 == 0;
boolean b = isEven.test(10); // true
```

`Predicate`还支持更复杂的组合判断：

```java
Predicate<Integer> isEven = num -> num % 2 == 0;
Predicate<Integer> isPositive = num -> num > 0;

// 组合条件：正偶数
Predicate<Integer> isPositiveEven = isEven.and(isPositive);
isPositiveEven.test(12);  // true
isPositiveEven.test(-4);  // false
```

## 2.8 函数式接口变体

Java还提供了很多变体，如：
- <font color="red">**UnaryOperator\<T\>**</font>：是`Function<T, T>`的特例，接受一个参数T，返回相同类型T的结果

大多数情况，不需要自己定义新的函数式接口，应该优先查找Java是否内置了符合需求的接口。

## 2.9 为什么需要方法引用

方法引用提供了一种更简洁的方式来调用已有方法。当Lambda体仅仅是调用一个已经存在的方法时，可以直接通过方法名来引用，而不是用Lambda重写整个调用过程。

对比一个经典例子：对字符串列表排序，忽略大小写。

```java
// Lambda表达式
list.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// 方法引用
list.sort(String::compareToIgnoreCase);
```

方法引用`String::compareToIgnoreCase`更加直接，清晰地表达了"我想使用String类的compareToIgnoreCase方法来定义排序规则"。

语法：`类名：:方法名` 或 `对象：:方法名`

**注意**：方法引用只能在Lambda表达式的方法体只调用一个方法时使用。

## 2.10 四种方法引用

假设有一个名字列表：
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
```

### 静态方法引用

语法：`ClassName::staticMethodName`

```java
// Lambda写法
names.forEach(name -> System.out.println(name));

// 方法引用写法
names.forEach(System.out::println);
```

### 引用外部对象的实例方法

语法：`objectInstance::instanceMethodName`

```java
// Lambda写法
String prefix = "Hello, ";
names.forEach(name -> prefix.concat(name));

// 方法引用写法
String prefix = "Hello, ";
names.forEach(prefix::concat);
```

### 引用特定类型的任意对象的实例方法

语法：`ClassName::instanceMethodName`

这是最容易混淆但非常有用的一种。当Lambda表达式的第一个参数是方法的调用者，其余参数是方法的参数时，可以使用这种引用。

```java
String::toUpperCase 
// 等价于 (s) -> s.toUpperCase()

String::compareToIgnoreCase 
// 等价于 (s1, s2) -> s1.compareToIgnoreCase(s2)

// 使用示例
names.forEach(String::length);      // 引用String类的length方法
names.forEach(String::toUpperCase); // 引用String类的toUpperCase方法
```

### 引用构造方法

语法：`ClassName::new`

```java
// Lambda写法
List<String> numbers = Arrays.asList("1", "2", "3");
numbers.forEach(numStr -> new String(numStr));

// 方法引用写法
numbers.forEach(String::new);
```

## 2.11 总结对比

| 场景 | Lambda表达式 | 方法引用 |
|------|-------------|---------|
| 调用静态方法 | `s -> Integer.parseInt(s)` | `Integer::parseInt` |
| 调用外部对象方法 | `s -> myChecker.isValid(s)` | `myChecker::isValid` |
| 调用任意类型实例方法 | `s -> s.toUpperCase()` | `String::toUpperCase` |
| 调用构造方法 | `s -> new User(s)` | `User::new` |

把方法引用想成"快捷键"：当你Lambda表达式的内容仅仅是调用一个已存在的方法时，就毫不犹豫地用它。

## 2.12 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| Lambda表达式 | Lambda Expression | /ˈlæmdə ɪkˈspreʃən/ | 匿名函数 |
| 匿名函数 | Anonymous Function | /əˈnɒnɪməs ˈfʌŋkʃən/ | 没有名字的函数 |
| 函数式编程 | Functional Programming | /ˈfʌŋkʃənəl ˈproʊɡræmɪŋ/ | 以函数为核心的编程范式 |
| 箭头符号 | Arrow Token | /ˈæroʊ ˈtoʊkən/ | -> 分隔符 |

## 2.13 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1562) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
