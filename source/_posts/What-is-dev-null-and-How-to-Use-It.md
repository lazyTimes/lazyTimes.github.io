---
title: What is /dev/null and How to Use It
subtitle: 黑洞的概念了解
author: lazytime
url_suffix: devnull
tags:
  - Linux
categories:
  - Linux
keywords: Linux知识点
description: Linux知识点
copyright: true
date: 2023-02-27 07:35:24
---
# What is /dev/null and How to Use It

[What is /dev/null and How to Use It (linuxhint.com)](https://linuxhint.com/what_is_dev_null/)

# 前置知识

了解/dev/null需要先了解stdin、stdout等Linux输入输出流的概念：

[How to Use the Stdin, Stderr, and Stdout Streams in Bash (linuxhint.com)](https://linuxhint.com/bash_stdin_stderr_stdout/)

这里简单概括一下，默认情况下我们执行一个shell程序都会获得两种输出流，**标准输出**和 **（标准）错误输出**，分别叫做 **stdout** 以及 **stderr** 。

例如以下命令将打印出双引号内的字符串。在这里输出结果存储在标准输出中：

```sh
xander@xander:~$ echo "hello world"
hello world

```

下一个命令将向我们显示先前运行的命令的退出状态。

```sh
xander@xander:~$ echo $?
0
```

如果上一条指令存在错误输出，则结果是一个非0的值（不一定是127）

```sh
xander@xander:~$ echo $?
127
```

如果是正确内容，命令的返回结果会是0：

```sh
xander@xander:~$ lll
Command 'lll' not found, did you mean:
  command 'lld' from deb lld (1:14.0-55~exp2)
  command 'dll' from deb brickos (0.9.0.dfsg-12.2)
  command 'lli' from deb llvm-runtime (1:14.0-55~exp2)
  command 'llt' from deb storebackup (3.2.1-2)
  command 'llc' from deb llvm (1:14.0-55~exp2)
Try: sudo apt install <deb name>
xander@xander:~$ echo $?
127
xander@xander:~$ echo "hello world"

xander@xander:~$ echo $?
0

```

<!-- more -->

# 文件描述符

在Linux当中存在文件描述符，所有 sdtout（=1） 和 stderr（=2） 都对应了特殊的文件描述符，前面的实验中我们没有指定文档描述符。如果未指定描述符，bash 将**默认使用 stdout**。也就是常说的标准输出。

**标准输出**

```sh
echo “Hello World” > log.txt
```

输出结果如下

```sh
xander@xander:~$ echo “Hello World” > log.txt
xander@xander:~$ cat log.txt 
“Hello World”

```

**错误输出**

```sh
asdasdasds 2> error.txt
```

输出结果如下

```sh
xander@xander:~$ asdasdasds 2> error.txt
xander@xander:~$ cat error.txt 
asdasdasds: command not found

```

# 重定向输出流到/dev/null

现在我们准备学习如何使用 `/dev/null`。首先让我们看看如何过滤正常输出和错误。在以下命令中，grep 将尝试在“/sys”目录中搜索字符串（在本例中为 hello）。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230225072922.png)

`/sys/`通常只有具备root权限的角色才能访问，使用一般用户会出现无权限访问的页面，现在可以通过重定向输出流到 `/dev/null`，会发现所有`grep`错误流信息全部被丢弃。

```sh
xander@xander:~$ grep -r hello /sys/ 2> /dev/null
xander@xander:~$ 
```

## ping google.com

有时候我们只想看错误的输出结果，可以把标准输出丢弃到`/dev/null`：

```sh
ping google.com 1> /dev/null
```

```sh
xander@xander:~$ ping google.com
PING google.com (142.251.42.238) 56(84) bytes of data.
--- google.com ping statistics ---
210 packets transmitted, 0 received, 100% packet loss, time 214004ms
```


# 重定向所有流到/dev/null

在某些情况下，输出可能根本没有用，使用重定向，我们可以将所有输出转储到空白中。

```sh
grep -r hello /sys/ > /dev/null 2>&1
```

上面命令所做的事情大致如下：
- 标准输出丢弃到`/dev/null`
- 标准错误输出重定向到标准输出
- 最终标准输出和错误输出一起被丢弃到`/dev/null`



当然如果觉得这种`2>&1`写法抽象和难以理解，也可以用下面的命令代替，这也是更为**通用**的写法：

```sh
grep -r hello /sys/ &> /dev/null
```

执行这一串命令之后我们难免会有疑问，输出都丢弃了，**如何判断命令是否执行成功？** 这里就要使用之前介绍的一个技巧，那就是`ehco $?`

```sh
echo $?
2 
```

在执行命令之后执行`ehco $?`，如果结果是0表示命令执行是正确的，如果类似值为 2则是该命令生成了错误结果。我们总是可以通过`$?` 验证命令是否执行正确。

# 其他案例

## dd和/dev/null

使用 dd，我们可以测试磁盘的顺序读取速度。当然这不是一个非常准确的测量方式，但是对于快速测试它非常有用。

```sh
dd if=<big_file> of=/dev/null status=progress bs=1M iflag=direct
```

> <big_file> 替换为某个比较大的文件的绝对路径

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230225075430.png)

## 测试下载速度

相对应的也可以用来测试下载速度。

```sh
wget -O /dev/null <big_file_link>
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230225213519.png)

# 总结

总是`/dev/null`是一个非常有用的空间，在很多开源组件的启动脚本中很容易见到这些命令的使用场景。
