# 第9章 第3节 List 与 Set

> 原文：[逻辑帧课程页](https://www.logamee.com/course-learning/77/1522)  
> 整理原则：按网页正文走；只改笔误和明显不通的句子；重点用加粗和颜色标出。

---

> **阅读指南**  
> 数组长度固定，业务数据却经常要增删，List 就是为此存在的。这一节先掌握 ArrayList 的增删改查和遍历，理解它和数组的区别；再看 Set——它的核心特征是无序、元素不可重复，所以去重、求并集交集这些操作，用 HashSet 比手工写循环省力得多。

本节配套视频请到 [原文](https://www.logamee.com/course-learning/77/1522) 观看。

---

## 3.1 认识 List

数组有一个明显的局限：长度一旦确定就无法更改。如果需要随时添加或删除元素，数组就显得力不从心。

<font color="blue">**List**</font> 是 Java 提供的另一种集合类型，可以视作「动态数组」。与普通数组最大的不同在于，List 可以随时添加和删除元素，不需要关心容量限制。

List 本身不能直接实例化，它是一个接口类型。实际开发中使用的是实现了 List 接口的具体类。最常见的实现类有：

**ArrayList**——基于数组实现的列表，使用频率最高

**LinkedList**——基于链表实现的列表

**CopyOnWriteArrayList**——线程安全的列表，用于并发场景

实际开发中最常用的是 `ArrayList`。下面以它为例讲解 List 的基本用法。

## 3.2 ArrayList 的基本操作

使用 `ArrayList` 需要在文件开头导入对应的包：

```java
import java.util.ArrayList;

class Main {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();

        // 添加元素
        list.add("Java");           // 在尾部添加
        list.add(0, "Python");      // 在指定位置插入

        // 获取元素
        String first = list.get(0).toString();

        // 修改元素
        list.set(1, "C++");         // 替换指定位置的元素

        // 删除元素
        list.remove(0);             // 按索引删除
        list.remove("C++");         // 按元素值删除

        // 查找元素
        int index = list.indexOf("Java");    // 返回索引,未找到返回-1
        boolean exists = list.contains("Python");  // 判断是否存在
    }
}
```

`add` 方法有两种调用形式：传入一个参数时，元素添加到列表末尾；传入两个参数时，第一个参数是插入位置，第二个参数是要插入的元素。

`remove` 方法同样有两种形式：按索引删除会移除该位置的元素；按元素值删除会移除第一个匹配的元素。

> **说明**  
> 上述代码在 IDE 中通常会提示警告。原因是直接使用 `ArrayList` 原生类型时，编译器无法确定元素的类型，获取元素后需要调用 `toString()` 才能当作字符串使用。更推荐的做法是使用「泛型」版本的 `ArrayList`，但目前的学习进度还没有讲到泛型，暂时可以忽略这些警告。

## 3.3 ArrayList 的遍历

和数组一样，List 也经常结合循环来遍历所有元素。Java 提供了多种方式：

**方式一：普通 for 循环**

```java
import java.util.ArrayList;

ArrayList list = new ArrayList();
list.add("Java");
list.add("Python");
list.add("C++");

for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

通过 `list.size()` 获取列表长度，用 `list.get(i)` 按索引逐个访问元素。这种方式适合需要用到索引的场景。

**方式二：增强 for 循环**

```java
for (Object lang : list) {
    System.out.println(lang.toString());
}
```

增强 for 循环不需要关心索引，它从第一个元素开始，依次取出每个元素，直到最后一个。

增强 for 循环适用于各种数组和集合类型：

```java
int[] arrays = {1, 2, 3, 4, 5};
for (int num : arrays) {
    System.out.println(num);
}
```

实践中使用增强 for 循环的频率远高于普通 for 循环。除非需要用到元素序号，否则应该优先选择增强 for 循环。

**方式三：迭代器（了解即可）**

```java
import java.util.Iterator;

Iterator it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next().toString());
}
```

迭代器是一种更底层的遍历方式，日常开发中用得较少，了解其存在即可。

还有一些更高级的遍历方式，需要后续学完 Lambda 表达式后才能理解：

```java
// forEach + Lambda
list.forEach(lang -> System.out.println(lang));

// 方法引用
list.forEach(System.out::println);

// 流式处理
list.stream()
    .filter(lang -> lang.toString().length() > 3)
    .forEach(System.out::println);
```

这些高级用法留到后续章节再详细讲解。

## 3.4 ArrayList 的特点

ArrayList 具有以下特征：

**有序性**——元素按插入顺序排列，先插入的排在前面

**可重复**——允许存储相同值的元素

**索引访问**——可以通过整数索引访问任意位置的元素

**允许 null**——可以包含 null 元素

```java
ArrayList list = new ArrayList();
list.add("Java");
list.add("Java");       // 允许重复
list.add(null);         // 允许null

System.out.println(list.get(0));  // Java
System.out.println(list.get(1));  // Java,与第一个元素相同
```

---

## 3.5 认识 Set

List 允许存储重复元素且保持插入顺序。有些场景下需要保证集合里的每个元素都不重复，这时就该使用 <font color="blue">**Set**</font>。

Set 的特征可以概括为两点：

<font color="orange">**无序**</font>——元素不保证按插入顺序排列（部分实现类除外）

<font color="orange">**唯一**</font>——不允许存储重复的元素

与 List 类似，Set 也是一个接口，不能直接实例化。常见的实现类有：

**HashSet**——基于哈希表实现，使用频率最高

**LinkedHashSet**——在 HashSet 基础上维护插入顺序

**TreeSet**——基于红黑树实现，元素按自然顺序排序

**EnumSet**——专门用于枚举类型的 Set

**CopyOnWriteArraySet**——线程安全的 Set

实际开发中最常用的是 `HashSet`。下面以它为例讲解 Set 的用法。

## 3.6 HashSet 的基本操作

使用 HashSet 需要导入对应的包：

```java
import java.util.HashSet;

class Main {
    public static void main(String[] args) {
        HashSet set = new HashSet();

        // 添加元素
        boolean added = set.add("Java");   // 添加成功,返回true
        added = set.add("Java");           // 已存在,添加失败,返回false

        // 删除元素
        boolean removed = set.remove("Python");  // 不存在,返回false
        removed = set.remove("Java");            // 存在,删除成功,返回true

        // 检查元素是否存在
        boolean contains = set.contains("C++");  // false

        // 集合信息
        int size = set.size();           // 元素数量
        boolean empty = set.isEmpty();   // 是否为空
    }
}
```

`add` 方法的返回值是布尔值，这一点和 List 不同。返回 `true` 表示元素是新加入的，返回 `false` 表示该元素已经存在，添加被拒绝。利用这个特性，可以通过返回值判断元素是否重复。

## 3.7 HashSet 的集合运算

两个 Set 之间可以进行数学意义上的集合运算，包括并集、交集和差集：

```java
import java.util.HashSet;

class Main {
    public static void main(String[] args) {
        HashSet set1 = new HashSet();
        HashSet set2 = new HashSet();

        set1.add("A");
        set1.add("B");
        set1.add("C");

        set2.add("B");
        set2.add("C");
        set2.add("D");

        // 并集:set1和set2的所有元素
        HashSet union = new HashSet(set1);
        union.addAll(set2);       // [A, B, C, D]

        // 交集:set1和set2共有的元素
        HashSet intersection = new HashSet(set1);
        intersection.retainAll(set2);  // [B, C]

        // 差集:在set1中但不在set2中的元素
        HashSet difference = new HashSet(set1);
        difference.removeAll(set2);    // [A]
    }
}
```

运算的关键点在于：需要先创建一个 Set 的副本，再调用 `addAll`、`retainAll` 或 `removeAll` 方法。这些方法会直接修改调用者本身，不会返回新的集合。

## 3.8 Set 的遍历

Set 的遍历方式与 List 类似：

**增强 for 循环**

```java
for (Object element : set) {
    System.out.println(element.toString());
}
```

**迭代器**

```java
import java.util.Iterator;

Iterator it = set.iterator();
while (it.hasNext()) {
    System.out.println(it.next().toString());
}
```

**Java 8 forEach**

```java
set.forEach(element -> System.out.println(element.toString()));
```

由于 Set 是无序的，遍历输出的元素顺序可能与插入顺序不一致。这是正常现象。

## 3.9 Set 的去重应用

利用 Set 元素不可重复的特性，可以轻松完成去重操作。这是 Set 最常见的实际用途：

```java
import java.util.HashSet;
import java.util.ArrayList;

class Main {
    public static void main(String[] args) {
        // 原始列表,包含重复元素
        ArrayList list = new ArrayList();
        list.add("Java");
        list.add("Python");
        list.add("Java");
        list.add("C++");
        list.add("Python");

        // 将List转为Set进行去重
        HashSet uniqueSet = new HashSet(list);

        // 输出结果
        for (Object item : uniqueSet) {
            System.out.println(item.toString());
        }
    }
}
```

将包含重复元素的 List 传入 HashSet 的构造方法，自动过滤掉重复项，得到不含重复元素的集合。

---

## 3.10 ■ 学点英语

| 中文 | English | 音标 | 说明 |
|------|---------|------|------|
| 列表 | List | /lɪst/ | 有序的、可重复的元素集合 |
| 动态数组 | Dynamic Array | /daɪˈnæmɪk əˈreɪ/ | 长度可变的数组实现 |
| 接口 | Interface | /ˈɪntərfeɪs/ | 定义方法签名但不提供实现的类型 |
| 实现类 | Implementation Class | /ˌɪmplɪmenˈteɪʃn klɑːs/ | 实现接口的具体类 |
| 泛型 | Generics | /dʒəˈnerɪks/ | 参数化类型的机制 |
| 迭代器 | Iterator | /ˈɪtəreɪtər/ | 用于遍历集合的对象 |
| 容量 | Capacity | /kəˈpæsɪti/ | 列表当前可容纳的最大元素数 |

## 3.11 ■ 思考帧

本节思考题为课程页交互题，请到 [原文](https://www.logamee.com/course-learning/77/1522) 完成本节练习。

---

## 相对网页原文改了什么

只动明确笔误 / 病句，不动教学内容。

| 原文 | 现写法 |
|------|--------|
| `[!TIP]` 阅读指南；`[!NOTE]` | 普通引用块 |
| `@logicframe-video{25}` | 改为到课程页观看 |
| 中文句子里的英文逗号、冒号 | 改为中文标点 |
| `@logicframe-question{…}` | 改为到课程页完成 |
