# 第14章 第2节 异常分类与try-catch

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1550)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 程序出错时，Java会用异常把错误信息带出来。这一节先看清异常的继承体系：Error是系统级故障，Checked Exception是需要我们显式处理的错误，RuntimeException则是程序逻辑bug，三者性质不同，处理方式也不同。在此基础上再学try-catch——把可能出错的代码包起来，异常发生时程序才能继续走下去，而不是直接崩溃。

## 2.1 Throwable根类

Java中所有异常和错误的根类是`java.lang.Throwable`。它有两个直接子类，构成了整个异常体系的基石：

```
              Throwable
              /       \
           Error     Exception
                       /    \
          Checked Exception   RuntimeException (Unchecked)
```

## 2.2 Error（错误）

<font color="red">**Error**</font>表示程序无法处理的严重问题，通常与JVM（虚拟机）相关。应用程序不应该试图捕获和处理这类问题，因为即使捕获了也无法处理。

常见Error举例：

- <font color="red">**OutOfMemoryError**</font>：内存耗尽。计算机的内存是有限的，程序可能耗尽内存。
- <font color="red">**StackOverflowError**</font>：栈溢出。通常由无限递归或方法调用层次过深引起。
- <font color="red">**NoClassDefFoundError**</font>：无法找到某个类的定义。可能在运行时缺少.class文件或jar包。

```
    Error不需要处理 ≠ 错误不需要修复
    
    运行时阶段：无法处理（捕获了也没用）
    源码阶段：  必须修复（修改代码解决问题）
```

这些错误在程序运行时出现时，你无能为力。唯一的选择是停止程序，回到源码中找到问题并修复它。所以运行时捕获Error没有意义。

## 2.3 Exception（异常）

<font color="red">**Exception**</font>表示程序本身可以处理的非严重问题。这是我们重点关注和处理的类别。Exception又分为两大类。

### 检查型异常（Checked Exception）

CheckedException是Exception类本身及其非RuntimeException的子类。它们通常代表可预测的、程序应该有能力从中恢复的问题。

常见Checked Exception：
- <font color="red">**IOException**</font>及其子类：输入输出操作失败
- <font color="red">**SQLException**</font>：数据库访问出错
- <font color="red">**ParseException**</font>：解析字符串格式错误

```java
// 检查型异常必须在方法上声明throws，或捕获处理
public void readFile() throws IOException {
    // 编译器强制要求处理
}
```

检查型异常大部分由外部系统引起，属于程序员不可控的情况。比如想读取的文件可能被其他人删除了。

### 非检查型异常（RuntimeException）

RuntimeException及其子类统称为<font color="red">**非检查型异常**</font>（Unchecked Exception）。Java编译器不会检查它们，方法可以抛出RuntimeException而无需在方法签名上声明。

这类异常通常代表程序的逻辑错误或API使用不当，是程序员应该在编码阶段避免的bug。

常见RuntimeException：
- <font color="red">**NullPointerException**</font>：尝试访问null对象的成员
- <font color="red">**ArrayIndexOutOfBoundsException**</font>：数组下标越界
- <font color="red">**IllegalArgumentException**</font>：传递给方法的参数不合法
- <font color="red">**NumberFormatException**</font>：尝试将非数字字符串转换为数字
- <font color="red">**ArithmeticException**</font>：算术错误，如整数除以零
- <font color="red">**ClassCastException**</font>：错误的类型转换

## 2.4 三类异常的对比

| 特性 | Error | Checked Exception | RuntimeException |
|------|-------|-------------------|------------------|
| 编译期检查 | 否 | 是 | 否 |
| 应该捕获 | 否 | 是 | 可选 |
| 代表含义 | 系统级故障 | 外部可恢复错误 | 程序逻辑bug |
| 示例 | OutOfMemoryError | IOException | NullPointerException |

## 2.5 try-catch的作用

程序在正常运行时，按照编写的指令流逐条执行。但如果出现错误又没有处理，程序就会崩溃退出。

```java
int a = 1 / 0;       // 除零错误
System.out.println("Hello"); // 这行不会执行
```

上述代码在遇到除零错误后，程序终止，后续的打印语句不会执行。

<font color="red">**try-catch**</font>的作用是将可能出现错误的代码包裹起来，当错误发生时，程序不会崩溃，转而执行catch块中的处理逻辑，然后继续运行后续代码。

```java
try {
    int a = 1 / 0;
} catch (Exception e) {
    System.out.println("1不能除以0");
}
System.out.println("Hello"); // ✓ 这行会正常执行
```

这就是异常处理最大的意义：不让错误的出现导致程序崩溃，它会尝试进行异常处理，并继续运行后续代码。

## 2.6 try-catch语法

```java
try {
    // 可能会抛出异常的代码
} catch (异常类型 变量名) {
    // 处理特定异常的代码
}
```

如果`try`块中的代码出现异常，程序会立即跳出`try`块，进入匹配的`catch`块执行处理逻辑。

## 2.7 多重catch块

一个`try`块后可以跟多个`catch`块，用于处理不同类型的异常：

```java
try {
    int[] arr = new int[5];
    arr[10] = 30; // 可能抛出 ArrayIndexOutOfBoundsException
    Class.forName("com.unknown.Class"); // 可能抛出 ClassNotFoundException
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("数组越界了！");
} catch (ClassNotFoundException e) {
    System.out.println("类没找到！");
} catch (Exception e) {
    System.out.println("发生了其他未知错误！");
}
```

这里要注意一个规则：如果多个catch中的异常类型有父子类关系，必须将<font color="red">**父类放在子类的后面**</font>。这叫做"兜底"。

`Exception`是所有异常的父类，所以必须放在最后。如果把父类放在前面，子类的catch永远不会被执行（编译器会直接报错）。

## 2.8 多重捕获

如果多种异常需要相同的处理逻辑，可以使用`|`合并：

```java
try {
    // ...
} catch (IOException | SQLException e) {
    e.printStackTrace(); // 同时处理 IO 和 SQL 异常
}
```

这避免了重复编写相同的异常处理代码。

## 2.9 try块内的执行流

在try的内部，如果出现了异常，程序不会继续执行try内部剩余的代码：

```java
try {
    int a = 1 / 0;              // #1 出现异常
    System.out.println("我会执行吗"); // #2 不会执行
} catch (Exception e) {
    System.out.println("出错了");       // #3 执行
}
System.out.println("Hello");           // #4 执行
```

当遇到异常时（#1），直接跳出到catch块执行（#3），随后再继续主流程（#4）。

## 2.10 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 错误 | Error | /ˈerər/ | 系统级严重问题 |
| 检查型异常 | Checked Exception | /tʃekt ɪkˈsepʃən/ | 编译期强制处理的异常 |
| 运行时异常 | RuntimeException | /rʌn taɪm ɪkˈsepʃən/ | 非检查型异常 |
| 堆栈跟踪 | Stack Trace | /stæk treɪs/ | 异常调用链信息 |

## 2.11 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1550) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
