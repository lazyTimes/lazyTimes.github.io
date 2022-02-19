---
title: JDK 1.8 LAMBADA表达式
subtitle: jdk1.8中对于lambada表达式
author: lazytime
url_suffix: note27
tags:
  - java
categories:
  - java-java基础
keywords: lambada
description: 使用lambada表达式处理
copyright: true
date: 2020-09-25 23:57:19
---

# Lambada 表达式资料参考

## 简介：

现在的工作当中对于jdk1.8的lambada表达式使用越来越频繁，但是对于内部的一些细节却不是特别的清楚

这里借鉴了几篇博客，对比了之后对于博客到推荐进行了排序，同时也整理了这几篇博客提到的个人觉得比较重要到部分进行了验证

<!-- more -->

## 第一篇

**https://objcoding.com/2019/03/04/lambda/** 一篇博客，介绍的比较到位，虽然文章很长但是通篇看完完全没有想点右上角的冲动

## 第二篇

https://juejin.im/post/5abc9ccc6fb9a028d6643eea

关于lambada的十个案例

### 重点部分：

#### Q：如何在lambda表达式中加入谓词？

### A：解答

首先代码如下，按照个人理解，其实本质就是把用户的操作“拼接”，包括筛选，合并等操作，依赖`java.util.function.Predicate`此接口实现

```
// 甚至可以用and()、or()逻辑函数来合并Predicate，
// 例如要找到所有以J开始，长度为四个字母的名字，你可以合并两个Predicate并传入
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
    .filter(startsWithJ.and(fourLetterLong))
    .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n)); 
```

## 第三篇

https://segmentfault.com/a/1190000009186509

关于lambada的一个详解

> 版权声明：本文由*吴仙杰*创作整理，转载请注明出处：https://segmentfault.com/a/1190000009186509

## 第四篇

https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html

Oracale官方案例

https://blog.csdn.net/renfufei/article/details/24600507

比较久远到一个文章，更多是批判的角度看待lambada

## 概览

1. 介绍了stream 的基本概念
   1. 简写的依据 - 函数接口 和类型推断机制
2. 函数式编程和匿名内部类的区别
   1. 使用上区别
   2. JVM上的本质区别
3. JDK8 对于容器到扩展方法，其中部分是为了配合函数式编程而出现的
   1. Collections
   2. Map
4. Stream API 介绍stream 的常用方法以及基本特性
   1. `sort()`
   2. 
5. stream内部实现的简单了解
   1. 用户的操作如何记录？
   2. 操作如何叠加？
   3. 叠加之后的操作如何执行？
   4. 执行后的结果（如果有）在哪里？

## 重点部分

### Q：匿名内部类和lambada表达式的关键区别

### A 回答

#### **this**的指向对象

- 匿名内部类：`this` 指向的是匿名内部类的所属对象
- lambada：`this` 指向当前运行的类，也就是当前运行的对象

#### jvm层面

- 匿名内部类：会生成一个 `$1`到匿名内部类的对象，使用`new`指令生成对象并且执行
- lambada：会生成一个私有方法，使用`invokedynamic`指令调用

匿名内部类的编译过程

为了更好到理解，自己做了一个实验

1. 创建一个基本的匿名内部类到方法

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200505103819.png?ynotemdtimestamp=1601045594695)

代码如下：

```
/**
     * 匿名内部类和lambada的区别
     */
    @Test
    public void test1() throws InterruptedException {
        new Thread(new Runnable() {
            public void run() {
                String name = Thread.currentThread().getName();
                System.err.println(name + " is run as a thread ");
            }
        }).start();
        Thread.sleep(2000);

    }
```

1. 查看classes ，可以看到编译后的class 文件如下，看到明显多出来一个`LambadaTest1$1.class` 这事啥东西？

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200505103923.png?ynotemdtimestamp=1601045594695)

1. 打开`cmd`使用`cd`命令移动到改目录下面

> 小技巧1：
>
> 编译后到classes 如果想要直接查看class 文件的反编译到内容，可以直接把class 文件拖拽到 idea 里面进行打开，就可以看到class 到反编译内容~~~

1. 反编译的内容如下，可以看到匿名内部类的编译后的内容经过解释之后，**实际上是创建了一个匿名内部类的对象**

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200505104143.png?ynotemdtimestamp=1601045594695)

1. 在第一步的基础上，我们除了打印线程的名称，还可以打印`this` 看一下匿名内部类的this 是啥东西，我打印之后得到到匿名内部类的 this 如下: `com.jdk.lambada.LambadaTest1$1@2fd014c6`

> 小技巧2：
>
> 如何使用javap 查看java的汇编指令呢？
>
> 以上面到案例为例：
>
> 1. 打开`cmd`，进入对应的class目录，我们可以执行`javap <options> <classes>`的格式进行翻译操作
> 2. 执行`javap -c -l -p xxx`，得到结果:
>
> 选项的含义？
>
> ```
>  -help  --help  -?        输出此用法消息
>  -version                 版本信息，其实是当前javap所在jdk的版本信息，不是class在哪个jdk下生成的。
>  -v  -verbose             输出附加信息（包括行号、本地变量表，反汇编等详细信息）
>  -l                         输出行号和本地变量表
>  -public                    仅显示公共类和成员
>  -protected               显示受保护的/公共类和成员
>  -package                 显示程序包/受保护的/公共类 和成员 (默认)
>  -p  -private             显示所有类和成员
>  -c                       对代码进行反汇编
>  -s                       输出内部类型签名
>  -sysinfo                 显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列)
>  -constants               显示静态最终常量
>  -classpath <path>        指定查找用户类文件的位置
>  -bootclasspath <path>    覆盖引导类文件的位置
> ```

1. 为了更进一步了解底层的jvm指令执行，使用`javap -c -l -p xxx`反编译结果如下

```
Compiled from "LambadaTest1.java"
public class com.jdk.lambada.LambadaTest1 {
  public com.jdk.lambada.LambadaTest1();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
    LineNumberTable:
      line 11: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       5     0  this   Lcom/jdk/lambada/LambadaTest1;

  public void test1() throws java.lang.InterruptedException;
    Code:
       0: new           #2                  // class java/lang/Thread
       3: dup
       4: new           #3                  // class com/jdk/lambada/LambadaTest1$1
       7: dup
       8: aload_0
       9: invokespecial #4                  // Method com/jdk/lambada/LambadaTest1$1."<init>":(Lcom/jdk/lambada/LambadaTest1;)V
      12: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      15: invokevirtual #6                  // Method java/lang/Thread.start:()V
      18: ldc2_w        #7                  // long 2000l
      21: invokestatic  #9                  // Method java/lang/Thread.sleep:(J)V
      24: return
    LineNumberTable:
      line 19: 0
      line 24: 15
      line 25: 18
      line 27: 24
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      25     0  this   Lcom/jdk/lambada/LambadaTest1;
}
```

可以发现，匿名内部类使用了 `new`到指令来实现匿名对象的生成，调用`invokespecial`执行内部的`run`方法

接下来我们看下Lambada简写上面的匿名内部类代码

lambada表达式编译过程

1. 和上面的流程类似，这里直接略过一些不必要的点，代码如下

```
import java.util.concurrent.TimeUnit;

/**
 * @program: lambada
 * @description: 测试jdk1.8
 * @author: zhaoxudong
 * @create: 2020-05-05 11:42
 **/
public class Test {

    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            System.err.println(" test" + Thread.currentThread().getName() + " is run as a thread " );
        }).start();
        TimeUnit.SECONDS.sleep(5L);
    }
}
```

1. 同样的，我们先把class文件放到`idea`下面查看反编译之后到代码

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

import java.util.concurrent.TimeUnit;

public class Test {
    public Test() {
    }

    public static void main(String[] args) throws InterruptedException {
        (new Thread(() -> {
            System.err.println(" test" + Thread.currentThread().getName() + " is run as a thread ");
        })).start();
        TimeUnit.SECONDS.sleep(5L);
    }
}
```

1. 打开class所在目录，我们会发现lambada并没有像匿名内部类一样创建了一个$1 的对象，那么他是如何实现的？
2. 我们使用`javap -c -l -p xxx` 翻译class 得到到结果如下

```
public class Test {
  public Test();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
    LineNumberTable:
      line 9: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       5     0  this   LTest;

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: new           #2                  // class java/lang/Thread
       3: dup
       4: invokedynamic #3,  0              // InvokeDynamic #0:run: ()Ljava/lang/Runnable;
       9: invokespecial #4                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      12: invokevirtual #5                  // Method java/lang/Thread.start:()V
      15: getstatic     #6                  // Field java/util/concurrent/TimeUnit.SECONDS:Ljava/util/concurrent/TimeUnit;
      18: ldc2_w        #7                  // long 5l
      21: invokevirtual #9                  // Method java/util/concurrent/TimeUnit.sleep:(J)V
      24: return
    LineNumberTable:
      line 12: 0
      line 14: 12
      line 15: 15
      line 16: 24
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      25     0  args   [Ljava/lang/String;

  private static void lambda$main$0();
    Code:
       0: getstatic     #10                 // Field java/lang/System.err:Ljava/io/PrintStream;
       3: new           #11                 // class java/lang/StringBuilder
       6: dup
       7: invokespecial #12                 // Method java/lang/StringBuilder."<init>":()V
      10: ldc           #13                 // String  test
      12: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      15: invokestatic  #15                 // Method java/lang/Thread.currentThread:()Ljava/lang/Thread;
      18: invokevirtual #16                 // Method java/lang/Thread.getName:()Ljava/lang/String;
      21: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      24: ldc           #17                 // String  is run as a thread
      26: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      29: invokevirtual #18                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      32: invokevirtual #19                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      35: return
    LineNumberTable:
      line 13: 0
      line 14: 35
}
```

对比匿名内部类和lambada表达式可以明显发现，Lambada的不同在于使用了`私有方法` 加上 `invokedynamic`指令进行编译，而不会产生一个匿名内部类的对象class文件。

1. 我们可以看下Lambada的 `this`打印的是什么东西

```
public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            System.err.println(" test" + Thread.currentThread().getName() + " is run as a thread " );
            System.err.println("lambada this is = " + this);
        }).start();
        TimeUnit.SECONDS.sleep(5L);
    }
```

**这里惊讶的发现,编译居然无法通过！！！！**

我们再对比使用匿名内部类的方式：

```
public static void main(String[] args) throws InterruptedException {
    new Thread(new Runnable() {
        public void run() {
            String name = Thread.currentThread().getName();
            System.err.println(name + " is run as a thread " + this);
        }
    }).start();
    Thread.sleep(2000);
}
```

此方式可以正常编译通过并且运行，由此我们可以发现lambada的`this` 和匿名内部类的 `this`有着本质的区别

1. 我们无法使用正常到方式打印this, 我们换一种方式，这里稍微调整代码，最后的代码如下:

```
/**
 * @program: lambada
 * @description: 测试jdk1.8
 * @author: zhaoxudong
 * @create: 2020-05-05 11:42
 **/
public class Test {

    Runnable run = () ->{
        System.err.println(this);
    };

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Test().run).start();
        TimeUnit.SECONDS.sleep(5L);
    }

    @Override
    public String toString() {
        return "test";
    }
}
```

执行上面的代码，可以得到结果`test`

> 在匿名内部类如何得到结果呢？
>
> 可以看下面的代码，结果依然是打印 `test`

```
/**
 * @program: smartframwork
 * @description: lambada表达式学习1
 * @author: zhaoxudong
 * @create: 2020-05-05 10:29
 **/
public class LambadaTest1 {


    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            public void run() {
                String name = Thread.currentThread().getName();
                System.err.println(name + " is run as a thread " + this);
            }

            @Override
            public String toString() {
                return "test";
            }
        }).start();
        Thread.sleep(2000);
    }


    @Override
    public String toString() {
        return "LambadaTest1{}";
    }
}
```

结论：

匿名内部类：`this` 指向的是匿名内部类的所属对象（**也就是Runnable**）

lambada：`this` 指向当前运行的类（**也就是Test**），也就是当前运行的对象

### Q：Java8里面的java.util.Spliterator接口有什么用？

### A：参考博客

[思否](https://segmentfault.com/q/1010000007087438)

一句话概括：为了多线程并行遍历元素而设计到迭代器

版本：JDK1.8

目的：简化复杂到并行迭代程序编写

参考：Java7的Fork/Join(分支/合并)框架

[CSDN上面到解答](https://blog.csdn.net/silver9886/article/details/87917823)

[JAVA8 stream 中Spliterator的使用(一)](https://ifeve.com/java8-stream-中spliterator的使用一/)

[JAVA8 stream 中Spliterator的使用(二)](https://ifeve.com/java8-stream-中spliterator的使用二/)