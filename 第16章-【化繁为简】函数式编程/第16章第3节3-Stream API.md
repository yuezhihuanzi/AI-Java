# 第16章 第3节 Stream API

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1563)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 处理集合时，传统的for循环要关心"怎么遍历"，Stream让你只关心"要做什么"。这一节先理解Stream的流水线模型——数据从源头发出来，经过filter、map、sorted这些中间操作，最后由一个终端操作产出结果。中间操作不会立刻执行，只有终端操作出现时才真正开始计算，记住这一点，就不会写出白白遍历好几遍的代码。

## 3.1 什么是Stream API

Stream API是Java 8中处理集合数据的强大工具。它允许你以声明式、函数式的风格对数据集合进行复杂操作，如查找、过滤、映射、归约、排序等。

核心思想：你只需描述"要做什么"，而不需要关心"如何一步步去做"。

```
    传统方式（How-to）：写for循环 + if条件 + 中间变量
    Stream方式（What-to）：filter → map → sorted → collect
```

## 3.2 为什么需要Stream

假设有一个菜肴列表，想找出低热量的菜肴名，并按热量排序。

<font color="red">**传统命令式编程**</font>：

```java
List<Dish> menu = ...;
List<Dish> lowCaloricDishes = new ArrayList<>();

// 1. 过滤
for (Dish dish : menu) {
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}

// 2. 排序
Collections.sort(lowCaloricDishes, ...);

// 3. 获取菜名
List<String> names = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
    names.add(dish.getName());
}
```

这种方式冗长、嵌套，且需要多个中间集合。

<font color="red">**使用Stream API**</font>：

```java
List<String> lowCaloricDishesName = 
    menu.stream()                                  // 1. 获取流
        .filter(d -> d.getCalories() < 400)        // 2. 过滤
        .sorted(comparing(Dish::getCalories))      // 3. 排序
        .map(Dish::getName)                        // 4. 提取菜名
        .collect(Collectors.toList());             // 5. 收集
```

代码变成了一条清晰的声明式流水线，直接表达了业务逻辑。

## 3.3 Stream的特点

<font color="red">**不是数据结构**</font>：Stream本身不存储数据，而是对数据源进行计算操作。

<font color="red">**函数式编程风格**</font>：支持Lambda表达式和方法引用。

<font color="red">**延迟执行**</font>：许多操作（如过滤、映射）是延迟执行的，只有在需要结果时才会真正计算。

<font color="red">**可消费性**</font>：Stream只能计算一次结果。一旦执行终端操作，Stream就被消费掉了，无法再继续操作。

<font color="red">**隐式迭代**</font>：不需要显式使用循环，所有的遍历操作隐藏在Stream内部。

```java
// 传统方式：显式for循环
for (String name : names) {
    if (name.length() > 3) {
        result.add(name.toUpperCase());
    }
}

// Stream方式：毫无遍历的痕迹
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .toList();
```

## 3.4 创建Stream的方式

从集合创建（最常用）：
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

从数组创建：
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);
```

使用Stream.of()：
```java
Stream<String> stream = Stream.of("a", "b", "c");
```

使用Stream.generate()和Stream.iterate()：
```java
// 生成10个随机数的流
Stream<Double> randomStream = Stream.generate(Math::random).limit(10);

// 生成从0开始的等差数列
Stream<Integer> iterateStream = Stream.iterate(0, n -> n + 2).limit(5);
```

## 3.5 Stream操作分类

Stream操作分为两类：

- <font color="red">**中间操作**</font>：返回一个新的Stream，可以链式调用（延迟执行）
- <font color="red">**终端操作**</font>：返回最终结果，会触发实际计算

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

List<String> filtered = names.stream()
    .filter(name -> name.length() > 3)  // 中间操作
    .toList();                           // 终端操作，流被关闭
```

## 3.6 常用中间操作

### filter - 过滤

从集合中过滤掉不符合需求的元素：

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

List<String> filtered = names.stream()
    .filter(name -> name.length() > 3)
    .toList(); // ["Alice", "Charlie", "David"]
```

### map - 映射/转换

将集合中的元素依次进行形式上的转换。原来的集合长度不变，但元素都变成了Lambda表达式处理后的结果。

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 转换为大写
List<String> upperCaseNames = names.stream()
    .map(String::toUpperCase) 
    .toList(); // ["ALICE", "BOB", "CHARLIE"]

// 获取每个名字的长度
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .toList(); // [5, 3, 7]
```

### flatMap - 扁平化映射

将嵌套列表展开为一维列表：

```java
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d"),
    Arrays.asList("e", "f")
);

List<String> flatList = listOfLists.stream()
    .flatMap(List::stream)
    .toList(); // [a, b, c, d, e, f]
```

对于更高维的嵌套，可以链式调用多次`flatMap`：

```java
List<Integer> oneDList = threeDList.stream()
    .flatMap(List::stream)  // 展开第一维
    .flatMap(List::stream)  // 展开第二维
    .toList();
```

### sorted - 排序

```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob", "David");

// 自然排序
List<String> sortedNames = names.stream()
    .sorted()
    .toList(); // ["Alice", "Bob", "Charlie", "David"]

// 自定义排序（按长度）
List<String> lengthSorted = names.stream()
    .sorted((s1, s2) -> s1.length() - s2.length())
    .toList(); // ["Bob", "Alice", "David", "Charlie"]
```

### distinct - 去重

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> distinctNumbers = numbers.stream()
    .distinct()
    .toList(); // [1, 2, 3, 4, 5]
```

## 3.7 常用终端操作

### forEach - 遍历

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream().forEach(System.out::println);
```

执行后流被关闭。

### toList - 转换为列表

```java
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .toList();
```

### min和max - 最小值和最大值

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
```

### findFirst - 查找第一个元素

通常与filter结合使用：

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

Optional<String> firstLongName = names.stream()
    .filter(name -> name.length() > 3)
    .findFirst();
firstLongName.ifPresent(System.out::println); // Alice
```

### findAny - 查找任意元素

与findFirst类似，但在并行流中性能更好：

```java
Optional<String> anyLongName = names.stream()
    .filter(name -> name.length() > 3)
    .findAny();
```

## 3.8 综合示例

```java
// 用户列表
List<User> users = Arrays.asList(
    new User("Alice", 25, "London"),
    new User("Bob", 30, "New York"),
    new User("Charlie", 35, "London"),
    new User("David", 28, "Paris"),
    new User("Eve", 22, "London")
);

// 找出伦敦用户名，按年龄排序
List<String> londonUsers = users.stream()
    .filter(user -> "London".equals(user.getCity()))
    .sorted(Comparator.comparingInt(User::getAge))
    .map(User::getName)
    .toList(); // ["Eve", "Alice", "Charlie"]

// 计算伦敦用户的平均年龄
double averageAge = users.stream()
    .filter(user -> "London".equals(user.getCity()))
    .mapToInt(User::getAge)
    .average()
    .orElse(0.0); // 27.333...
```

## 3.9 peek调试

`peek()`允许在不改变流元素的情况下"窥视"流中的每个元素，对调试非常有用：

```java
List<String> result = Stream.of("apple", "banana", "cherry", "date")
    .peek(e -> System.out.println("原始元素: " + e)) 
    .filter(s -> s.length() > 4)
    .peek(e -> System.out.println("过滤后: " + e))    
    .map(String::toUpperCase)
    .peek(e -> System.out.println("转换后: " + e))    
    .toList();
```

`peek()`是中间操作，只有在终端操作执行时才会实际执行。

## 3.10 最佳实践

- Stream只能消费一次
- 尽量使用无状态的Lambda表达式
- 优先使用方法引用
- 数据量大且操作耗时时才考虑使用`parallelStream()`

## 3.11 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 流 | Stream | /striːm/ | 数据处理管道 |
| 声明式 | Declarative | /dɪˈklerətɪv/ | 描述做什么而非怎么做 |
| 中间操作 | Intermediate Operation | /ˌɪntərˈmiːdiət ˌɒpəˈreɪʃən/ | 返回新Stream的操作 |
| 终端操作 | Terminal Operation | /ˈtɜːrmɪnəl ˌɒpəˈreɪʃən/ | 触发计算并关闭流的操作 |
| 延迟执行 | Lazy Evaluation | /ˈleɪzi ɪˌvæljuˈeɪʃən/ | 需要结果时才计算 |

## 3.12 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1563) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
