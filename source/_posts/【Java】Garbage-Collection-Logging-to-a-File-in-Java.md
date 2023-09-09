---
title: 【Java】Garbage Collection Logging to a File in Java
subtitle: 垃圾收集器优质文章介绍
author: lazytime
url_suffix: engctojava
tags:
  - Java
categories:
  - Java
keywords: 垃圾收集,gc
description: gc
copyright: true
date: 2023-09-04 14:35:23
---
# 原文

[https://www.baeldung.com/java-gc-logging-to-file](https://www.baeldung.com/java-gc-logging-to-file)

## [1. Overview](https://www.baeldung.com/java-gc-logging-to-file#overview)

Garbage collection is a marvel of the Java programming language providing us with automatic memory management. 

垃圾回收是 Java 编程语言的一个奇迹，它为我们提供了自动内存管理功能。

Garbage collection hides the details of having to manually allocate and deallocate memory. 

垃圾回收隐藏了手动分配和删除内存的细节。

While this mechanism is fantastic, sometimes it doesn't work the way we want. 

虽然这种机制非常好，但有时并不能如我们所愿。

In this tutorial, we'll explore Java's **logging options for garbage collection statistics** and discover how to **redirect these statistics to a file**.

在本教程中，我们将探索 Java 的**垃圾收集统计日志选项**，并了解如何**将这些统计信息重定向到文件**。



<!-- more -->



## 2. GC Logging Flags in Java 8 and Earlier[](https://www.baeldung.com/java-gc-logging-to-file#gc-logging-flags-in-java-8-and-earlier)

First, let's explore the JVM flags relating to GC logging in Java versions prior to Java 9.

首先，让我们探讨 Java 9 之前的 Java 版本中与 GC 日志相关的 JVM 标志。

### [2.1. _-XX:+PrintGC_](https://www.baeldung.com/java-gc-logging-to-file#1--xxprintgc)

The _-XX:+PrintGC_ flag is an alias for _-verbose:gc_ and **turns on basic GC logging**. 

_-XX:+PrintGC_ 标志是_-verbose:gc_的别名，作用是**开启基本的 GC 日志**。

In this mode, a single line is printed for every young-generation and every full-generation collection. 

在这个模式中，每个年轻代和完整代的收集操作都会打印一行。

Let's now turn our attention to providing detailed GC information.

现在，让我们把注意力转向提供详细的 GC 信息。

### [2.2. _-XX:+PrintGCDetails_](https://www.baeldung.com/java-gc-logging-to-file#2--xxprintgcdetails)

Similarly, we have the flag _-XX:+PrintGCDetails_ used to **activate detailed GC logging** instead of _-XX:+PrintGC_.

同样，我们使用标记 _-XX:+PrintGCDetails_ 来**激活详细的 GC 日志**，而不是 _-XX:+PrintGC_。

Note that the output from _-XX:+PrintGCDetails_ changes depending on the GC algorithm in use.

> 请注意，_-XX:+PrintGCDetails_ 的输出会根据使用的 GC 算法而改变。

Next, we'll look at annotating our logs with date and time information.

> 接下来，我们将了解如何用日期和时间信息来注释日志。
### [2.3. _-XX:+PrintGCDateStamps_ and _-XX:+PrintGCTimeStamps_](https://www.baeldung.com/java-gc-logging-to-file#3--xxprintgcdatestamps-and--xxprintgctimestamps)

We can **add dates and timing information to our GC logs** by utilizing the flags _-XX:+PrintGCDateStamps_ and _-XX:+PrintGCTimeStamps_, respectively.

我们可以分别利用标记 _-XX:+PrintGCDateStamps_ 和 _-XX:+PrintGCTimeStamps_ 在 GC 日志中**添加日期和时间信息。

First, _-XX:+PrintGCDateStamps_ adds the date and time of the log entry to the beginning of each line.

首先，_-XX:+PrintGCDateStamps_ 会在每行开头添加日志条目的日期和时间。

Second, _-XX:PrintGCTimeStamps_ adds a timestamp to every line of the log detailing the time passed (in seconds) since the JVM was started.

其次，_-XX:PrintGCTimeStamps_ 会在日志的每一行添加一个时间戳，详细记录 JVM 启动后的时间（以秒为单位）。
### [2.4. _-Xloggc_](https://www.baeldung.com/java-gc-logging-to-file#4--xloggc)

Finally, we come to **redirecting the GC log to a file**. 

最后，我们来**将 GC 日志重定向到文件**。

This flag takes an optional filename as an argument using the syntax _-Xloggc:file_ and without the presence of a file name the GC log is written to standard out.

该标志使用 _-Xloggc:file_ 语法将一个可选的文件名作为参数，如果没有文件名，GC 日志将被写入标准输出。

Additionally, this flag also sets the _-XX:PrintGC_ and _-XX:PrintGCTimestamps_ flags for us. Let's look at some examples:

此外，该标记还会为我们设置 _-XX:PrintGC_ 和 _-XX:PrintGCTimestamps_ 标记。让我们来看几个例子：

If we want to write the GC log to standard output, we can run:

如果我们想将 GC 日志写入标准输出，可以运行：

```bash
java -cp $CLASSPATH -Xloggc mypackage.MainClass
```

Or to write the GC log to a file, we would run:

如果要写入GC日志到一个文件，使用下面参数

```sh
java -cp $CLASSPATH -Xloggc:/tmp/gc.log mypackage.MainClass
```

## [3. GC Logging Flags in Java 9 and Later](https://www.baeldung.com/java-gc-logging-to-file#gc-logging-flags-in-java-9-and-later)

In Java 9+, _-XX:PrintGC_, the alias for _-verbose:gc_, has been deprecated in favor of the **unified logging option, _-Xlog_**. 

Java9+ 的版本，_-Xlog_ 取代了  _-XX:PrintGC_ 和别名 _-verbose:gc_，此前的两个参数均已经标记为“已弃用”

All other GC flags mentioned above are still valid in Java 9+. 

当然，虽然被标记弃用，但是上述所有其他 GC 标志在 Java 9+ 中仍然有效。

This new logging option allows us to **specify which messages should be shown, set the log level, and redirect the output**.

这个新的日志选项允许我们**指定应显示哪些信息、设置日志级别和重定向输出**。

We can run the below command to see all the available options for log levels, log decorators, and tag sets:

我们可以使用下面的命令查看所有可用的参数，比如日志等级，日志装饰器和标记集：

```bash
java -Xlog:logging=debug -version
```

For example, if we wanted to log all GC messages to a file, we would run:

如果我们想要记录GC日志到一个外部文件，可以使用下面的启动参数。

```bash
java -cp $CLASSPATH -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

Additionally, this new unified logging flag is repeatable, so you can, for example, **log all GC messages to both standard out and a file**:

此外，这一新的统一日志标记（-Xlog:gc）是可重复的，因此您可以**将所有 GC 消息同时记录到标准输出和文件**中：

```bash
java -cp $CLASSPATH -Xlog:gc*=debug:stdout -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

## [4. Conclusion](https://www.baeldung.com/java-gc-logging-to-file#conclusion)

In this article, we've shown how to log garbage collection output in both Java 8 and Java 9+ including how to redirect that output to a file.

本节内容展示了开启收集打印日志在JDK8和JDK9+的区别，包括如何将输出重定向到文件等操作。