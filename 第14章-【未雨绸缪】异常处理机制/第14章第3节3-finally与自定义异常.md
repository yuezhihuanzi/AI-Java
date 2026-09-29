# 第14章 第3节 finally与自定义异常

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1551)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 这一节继续把异常处理补完。先看finally块——无论有没有异常它都会执行，是释放资源的传统位置，而Java 7的try-with-resources让资源管理更省心；然后讲主动抛出异常：用throw抛出业务错误、自定义异常类让错误信息更贴近业务语义，这些是大型项目里常见的做法。

## 3.1 finally块

`finally`块中的代码无论是否发生异常，都会被执行。它通常用于释放资源、关闭连接等清理工作。

```java
try {
    FileInputStream file = new FileInputStream("file.txt");
    // 读取文件...
} catch (IOException e) {
    e.printStackTrace();
} finally {
    // 无论是否发生异常，都要确保文件流被关闭
    if (file != null) {
        try {
            file.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    System.out.println("资源清理完毕。");
}
```

释放连接是一个无论程序是否出现错误都必须要做的事情。像这种无论是否异常都必须执行的操作，适合放在`finally`块里。

## 3.2 为什么必须释放资源

以数据库为例。数据库连接属于稀有的公共资源，多个程序均需要访问。如果程序占用了连接却不释放，其他程序就无法获取连接。连接很快会耗尽，新的请求会被数据库拒绝。

所以良性的获取/释放连接非常重要。

## 3.3 try-with-resources

传统的`finally`写法冗长且容易出错。Java 7引入了<font color="red">**try-with-resources**</font>语法，可以自动关闭实现了`AutoCloseable`接口的资源，无需显式编写`finally`块。

```java
// 传统写法（冗长）
FileInputStream file = null;
try {
    file = new FileInputStream("file.txt");
    // 使用文件...
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (file != null) {
        try {
            file.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// try-with-resources写法（简洁）
try (FileInputStream file = new FileInputStream("file.txt")) {
    // 使用文件...
} catch (IOException e) {
    e.printStackTrace();
}
// 离开try块后，file自动被关闭
```

多个资源用分号分隔：

```java
try (FileInputStream in = new FileInputStream("input.txt");
     FileOutputStream out = new FileOutputStream("output.txt")) {
    int data;
    while ((data = in.read()) != -1) {
        out.write(data);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**注意**：只有实现了`AutoCloseable`接口的资源类才会被自动调用`close()`方法。`FileInputStream`与`FileOutputStream`均实现了此接口。

## 3.4 传统方式与try-with-resources对比

```
    ┌────────────────────┬──────────────────────┐
    │    传统finally      │  try-with-resources   │
    ├────────────────────┼──────────────────────┤
    │ 需要手动关闭         │ 自动关闭              │
    │ 需要null检查         │ 不需要                │
    │ 关闭异常需嵌套try    │ 自动处理关闭异常       │
    │ 代码冗长             │ 代码简洁              │
    └────────────────────┴──────────────────────┘
```

## 3.5 throw关键字

`throw`用于在代码中主动地、显式地抛出一个异常对象。当程序执行到`throw`语句时，它会立即停止当前代码的执行，将异常抛出到上一级调用栈。

```java
throw new Exception("错误信息");
```

`throw`后面必须跟一个`Throwable`对象（即`java.lang.Throwable`类或其子类的实例）。可以在异常对象的构造函数中传入字符串，描述异常的详细信息。

```java
public class ThrowExample {
    public static void main(String[] args) {
        try {
            throw new ArithmeticException("这是一个除零错误的模拟");
        } catch (ArithmeticException e) {
            System.out.println("捕获到异常: " + e.getMessage());
        }
        System.out.println("程序继续执行...");
    }
}
```

## 3.6 为什么要使用throw

一个错误如果进行了处理，可以理解为被"消化"了。但并不是所有错误都可以在当前代码里处理。遇到当前代码不能处理的错误，可以选择用`throw`把错误抛出去，让其他代码去处理。

```java
try {
    int a = 1 / 0;
} catch (Exception e) {
    throw e; // 我处理不了，抛出去给别人处理
}
```

理解这一点很重要：即使你不写`throw`，Java也会在遇到错误时自动抛出异常。`throw`的价值在于<font color="red">**主动表达**</font>——程序员在代码中明确地告诉调用者"这里出了问题"。

## 3.7 自定义异常

Java内置了很多常见的异常，但这些异常都是通用的。业务场景中往往需要表达更具体的错误含义。

比如找出所有大于18岁的成年人。对于这段逻辑，所有小于18岁的都是非法数据。但Java不会内置一个"年龄小于18岁"的异常——这不是一个通用的错误。

这时候我们可以自己定义异常：

```java
// 必须继承Exception或RuntimeException
class AgeUnder18Exception extends RuntimeException {
    public AgeUnder18Exception(String message) {
        super(message);
    }
}

public class AgeValidator {
    public static void validateAge(int age) {
        if (age < 18) {
            throw new AgeUnder18Exception("年龄必须大于等于18岁，当前年龄：" + age);
        }
    }
    
    public static void main(String[] args) {
        validateAge(20); // 正常
        validateAge(15); // 抛出异常
    }
}
```

<font color="red">**命名规范**</font>：异常类名应以"Exception"结尾。同时需要提供一个构造方法，便于传入异常的文字说明。

## 3.8 throws关键字

`throws`用于方法声明中，指明本方法内部不处理某些检查型异常，而是将异常"抛出"给调用者来处理。这是一种"责任转移"机制。

```java
public class ThrowsDemo {
    // 方法声明：此方法可能抛出 IOException
    public void readFile() throws IOException {
        throw new FileNotFoundException("文件没找到！");
    }

    public void startToRead() {
        try {
            readFile(); // 调用者必须处理 readFile() 声明的异常
        } catch (IOException e) {
            System.err.println("处理IO异常: " + e.getMessage());
        }
    }
}
```

如果在调用时不去捕获异常，编译器会直接报错。因为`readFile()`在声明时强调了：调用我就必须处理`IOException`。

所有的CheckedException在抛出时，都需要在方法上用`throws`声明。

## 3.9 throw与throws的区别

这是一个关键区别，经常被初学者混淆：

| 特性 | throw | throws |
|------|-------|--------|
| 位置 | 方法体内部 | 方法声明处 |
| 含义 | 主动抛出一个异常对象 | 声明方法可能抛出的异常类型 |
| 性质 | 是一个动作 | 是一个承诺 |
| 后接内容 | 异常对象实例 | 异常类名 |

```java
public void readFile() throws IOException {  // throws：声明
    throw new FileNotFoundException("...");  // throw：动作
}
```

## 3.10 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 资源管理 | Resource Management | /ˈriːsɔːrs ˈmænɪdʒmənt/ | 获取和释放资源 |
| 清理 | Cleanup | /ˈkliːnʌp/ | 释放不再需要的资源 |
| 自动关闭 | Auto-Closeable | /ˈɔːtoʊ ˈkloʊzəbl/ | 可自动关闭的接口 |

## 3.11 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1551) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` / `[!NOTE]` / `[!WARNING]` | 普通引用块 |
| `@logicframe-question{…}` | 改为到课程页完成 |
| 中文句子里的英文逗号、问号、感叹号 | 改为中文标点 |
