---
title: '【Linux】How to Use the Stdin, Stderr, and Stdout Streams in Bash'
subtitle: 有关Linux三种流介绍
author: lazytime
url_suffix: stdinouterr
tags:
  - Linux
categories:
  - Linux
keywords: 流介绍
description: 流介绍
copyright: true
date: 2023-03-15 14:49:31
---
# How to Use the Stdin, Stderr, and Stdout Streams in Bash

## 目录

-   [原文](#原文)
-   [引言](#引言)
    -   [Redirection Operators of Stdin, Stdout, and Stderr](#Redirection-Operators-of-Stdin-Stdout-and-Stderr)
-   [Uses of Stdin, Stdout, and Stderr](#Uses-of-Stdin-Stdout-and-Stderr)
-   [Example 1: Use of Stdin](#Example-1-Use-of-Stdin)
    -   [>、>>、< 等操作符使用](#-等操作符使用)
-   [Example 2: Use of Stdout](#Example-2-Use-of-Stdout)
    -   [pipe (|) 管道符和stdout](#pipe--管道符和stdout)
-   [Example 3: Use of Stdin and Stdout](#Example-3-Use-of-Stdin-and-Stdout)
-   [Example 4: Use of Stderr](#Example-4-Use-of-Stderr)

# 原文

[How to Use the Stdin, Stderr, and Stdout Streams in Bash – Linux Consultant](https://www.linuxconsultant.org/how-to-use-the-stdin-stderr-and-stdout-streams-in-bash/ "How to Use the Stdin, Stderr, and Stdout Streams in Bash – Linux Consultant")

# 引言

当Linux操作系统启动时，将会有三个流被打开。它们是**stdin**、**stdout**和**stderr**。

**stdin** 的全称是标准输入，用于接受用户的输入。

**stdout** 的完整形式是标准输出，用于将命令的输出存储到stdout流中。

**stderr** 的完整形式是标准错误，用于将任何命令产生的错误信息存储到数据流中。

**stdin**、**stdout** 和 **stderr** 的相应数字标识符值为**0**、**1**和**2**。

<!-- more -->

## Redirection Operators of Stdin, Stdout, and Stderr 

Stdin、Stdout和Stderr的重定向操作符

重定向符号使用：&#x20;

-   "<" 或  "0<"用于stdin流。
-   ">" 或 "1>"用于stdout流。
-   "2"用于stderr流。

# Uses of Stdin, Stdout, and Stderr

下面是一些使用stdin，stdout，stderr的使用案例。

取出文件的内容并在终端打印的方法在这个例子中显示。

# **Example 1: Use of Stdin**

### >、>>、< 等操作符使用

运行下面的 "cat "命令，创建一个名为**testdata.txt**的带有一些内容的文本文件。

```bash
$ cat > testdata.txt
```

注意输入上面的命令之后，此时shell会等待输入流进行输入，此时可以再控制台随意输入一些字符，之后按键ctrl + c**的方式结束输入，此时`ls`当前可以看到会出现新文件 testdata.txt。

注意如果我们重复执行此命令，那么每次新的输入都会 **覆盖掉旧的输入。**

```bash
ubuntu@VM-8-8-ubuntu:~$ cat > testdata.txt
abcdefg
^C
ubuntu@VM-8-8-ubuntu:~$ cat testdata.txt 
abcdefg

```

运行下面的 "cat "命令，将一些内容**追加**到testdata.txt文件中。

```bash
$ cat >> testdata.txt
```

下面是实践代码：

```bash
ubuntu@VM-8-8-ubuntu:~$ cat testdata.txt 
abcdefg
ubuntu@VM-8-8-ubuntu:~$ cat >> testdata.txt
hijklmn
^C
ubuntu@VM-8-8-ubuntu:~$ cat testdata.txt 
abcdefg
hijklmn
```

运行下面的 "cat "命令，从testdata.txt文件中获取一个输入流，并将其打印到终端。

```bash
$ cat < testdata.txt
```

从表面效果上来看此命令实际上只是执行了`<` 符号前面的命令而已。但是在后续的案例中，将会介绍如何读入输入流重定向到另一个输出流：

```bash
ubuntu@VM-8-8-ubuntu:~$ ls < testdata.txt 
testdata2.txt  testdata.txt
ubuntu@VM-8-8-ubuntu:~$ ls -l < testdata.txt 
total 8
-rw-rw-r-- 1 ubuntu ubuntu 23 Mar 14 13:06 testdata2.txt
-rw-rw-r-- 1 ubuntu ubuntu 16 Mar 14 13:13 testdata.txt
```

**Output:**

> The following output appears after executing the previous commands after adding the string, “[linuxhint.com](http://linuxhint.com "linuxhint.com")”, and “Scripting Language” into the testdata.txt file:

输出：

在英文原文的案例中，在testdata.txt文件中加入 "[linuxhint.com](http://linuxhint.com "linuxhint.com") "和 "脚本语言 "这两个字符串后，执行前面的命令会出现以下输出。

![image-20230315143349057](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/image-20230315143349057.png)

> The method of creating a file using pipe (|) and redirection operator is shown in this example.

除了上面几种方法外，还可以使用管道（|）和重定向操作符创建文件。

# **Example 2: Use of Stdout**

### pipe (|) 管道符和stdout

下面是一个使用管道符重定向输出并且创建文件的例子。

> Run the following command to write a string data into the text file named testdata2.txt by piping. The output of the “echo” command is sent to the input of the “cat” command using the pipe (|) operator:

通过下面的命令，把echo的命令发送到cat当中，最后重定向输出流到文件testdata2.txt。

```bash
ubuntu@VM-8-8-ubuntu:~$ echo "Learn Bash Programming" | cat > testdata2.txt
ubuntu@VM-8-8-ubuntu:~$ ls
testdata2.txt
```

> Run the following “cat” command to check the content of the testdata2.txt file:

再次运行 "cat "命令检查testdata2.txt文件的内容。

```bash
ubuntu@VM-8-8-ubuntu:~$ cat testdata2.txt 
Learn Bash Programming
```

> **Output:**

> The following output appears after executing the previous commands. According to the output, the output of the “echo” command is written into the testdata2.txt file:

输出：

下面的例子可以可以看到echo的输出内容被重定向写入到testdata2.txt 这个文件当中，下面是合并两个命令的输出结果：

```bash
ubuntu@VM-8-8-ubuntu:~$ echo "Learn Bash Programming" | cat > testdata2.txt
ubuntu@VM-8-8-ubuntu:~$ ls
testdata2.txt
ubuntu@VM-8-8-ubuntu:~$ cat testdata2.txt 
Learn Bash Programming

```

> Run the following command to write the output of the “ls –l” command into a text file named list.txt using the redirection operator (‘>’):

运行以下命令，使用重定向操作符（'`>`'）将 `ls -l `命令的输出写入一个名为list.txt的文本文件中。

```bash
ls -l  > list.txt
```

个人的实验结果如下：

```bash
ubuntu@VM-8-8-ubuntu:~$ ls -l  > list.txt
ubuntu@VM-8-8-ubuntu:~$ cat list.txt 
total 8
-rw-rw-r-- 1 ubuntu ubuntu  0 Mar 14 13:18 list2.txt
-rw-rw-r-- 1 ubuntu ubuntu  0 Mar 14 13:22 list.txt
-rw-rw-r-- 1 ubuntu ubuntu 23 Mar 14 13:06 testdata2.txt
-rw-rw-r-- 1 ubuntu ubuntu 16 Mar 14 13:13 testdata.txt

```

# Example 3: Use of Stdin and Stdout

> The method of using both stdin and stdout to take an input from a file and write it into a file is shown in this example.

这部分介绍了如何同时使用stdin和stdout。

> Run the following “cat” command to take the content of the testdata.txt file and write it into the testfile.txt file and the terminal:

下面的cat命令可以把testdata.txt文件的内容打印到控制台，同时重定向输出流写入文件到另一个文件：

```bash
ubuntu@VM-8-8-ubuntu:~$ cat < testdata.txt > otherfile.txt
ubuntu@VM-8-8-ubuntu:~$ cat testdata.txt 
abcdefg
hijklmn
ubuntu@VM-8-8-ubuntu:~$ cat otherfile.txt 
abcdefg
hijklmn

```

>  PS：为了方便理解，建议读者把上面的命令分为两个操作，类似这样的写法：`( cat < testdata.txt ) > otherfile.txt`。

上面的命令可以看作两个部分，第一部分是读取testdata.txt的内容作为输入流，然后输出再输出到 otherfile.txt。最终两个文件内容是一样的，这个操作的命令效果和CP复制一个文件的效果类似：

```bash
ubuntu@VM-8-8-ubuntu:~$ cp otherfile.txt otherfile2.txt
```



# Example 4: Use of Stderr

> The content of the standard error can be printed in the terminal or redirected into a file or sent to the /dev/null that works like the recycle bin. The different ways to pass the standard error are shown in this example

stderr是标准错误信息，通常的做法是输出到控制台或者输出到文件，还有一种方式是丢弃到**/dev/null**这个“黑洞”当中，下面的例子是stderr的用法案例：

下面的命令是正确的，它用换行符打印了 "Hello "字符串。所以下面的命令**没有产生标准错误**。

```bash
ubuntu@VM-8-8-ubuntu:~$ printf "%s\n" "Hello"
Hello
```

我们可以通过 `echo $?` 的方式，检查上一个命令是否正确：

```bash
ubuntu@VM-8-8-ubuntu:~$ echo $?
0
```

下面的命令是错误的，因为没有名为 "`pirntf` "的命令。所以它产生了一个**标准错误**，并且错误被打印控制台。

```bash
ubuntu@VM-8-8-ubuntu:~$ pirntf "%s\n" "Hello"
Command 'pirntf' not found, did you mean:
  command 'printf' from deb coreutils (8.32-4.1ubuntu1)
Try: sudo apt install <deb name>

```

这时候检查上一个命令是否正确会，结果返回一个非0值代表上一条命令有误：

```bash
ubuntu@VM-8-8-ubuntu:~$ echo $?
127
```

> Sometimes, it requires printing the custom error by hiding the standard error to make the error more understandable for the users. This task can be done by redirecting the error into the /dev/null. The “2>” is used here to redirect the error into /dev/null.

有时，控制台需要通过隐藏标准错误来打印自定义错误，使用户更容易理解错误，这个任务可以通过将错误重定向到`/dev/null`中来完成。这里使用 "`2>`"来重定向错误到`/dev/null`。

```bash
ubuntu@VM-8-8-ubuntu:~$ pirntf "%s\n" "Hello" 2> /dev/null
ubuntu@VM-8-8-ubuntu:~$ echo $?
127

```

> Sometimes, the standard error requires storing into a file for future use. This task can be done by redirecting the error into a file using the “2>” operator.

有时，标准错误需要存储到一个文件中提供给以后使用（日志备份）。这项任务同样可以通过使用 "2>"操作符将错误重定向到一个文件中来完成。

```bash
ubuntu@VM-8-8-ubuntu:~$ pirntf "%s\n" "Hello" 2> error.txt
ubuntu@VM-8-8-ubuntu:~$ cat error.txt 
Command 'pirntf' not found, did you mean:
  command 'printf' from deb coreutils (8.32-4.1ubuntu1)
Try: sudo apt install <deb name>

```

从结果可以看到在执行命令后标准错误被正确写入error.txt文件。

# 总结

> The uses of stdin, stdout, and stderr are explained in this tutorial using multiple examples that will help the Linux users to understand the concept of these streams and use them properly when required.

本教程用多个例子解释了stdin、stdout和stderr的用途，这将有助于Linux用户理解这些流的概念，并在需要时正确使用它们。

