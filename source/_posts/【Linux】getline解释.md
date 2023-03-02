---
title: 【Linux】getline解释
subtitle: getline使用
url_suffix: getlines
author: lazytime
tags:
  - Linux
categories:
  - Linux
keywords: getline介绍）
description: getline介绍）
copyright: true
date: 2023-02-28 17:18:00
---

# 知识点

先说一下重要的知识点：
- `getline;`在`awk`中可以用于控制“循环”。
- `getline;`之后，awk会改变对应的NF，NR，FNR和$0等内部变量
- `getline;`拿到的是**下一行**而不是当前行，概念想象为`++i`操作即可。

# 介绍

getline命令改变了awk的运行逻辑，是awk命令不可或缺的一部分。awk本质上就是一个for循环，它每次对输入文件的一行进行处理，然后转而执行下一行，直到整个文件的每一行都被执行完毕。整个过程是自动的无需做什么。

`getline`命令却可以让你去**控制循环**。当然，getline命令执行后，awk会设置NF，NR，FNR和$0等这些**内部变量**。

<!-- more -->

# 简单使用

我们先看一个简单的例子，打印出从1到10之间的偶数：

```ruby
[zxd@localhost kafka2.8.X]$ seq 10 | awk '{getline; print $0}'
2
4
6
8
10
```

这个命令的的执行逻辑是执行一个for循环从1到10，在循环内部先执行`getline;`然后打印`$0`，`$0`指向的就是当前的变量，注意`getline;`获取的是**获取当前行的下一行**，类似我们编程语言的`++i`，注意`getline;`之后，awk会改变对应的NF，NR，FNR和$0等内部变量，所以`$0`值会随着遍历改变，最后实现打印偶数效果。

根据上面的介绍我们可以推导出打印奇数的逻辑：

```sh
[zxd@localhost kafka2.8.X]$ seq 10 | awk '{ print $0;getline;}'
1
3
5
7
9
```

# 临时变量使用

奇偶行对调打印，原来在奇数行的内容将其打印在偶数行，原来在偶数行的内容将其打印在奇数行，要实现这个功能，需要在循环中使用临时变量：

```sh
seq 10 | awk '{getline tmp; print tmp; print $0}'
```

结果如下：

```sh
[zxd@localhost kafka2.8.X]$ seq 10 | awk '{getline tmp; print tmp; print $0}'
2
1
4
3
6
5
8
7
10
9

```

# 文件合并

在上面的例子当中`tmp`变量是不会改变的。

getline也可以从另外一个文件中读取内容。下面例子实现将两个文件的每一行都打印在一行上：

```sh
vim b.txt
1
2
3
4
5
vim c.txt
5
6
7
8
9
10
[zxd@localhost ~]$ awk '{printf "%s ", $0; getline < "c.txt"; print $0}' b.txt 
1 6
2 7
3 8
4 9
5 10
```

# 日期获取

getline也可以用来执行一个UNIX命令，并得到它的输出。下面例子通过getline得到系统的当前时间：

```sh
awk 'BEGIN {"date" | getline; close("date"); print $0}'
```

```sh
[zxd@localhost ~]$ awk 'BEGIN {"date" | getline; close("date"); print $0}'
Wed Mar  1 00:34:01 CST 2023
```

# 参考资料

[# awk getline命令解析](https://blog.csdn.net/xibeichengf/article/details/51367311)