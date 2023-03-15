---
title: 【Java】Best coding practices every java
subtitle: JAVA开发人员规范指导
author: lazytime
url_suffix: bestjavacoding
tags:
  - Java
categories:
  - Java
keywords: java规范案例
description: java规范案例
copyright: true
date: 2023-03-15 14:55:58
---

# 原文

[Best coding practices every java developer should follow](https://medium.com/@abhisheksinghjava/best-coding-practices-every-java-developer-should-follow-7b77527a7048#id\_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjI1NWNjYTZlYzI4MTA2MDJkODBiZWM4OWU0NTZjNDQ5NWQ3NDE4YmIiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2NzgzNzA3NDUsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjEwMjUzNjAwNjY1MTA5NDQxNzM4MyIsImVtYWlsIjoienhkc2I2NjZAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImF6cCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsIm5hbWUiOiJBbGV4IHhhbmRlciIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS9BR05teXhZZGFjVHk3NkpRNTBMT2VDRWNZRUh6dzlsdURpRnZfVjZTMWVoaj1zOTYtYyIsImdpdmVuX25hbWUiOiJBbGV4IiwiZmFtaWx5X25hbWUiOiJ4YW5kZXIiLCJpYXQiOjE2NzgzNzEwNDUsImV4cCI6MTY3ODM3NDY0NSwianRpIjoiM2I5N2I0ODQ3YzdmOTQxNDg2NGYyN2EzNThhY2Q1MDI3YjYzYTQxMiJ9.I33Fy\_b0eQACI4KWMjz0RsWvaFh1X3-I6258uov2O6B1x4vqn3PWLA-NdiyT78toJUNIYGJ2QLRzq0wz336E4t\_nhrffy7qp7a4KUOKqs8IuPLDwgWKtqApNSkpy9TWB61rhlZ8Cnr7sgbZoG2xuxq0DjJUmVaYZ--GUUFftVIzTv9DyLEVNG71xFhX9S73qXSaJtybbp1JpjmM526DAf-61KMrsoCB\_YZ9DN2D4W4C0BdAMZQ2cWnZvvG7yHMAH0R51l7xn-nUMlMnKZN12wb-VZCSbw86d4-kKgrr9b18dP7R3JDlNhUo3CEL5lwZpPWdMj3gHzRw0ZBr94HWLSg)

## 目录

-   [引言](#引言)
-   [4. 尽可能让变量私有化](#4-尽可能让变量私有化)
-   [6. 警惕冗余初始化](#6-警惕冗余初始化)
-   [5. Stringbuilder替换字符串拼接【争议】](#5-Stringbuilder替换字符串拼接争议)
-   [15. dry和kiss](#15-dry和kiss)
-   [14. Solid](#14-Solid)
-   [7. 尽可能使用增强for循环或者foreach](#7-尽可能使用增强for循环或者foreach)
-   [13. 日志打印规则](#13-日志打印规则)
-   [12. Hardcoding硬编码](#12-Hardcoding硬编码)
-   [11. 注释](#11-注释)
-   [10. 避免创建不必要对象](#10-避免创建不必要对象)
-   [9. 返回空集合而不是null](#9-返回空集合而不是null)
-   [8. 精度选择](#8-精度选择)
-   [1. 项目结构](#1-项目结构)
-   [2. 遵循命名规范](#2-遵循命名规范)
-   [3. 不要吞异常](#3-不要吞异常)
-   [16. 使用枚举替代静态常量【建议】](#16-使用枚举替代静态常量建议)
-   [17. 按作用域划分成员变量](#17-按作用域划分成员变量)
-   [18. 在数字字段中使用下划线【建议】](#18-在数字字段中使用下划线建议)
-   [总结](#总结)
-   [参考](#参考)

# 引言

把标题翻译成中文在国内也是一个老生常谈的问题：编程习惯和编码规范。

这篇文章大部分观点和国内的规范习惯类似，令我好奇的是外国人是如何理解这些内容的？

注意本文的Tips排序是打乱的，个人把感兴趣放到了前面来了。这篇文章的评论区非常精彩，这里一并整合了评论区的读者观点。

不管是学习还是了解老外的思考方式，或是在评论区当中看读者的讨论和纠正错误，阅读本文都是值得推荐的。

抱着半信半疑的态度学习这些内容，你会得到比单纯的了解和接受建议更多的收获。

<!-- more -->

# 4. 尽可能让变量私有化

如果变量不需要意对外访问，那就建议使用私有描述对于参数进行描述。

评论区的读者对于这一个小节做了补充：

-   如果是dto并且是final的，拥有公用数据的时候**可以不私有化**.。
-   比大量的setter更好建议是更合理的利用设计模式构建对象（比如建造者），或者利用委托第三方对象生产合适的对象（静态工厂替代构造器）。
-   尽可能让对象不可变。



# 6. 警惕冗余初始化

> Therefore, a java best practice is to be aware of the default initialization values of member variables and avoid initializing the variables explicitly.

Java 最佳实践是了解成员变量的默认初始化值并避免显式初始化变量，Java语言很多变量存在默认值，在自己编写初始化的时候不建议使用Java的默认值。

另一方面，有时候可以利用Java的初始化做数据库字段的优化，比如开关状态建议把0设置为开放1设置为关闭。

下面的代码的的对象初始化代码是毫无意义的：

```java
public class Person {
    private String name;
    private int age;
    private boolean;
 
    public Person() {
        String name = null;
        int age = 0;
        boolean isGenius = false;
    }
}
```

> Although it is very common practice, it is not encouraged to initialize member variables with the values: like 0, false and null. These values are already the default initialization values of member variables in Java. Therefore, a java best practice is to be aware of the default initialization values of member variables and avoid initializing the variables explicitly.

尽管这是非常常见的做法，但不鼓励使用以下值初始化成员变量：如 0、false 和 null。这些值已经是 Java 中成员变量的默认初始化值。因此，Java 最佳实践是了解成员变量的缺省初始化值，并避免显式初始化变量。

See more here: [Java default Initialization of Instance Variables and Initialization Blocks](https://www.codejava.net/java-core/the-java-language/java-default-initialization-of-instance-variables-and-initialization-blocks "Java default Initialization of Instance Variables and Initialization Blocks")。



# 5. Stringbuilder 替换字符串拼接【争议】

实际上多数情况下“大可不必”，只有for循环的情况才考虑是否使用Stringbuilder替换。日常情况下字符拼接操作是完全没有问问题的，javac编译之后会把字符串自动用StringBuilder替换，真正应该手动创建该对象的场景是在**for循环当中的大量的字符串拼接**，内部会每次迭代新建Stringbuilder。

随着JDK版本的升级，到**JDK9**版本for循环新建StringBuilder的情况已经被改善了（但个人未从OpenJDK中找到对应源码验证）。

JDK13 从Python那边把多行文本的语法搬过来了，定义多行文本可以类似下面这样：

```java
String htmlContent = “““<html>
                           <head>
                           </head>
                           <body>
                              <p>Hello world!</p>
                           </body>
                         </html>""";
```

最后需要注意的是使用**Stringbuilder**，仅仅应该用在多线程循环操作字符串当中，如果在同步方法里面效率会非常低并且很慢。

介绍

> As a general rule, optimize at an architectural level, and write code for readability. Later on it’s easier to optimize a neat function, than to look for bugs in an optimized one.

上面意思大致是说通用规则是优先写出具备高可读性的代码，然后再去进行优化这些简单的可优化点，这比在优化代码找错误要简单很多。



# 15. dry和kiss

> DRY stands for “Don’s Repeat Yourself

如果代码可以公用就尽量公用，不要复制粘贴代码。

> kiss：KISS stands for “Keep It Simple, Stupid”.

KISS就是保持简单，愚蠢。这个愚蠢指的是方法最好只知道干一件事情。这两点和Unix最初的设计的理念是一样的，简单好用即是美。



# 14. Solid 

**Single Responsibility Principle** 单一职责：每个类和接口有明确目标。

**Open-Closed Principle** 开闭原则：开放扩展，封闭修改。

**Liskov Substitution Principle** 里式替代：尽可能让代码多态。

**Interface Segregation Principle** 接口隔离：实现接口替代类继承。

**Dependency Inversion Principle** 依赖倒转原则：依赖抽象而不是依赖实现。



# 7. 尽可能使用增强for循环或者foreach

建议使用增强for循环（JDK5）和JDK8的foreach，当然最好的建议是活学活用stream的API。为什么会有这样的建议？好像for-index也不是啥坏写法。

> It’s because the index variable is error-prone, as we may alter it incidentally in the loop’s body, or we may starts the index from 1 instead of 0.

这是因为索引变量容易出错，因为我们可能会在循环的主体中偶然改变它，或者我们可能从1而不是0开始索引。下面是Stream API带来便利的简单例子：

```java
public Integer findSmallesPositiveNumber(List<Integer> numbers) {
   Integer smallestPositiveNumber= null;
   for (Integer number: numbers) {
      if (number > 0) {
      if (smallestPositiveNumber == null
            || number < smallestPositiveNumber) {
         smallestPositiveNumber = number;
      }
   }
   return smallestPositiveNumber;
}
```

使用Stream APi精简之后：

```java
public Optional<Integer> findSmallesPositiveNumber(
      List<Integer> numbers) {
   return numbers.stream()
      .filter(number -> number > 0)
      .min(Integer::compare);
}
```



# 13. 日志打印规则

作者的下面几条规则**有待商榷**，我个人建议是**避免**下面的做法：

> Developer should add logger on method entry and exit.

在进出重要方法打印入参和出参。

> 谨慎使用，建议涉及金额的操作情况打印。

> Developer should add logger in “if” and “else” case to track which condition is true in case of any error.

在if和else的条件判断处打印参数。

> Developer should also log exception to track the issue

对于异常信息必须打印。



更为合理的做法是像下面这样：

> Definitely don’t log every if-else statement!

不要在if/else分支中打印，更为合理的建议是记录响应以及错误。

> **Don’t log every method entry and exit!**

不要在每个方法的入参和出参打印。当然这并不是铁律，比如三方接口调用就必须要在“入口”和“出口”中打印以便快速定位问题~~和甩锅~~。

> \*\*Log exceptions that are unexpected. \*\*

记录“意外”的异常。比如多线程有可能的interrupt处理，文件读写有可能的IO异常。

从褒义和贬义两个层面看，日志都是非常具备“价值”的，好的日志可以帮你快速定位问题，不好的日志就像是垃圾一样迅速占用服务器的磁盘。只有非常核心的日志才需要跟踪每一步的行为，绝大多数情况下日志只需要在影响业务的位置进行打印。在打印日志段时候多思考一下**日志等级**的选择，这意味着你生产能否快速定位问题。



# 12. Hardcoding硬编码

硬编码回会导致程序难以理解。使用硬编码会增加理解难度，通常使用枚举替代是不错建议。根据dry的原则，在定义硬编码的时候，如果魔法值在JDK中存在类似定义或者存在现实意义，应该果断通过下面的方式进行纠正，比如下面的例子：

```java
private int storeClosureDay = 7;
```

应该被替换为下面这样：

```java
private static final int SUNDAY = 7;  
[…]  
private int storeClosureDay = SUNDAY;
```

**不要**这样使用，更为合适的处理方式是使用[DayOfWeek](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html "DayOfWeek") 的API：

```java
private DayOfWeek storeClosureDay = DayOfWeek.SUNDAY;
```

避免硬编码是非常好的编程习惯，更好的习惯是使用易懂的硬编码。



# 11. 注释

关键：**注释只有在可读性较差并且需要注释描述关键意图的时候才使用**。

> As your code will be read by various people with varying knowledge of Java, Proper comments should be used to give overviews of your code and provide additional information that cannot be perceived from the code itself. Comments are supposed to describe the working of your code to be read by Quality assurance engineer, reviewer, or maintenance staff .

由于代码将被具有不同 Java 知识的各种人阅读，因此应该使用适当的注释来概述代码，并提供无法从代码本身感知的其他信息。注释应该描述代码的工作方式，以供质量保证工程师、审阅者或维护人员阅读。

注释关键点应该具备**可读性**，有时候没有任何注释的代码确实让人痛苦，更可怕的是因为代码更新但是注释不更新，这样的事情也时有发生=-=。最好的注释应该是代码本身会告知作者意图。

> PS：个人看过的没啥注释也能完全看懂，并且看完觉得写的非常美的代码目前只有Redission和Spring。



# 10. 避免创建不必要对象

对象创建是最消耗内存的操作之一，这就是为什么最好的 Java 实践是避免创建任何不必要的对象，并且只应在需要时创建它们。



# 9. 返回空集合而不是null

> NPE is the most frequent exception in production with absolute leadership, do not give it a chance.

更进一步说是让整个系统不要出现空指针异常，不应该因为项目代码妥协老旧的编程风格。一定不要让空指针有可乘之机。除了返回空集合以外，还可以利用Optional的工具类包装有可能为Null的对象，显式的告诉调用者对象有可能为Null。

> An empty collection might have a different meaning than no collection at all.&#x20;

有时候null 也可能是“有意义”的，使用的时候也需要考虑具体情况，当然返回空集合肯定是没有任何问题的。



## Collections

空集合相关的轮子在Collections可以找到答案，但是需要注意需要合理使用，JDK5之后出现了emptyXXX等集合类，但是这些类“暗藏杀机”，因为它们的效果和**List.subList()** 一样是**immutable** 的。

虽然JDK的工具类可以减少不必要的对象创建，但是假设接收方需要在收到空集合之后却还要往里面设置数据，这时候毫无疑问就会抛异常了。

```java
   /**
     * Returns an empty list (immutable).  This list is serializable.
     *
     * <p>This example illustrates the type-safe way to obtain an empty list:
     * <pre>
     *     List&lt;String&gt; s = Collections.emptyList();
     * </pre>
     *
     * @implNote
     * Implementations of this method need not create a separate <tt>List</tt>
     * object for each call.   Using this method is likely to have comparable
     * cost to using the like-named field.  (Unlike this method, the field does
     * not provide type safety.)
     *
     * @param <T> type of elements, if there were any, in the list
     * @return an empty immutable list
     *
     * @see #EMPTY_LIST
     * @since 1.5
     */
    @SuppressWarnings("unchecked")
    public static final <T> List<T> emptyList() {
        return (List<T>) EMPTY_LIST;
    }
```

我认为`emptyList()`更为合理的方法名称是 `immutableEmptyList()` 和 `mutableEmptyList()`，当然`mutableEmptyList()`就是new新的`ArrayList()`这样的方法，看起来是毫无意义的，也许JDK设计的时候也是考虑这种情况？



# 8. 精度选择

> Most processors take almost same time in processing the operations on float and double but double offers way more precision then float that is why it is best practice to use double when precision is important otherwise you can go with float as it requires half space than double.。

大型处理器在处理double和float的时候计算速度是类似的。但是在Java中大部分情况只要是涉及浮点数计算都是闭着眼睛用BigDecimal。如果精度很重要直接无脑使用BigDecimal，double和float都会骗人。

> When precision is important, use BigDecimal. Double and Float aren’t accurate. They will betray you when you least expect it. 1 + 1 can be 1.99999999999.

当精度很重要时请使用BigDecimal，Double和Float并不精确。它们会在你最不期望的时候背叛你。1+1可能是1.99999999999。



# 1. 项目结构

有点过于基础了。下面是Maven的经典结构，如果是Maven新手可以看看项目的基本布局

-   *src/main/java*: **For source files**
-   *src/main/resources*: **For resource files, like properties**
-   *src/test/java*: **For test source files**
-   *src/test/resources*: **For test resource files, like properties**



# 2. 遵循命名规范

没啥好讲的，程序员的基础素质。最好的命名规范不是参考某一个标准，而是能统一风格布局代码。



# 3. 不要吞异常

在异常处理时在 catch 块中编写适当且有意义的消息是精英 java 开发人员首选的 java 最佳实践。**新手常常会把异常捕获之后不做任何处理，吞异常是非常危险的行为**。

得益于IDE的帮助，catch之后不打印任何信息的情况不是很多见，但是打印堆栈其实也是非常消耗资源的操作，同时因为是打印在控制台，如果不调用日志保存关键信息也有可能导致关键信息丢失。



# 16. 使用枚举替代静态常量【建议】

在Java推出枚举之前，定义一个常量基本只能使用下面的接口方式。在很多优秀框架的最早期版本中经常能看到这样的写法，并且到现在使用这种写法的不在少数 。

```java
public interface Color {
    public static final int RED = 0xff0000;
    public static final int BLACK = 0x000000;
    public static final int WHITE = 0xffffff;
}
```

> It’s because the purpose of interfaces is for inheritance and polymorphism, not for static stuffs like that. So the best practice recommends us to use an enum instead.&#x20;

这是因为接口的目的是用于继承和多态性，而不是用于类似的静态东西。所以最佳实践建议我们使用枚举来代替。

```java
public enum Color {
 
    BLACK(0x000000),
    WHITE(0xffffff),
    RED(0xff0000);
 
    private int code;
 
    Color(int code) {
        this.code = code;
    }
 
    public int getCode() {
        return this.code;
    }
}
```

使用枚举和静态常量的对比：

-   枚举更具备描述性
-   Keep It Simple, Stupid。接口可以用于继承和多态，枚举能干的事情基本是常量定义。
-   接口的静态常量往往有其他的用途。

> 很少有人会在枚举设计复杂的逻辑，因为枚举的可扩展性很差并且理解和学习成本较高



# 17. 按作用域划分成员变量

> The best practice to organize member variables of a class by their scopes from most restrictive to least restrictive.&#x20;

最好的做法是按其作用域从最严格到最不严格来组织一个类的成员变量。

```java
public class StudentManager {
    protected List<Student> listStudents;
    public int numberOfStudents;
    private String errorMessage;
    float rowHeight;
    float columnWidth;
    protected String[] columnNames;
    private int numberOfRows;
    private int numberOfColumns;
    public String title;
 
}
```

上面的成员变量定义眼花缭乱，通过作用域划分如下：

```java
public class StudentManager {
    private String errorMessage;
    private int numberOfColumns;
    private int numberOfRows;
 
 
    float columnWidth;
    float rowHeight;
 
    protected String[] columnNames;
    protected List<Student> listStudents;
 
    public int numberOfStudents;
    public String title;
 
}
```



# 18. 在数字字段中使用下划线【建议】

来自Java 7的小更新可以帮助我们把冗长的数字字段写得更易读。

```java
int maxUploadSize = 20971520;
long accountBalance = 1000000000000L;
float pi = 3.141592653589F;
```

和下面的代码作比较：

```java
int maxUploadSize = 20_971_520;
long accountBalance = 1_000_000_000_000L;
float pi = 3.141_592_653_589F。
```

哪一个更易懂？记得在数字字面中使用下划线，以提高代码的可读性。

> ~~个人：尴尬，学了这么多年Java居然不知道有这种写法.....~~



# 总结

这里罗列一下存在争议以及印象比较深的部分：

-   StringBuilder替换字符串拼接。如果单纯拼接几个变量或者方法的结果，"+"号是完全没有问题的。
    -   虽然JDK 9 对于每次for循环new StringBuilder的BUG做了修复，但国内广泛使用JDK8的场景下依然需要避免for循环大量的字符串拼接。
    -   这种优化不是关键，请更加关注合理的架构设计，因为这一点性能损耗对于现代处理器影响真的不大。
    -   如果不需要考虑多线程的情况，更建议使用StringBuffer。
-   精度选择，实际上不管有没有精度要求，碰到需要小数的运算建议使用BigDecimal，因为double和float的数学运算真的很容易出问题。
-   注释
    -   训练写出易懂的代码而不是易懂的注释
    -   注释应该遵从KISS。“愚蠢”的代码最好理解。
-   在大型的数字参数值中使用下划线，这是一个很容易忽略的Java小技巧，对于和金额有关的处理，这种写法非常有帮助。



# 参考

另一位读者：[Best coding practices every Java developer should follow?](https://medium.com/@lajthabalazs/best-coding-practices-every-java-developer-should-follow-45dfbbc9774f)

[10 Java Core Best Practices Every Java Programmer Should Know](https://www.codejava.net/coding/10-java-core-best-practices-every-java-programmer-should-know#NamingConvention)

