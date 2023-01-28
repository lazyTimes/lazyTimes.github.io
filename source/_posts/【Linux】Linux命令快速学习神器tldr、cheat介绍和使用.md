---
title: 【Linux】Linux命令快速学习神器tldr、cheat介绍和使用
subtitle: 真的很方便
url_suffix: tldrcheat
author: lazytime
tags:
  - Linux
categories:
  - Linux
keywords: tldr,cheat
description: Linux命令行学习和使用
copyright: true
date: 2023-01-28 11:42:10
---

# 引言

本文介绍tldr和Cheat等实用工具的安装和使用，这些工具虽然本身不能替代`man`、`info`等命令，但是在很多时候想要快速学习和掌握命令但是忘记常见用法非常有帮助。

> 个人看法：对于非运维人员简直是神器。

**tldr**：全称 too long, Don’t read，翻译成中文就是太长不想阅读，比–help或者man这些传统手册更便捷、更便于使用。
**cheat**：作弊。

这两个命令有什么用？这里简单举个例子就知道了：

```sh
ubuntu@VM-8-8-ubuntu:~$ sudo tldr ls

  ls

  List directory contents.
  More information: https://www.gnu.org/software/coreutils/ls.

  - List files one per line:
    ls -1

  - List all files, including hidden files:
    ls -a

  - List all files, with trailing / added to directory names:
    ls -F

  - Long format list (permissions, ownership, size, and modification date) of all files:
    ls -la

  - Long format list with size displayed using human-readable units (KiB, MiB, GiB):
    ls -lh

  - Long format list sorted by size (descending):
    ls -lS

  - Long format list of all files, sorted by modification date (oldest first):
    ls -ltr

  - Only list directories:
    ls -d */

```

# 内容概览

1. Ubuntu和CenterOs介绍和安装**tldr**命令。
  1. CenterOs和Ubuntu的安装方式使用
  2. Ubuntu的常见问题和解决方案。
2. Ubuntu和CenterOs介绍和安装**cheat**命令。
  1. 安装验证和使用
  2. Ubuntu和CenterOs处理方式一致
3. 类似项目列举。

<!-- more -->


# 官方资料

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230125221920.png)



在线wiki搜索：[tldr | simplified, community driven man pages (ostera.io)](https://tldr.ostera.io/)
（注意国内访问比较慢，需要自带魔法）

官方的安装介绍页面地址：[https://tldr.sh/#installation](https://tldr.sh/#installation)

tldr项目地址：[https://github.com/tldr-pages/tldr](https://github.com/tldr-pages/tldr)

# tldr

## Ubuntu安装tldr

### 安装

Ubuntu 安装比较简单，直接使用`sudo apt-get install tldr`安装。

```sh
sudo apt-get install tldr
```

如果上面的命令安装之后依然无法使用，可以按照下面的命令进行安装：

```node
sudo apt-get install npm
```

```node
sudo apt-get install nodejs-legacy
```

```node
sudo npm install n -g
```

注意这里有两种方式，一种是使用latest版本，另外一种是使用stable版本，个人建议使用stable版本，如果Ubuntu的系统版本比较低，则需要降低node版本。

```node
sudo n stable
（或 sudo n latest）
```

```node
sudo npm install -g tldr
```

当然也可以不使用nodeJS，也可以使用`pip`

```sh
pip3 install tldr
```

如果是Mac系统就十分简单了：

```sh
brew install tldr
```


### 验证

在ubuntu当中验证直接使用：

```
ubuntu@VM-8-8-ubuntu:~$ tldr
tldr - Simplified and community-driven man pages

Usage: tldr [-v|--version] 
            ((-u|--update) | [-p|--platform PLATFORM] COMMAND | (-a|--about))
  tldr Client program

Available options:
  -h,--help                Show this help text
  -v,--version             Show version
  -u,--update              Update offline cache of tldr pages
  -p,--platform PLATFORM   Prioritize specfic platform while searching. Valid
                           values include linux, osx, windows, sunos
  COMMAND                  name of the command
  -a,--about               About this program
```

## Ubuntu安装常见问题

### No tldr entry for xxx

Ubuntu安装tldr使用`tldr ls`之后，很有可能出现类似`No tldr entry for ls`的命令，出现这种情况可能有下面两种情况：

- 首次安装需要更新tldr的“数据库”。
- 当天用户安装使用`sudo`，tldr的数据库没法访问。

```sh
ubuntu@VM-8-8-ubuntu:~$ tldr ls
No tldr entry for ls
```

更新tldr的数据库：

```sh
ubuntu@VM-8-8-ubuntu:~$ sudo tldr --update
✔ Updating...
✔ Creating index...
```

当然也可以使用`tldr -u`：

```sh
ubuntu@VM-8-8-ubuntu:~$ sudo tldr --u
```

**sudo安装使用**

个人的云服务器ubuntu使用了`sudo`安装之后，需要使用`sudo tldr`才可以正常使用，因为日常登录的用户为`ubuntu`用户，安装的过程全部使用`sudo`，查询某个命令也需要使用root身份进行查询。

如果出现`No tldr entry for xxx`，有可能是你用的`sudo`安装但是当前的用户却没有`sudo`的权限。

```sh
ubuntu@VM-8-8-ubuntu:~$ sudo tldr ls

  ls

  List directory contents.
  More information: https://www.gnu.org/software/coreutils/ls.

  - List files one per line:
    ls -1

  - List all files, including hidden files:
    ls -a

  - List all files, with trailing / added to directory names:
    ls -F

  - Long format list (permissions, ownership, size, and modification date) of all files:
    ls -la

  - Long format list with size displayed using human-readable units (KiB, MiB, GiB):
    ls -lh

  - Long format list sorted by size (descending):
    ls -lS

  - Long format list of all files, sorted by modification date (oldest first):
    ls -ltr

  - Only list directories:
    ls -d */

```

### nodeJs版本降级

版本降级相关资料参考自：[https://blog.csdn.net/Fabulous1111/article/details/84983869](https://blog.csdn.net/Fabulous1111/article/details/84983869)

#### （1）安装node版本管理模块n

```sh
	sudo npm install n -g
```

下边步骤请根据自己需要选择

#### （2）安装稳定版

```sh
	sudo n stable
```

#### （3）安装最新版

```sh
	sudo n latest
```

#### （4） 版本降级/升级

```sh
	sudo n 版本号
```

#### （5）检测目前安装了哪些版本的node

```
n
```

提示内容如下：

```
  ο node/16.15.1
    node/18.13.0
    node/19.5.0

Use up/down arrow keys to select a version, return key to install, d to delete, q to quit
```


#### 切换版本（不会删除已经安装的其他版本）

```
n 版本号
```

比如：`n 16.15.1`

#### （7）删除版本

```
sudo n rm 版本号
```

#### （8）直接移除Nodejs

PS：注意不同的操作系统命令会有差别，这里为Ubuntu的卸载方式

```sh
sudo apt-get remove nodejs
```

#### 简单使用

```sh
sudo npm install -g n

sudo n install 16.15.1 # 太新的也会有问题
```

以上就是ubuntu常见问题和处理。

## CenterOs 安装 tldr

CenterOs的安装方式和Ubuntu类似，这里展示安装Node环境之后安装tldr并使用的过程。

### 安装

1. 安装NodeJs，如果嫌麻烦可以直接安装`sudo yum install -y npm`。

[[【Linux】NodeJs 安装和环境变量配置]]

2. 我们使用官方提供的命令安装。

```node
sudo npm install -g tldr
```

3. 如果是使用NodeJs环境变量设置的方式安装，需要设置软链接，tldr命令默认会安装到`nodeJs`安装路径的**Bin**目录下面，如果不好理解，可以参考下面的软链接构建命令。

> 还有一种方式是在Path中设置`/xx/nodejs/bin` 为环境变量

```sh
[zxd@localhost ~]$ sudo ln -s /opt/nodeJs/bin/tldr /usr/local/bin
[sudo] password for zxd: 

```

> 如果软链接路径构建错误，可以使用`sudo ln -fs /opt/nodeJs/bin/tldr /usr/local/bin`加入 `-f`参数强制覆盖之前的软链接。

### 验证

因为构建了`tldr`软链接，我们可以再任意路径使用这个命令。如果敲入`tldr`命令出现下面的提示证明安装成功：

```sh
[zxd@localhost bin]$ tldr
Usage: tldr command [options]

Simplified and community-driven man pages

Options:
  -v, --version            Display version
  -l, --list               List all commands for the chosen platform in the cache
  -a, --list-all           List all commands in the cache
  -1, --single-column      List single command per line (use with options -l or -a)
  -r, --random             Show a random command
  -e, --random-example     Show a random example
  -f, --render [file]      Render a specific markdown [file]
  -m, --markdown           Output in markdown format
  -o, --os [type]          Override the operating system [linux, osx, sunos, windows]
  --linux                  Override the operating system with Linux
  --osx                    Override the operating system with OSX
  --sunos                  Override the operating system with SunOS
  --windows                Override the operating system with Windows
  -t, --theme [theme]      Color theme (simple, base16, ocean)
  -s, --search [keywords]  Search pages using keywords
  -u, --update             Update the local cache
  -c, --clear-cache        Clear the local cache
  -h, --help               Show this help message

  Examples:

    $ tldr tar
    $ tldr du --os=linux
    $ tldr --search "create symbolic link to file"
    $ tldr --list
    $ tldr --list-all
    $ tldr --random
    $ tldr --random-example

  To control the cache:

    $ tldr --update
    $ tldr --clear-cache

  To render a local file (for testing):

    $ tldr --render /path/to/file.md

```

我们使用`tldr ls`查看`ls`命令的用法，确实赏心悦目。

```sh
[zxd@localhost ~]$ tldr ls
✔ Page not found. Updating cache...
⠴ Creating index...
✔ Creating index...

  ls

  List directory contents.
  More information: https://www.gnu.org/software/coreutils/ls.

  - List files one per line:
    ls -1

  - List all files, including hidden files:
    ls -a

  - List all files, with trailing / added to directory names:
    ls -F

  - Long format list (permissions, ownership, size, and modification date) of all files:
    ls -la

  - Long format list with size displayed using human-readable units (KiB, MiB, GiB):
    ls -lh

  - Long format list sorted by size (descending):
    ls -lS

  - Long format list of all files, sorted by modification date (oldest first):
    ls -ltr

  - Only list directories:
    ls -d */
```


# Cheat

## 介绍

这个漫画是Cheat恶搞man命令查一个命令需要翻几本书的时间，挺有意思的。

> cheat：有欺骗的意思，可以直接理解为**舞弊**或者**作弊**。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124231214.png)


## 官方资料

github项目地址：
[cheat/cheat: cheat allows you to create and view interactive cheatsheets on the command-line. It was designed to help remind *nix system administrators of options for commands that they use frequently, but not frequently enough to remember. (github.com)](https://github.com/cheat/cheat)

官方安装教程：[cheat/INSTALLING.md at master · cheat/cheat (github.com)](https://github.com/cheat/cheat/blob/master/INSTALLING.md)

## 手动安装

### 类Unix

类Unix系统可以使用下面一串命令解决。

```sh
cd /tmp \
  && wget https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz \
  && gunzip cheat-linux-amd64.gz \
  && chmod +x cheat-linux-amd64 \
  && sudo mv cheat-linux-amd64 /usr/local/bin/cheat
```

个人的CenterOs实验如下：

```sh
[zxd@localhost ~]$ cd /tmp \
>   && wget https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz \
>   && gunzip cheat-linux-amd64.gz \
>   && chmod +x cheat-linux-amd64 \
>   && sudo mv cheat-linux-amd64 /usr/local/bin/cheat
--2023-01-28 18:34:35--  https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz
Resolving github.com (github.com)... 20.205.243.166
Connecting to github.com (github.com)|20.205.243.166|:443... connected.
Unable to establish SSL connection.
[zxd@localhost tmp]$ cd /tmp   && wget https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz   && gunzip cheat-linux-amd64.gz   && chmod +x cheat-linux-amd64   && sudo mv cheat-linux-amd64 /usr/local/bin/cheat
--2023-01-28 18:34:59--  https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz
Resolving github.com (github.com)... 
... 省略一些安装提示信息

... github访问比较慢，耐心等待
100%[=============================================================================>] 4,696,443   9.87KB/s   in 6m 11s 

2023-01-28 18:41:13 (12.4 KB/s) - ‘cheat-linux-amd64.gz’ saved [4696443/4696443]

```

个人的Ubuntu云服务器的实验如下：

```sh
ubuntu@VM-8-8-ubuntu:~$ cd /tmp \
  && wget https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz \
  && gunzip cheat-linux-amd64.gz \
  && chmod +x cheat-linux-amd64 \
  && sudo mv cheat-linux-amd64 /usr/local/bin/cheat
--2023-01-28 10:56:00--  https://github.com/cheat/cheat/releases/download/4.4.0/cheat-linux-amd64.gz
... 省略信息
Saving to: ‘cheat-linux-amd64.gz’

cheat-linux-amd64.gz              100%[==========================================================>]   4.48M   871KB/s    in 1m 42s  

2023-01-28 10:57:43 (44.9 KB/s) - ‘cheat-linux-amd64.gz’ saved [4696443/4696443]

ubuntu@VM-8-8-ubuntu:/tmp$ cheat
A config file was not found. Would you like to create one now? [Y/n]: y
Would you like to download the community cheatsheets? [Y/n]: y
Cloning community cheatsheets to /home/ubuntu/.config/cheat/cheatsheets/community.
Enumerating objects: 335, done.
Counting objects: 100% (335/335), done.
Compressing objects: 100% (314/314), done.
Total 335 (delta 36), reused 274 (delta 19), pack-reused 0
Cloning personal cheatsheets to /home/ubuntu/.config/cheat/cheatsheets/personal.
Created config file: /home/ubuntu/.config/cheat/conf.yml
Please read this file for advanced configuration information.
```

> 注意：这里可能需要更改版本号（“4.4.0”）和存档（“cheat-linux-amd64.gz”），具体取决于安装平台。

可以阅读 [releases page](https://github.com/cheat/cheat/releases) 了解当前命令支持的平台。

### 通过 `go install`安装

如果有GO 1.17 以上的版本，可以通过`go install`安装cheat。

```go
go install github.com/cheat/cheat/cmd/cheat@latest
```

### 其他安装方式

下面是官方介绍的其他安装方式。

| Package manager | Package(s)                                                   |
| --------------- | ------------------------------------------------------------ |
| aur             | [cheat](https://aur.archlinux.org/packages/cheat), [cheat-bin](https://aur.archlinux.org/packages/cheat-bin) |
| brew            | [cheat](https://formulae.brew.sh/formula/cheat)              |
| docker          | [docker-cheat](https://github.com/bannmann/docker-cheat)     |
| nix             | [nixos.cheat](https://search.nixos.org/packages?channel=unstable&show=cheat&from=0&size=50&sort=relevance&type=packages&query=cheat) |
| snap            | [cheat](https://snapcraft.io/cheat)                          |



## 基础使用

和tldr类似，第一次使用cheat也需要构建“数据库”，但是cheat比tldr的使用体验更好，我们只需要按照提示输入两次`y`确认即可：

```sh
[zxd@localhost tmp]$ cheat 
A config file was not found. Would you like to create one now? [Y/n]: y
Would you like to download the community cheatsheets? [Y/n]: y
Cloning community cheatsheets to /home/zxd/.config/cheat/cheatsheets/community.
Enumerating objects: 335, done.
Counting objects: 100% (335/335), done.
Compressing objects: 100% (314/314), done.
Total 335 (delta 36), reused 274 (delta 19), pack-reused 0
Cloning personal cheatsheets to /home/zxd/.config/cheat/cheatsheets/personal.
Created config file: /home/zxd/.config/cheat/conf.yml
Please read this file for advanced configuration information.
```

最后是直接使用，个人感觉要比tldr麻烦事情少很多：

> 和 tldr的区别是排版的方式有点不同

```sh
[zxd@localhost tmp]$ cheat ls
# To display everything in <dir>, excluding hidden files:
ls <dir>

# To display everything in <dir>, including hidden files:
ls -a <dir>

# To display all files, along with the size (with unit suffixes) and timestamp:
ls -lh <dir>

# To display files, sorted by size:
ls -S <dir>

# To display directories only:
ls -d */ <dir>

# To display directories only, include hidden:
ls -d .*/ */ <dir>

# To display all files sorted by changed date, most recent first:
ls -ltc 

# To display files sorted by create time:
ls -lt

# To display files in a single column:
ls -1

# To show ACLs (MacOS):
# see also `cheat chmod` for `/bin/chmod` options for ACLs
/bin/ls -le

# To show all the subtree files (Recursive Mode):
ls -R

```

Uuntu的使用类似：

```sh
# 基础使用
ubuntu@VM-8-8-ubuntu:/tmp$ cheat cp
# To copy a file:
cp ~/Desktop/foo.txt ~/Downloads/foo.txt

# To copy a directory:
cp -r ~/Desktop/cruise_pics/ ~/Pictures/

# To create a copy but ask to overwrite if the destination file already exists:
cp -i ~/Desktop/foo.txt ~/Documents/foo.txt

# To create a backup file with date:
cp foo.txt{,."$(date +%Y%m%d-%H%M%S)"}

# To copy a symlink that points to a directory (and is thus soft) and not
# 'expand' the symlink (aka, preserve its nature as a symlink):
# Note this does NOT work (note trailing '/'):  cp -P /path/to/symlink-dir/
cp -P <symlink-dir> <dest-dir>

# To copy sparsely:
cp --sparse=always <src> <dest>
```

# 类似项目

还有更多和cheat以及tldr类似的项目，这里就不过多介绍了。

[**Cheat**](https://github.com/cheat/cheat) 允许您在命令行上创建和查看交互式备忘单。它旨在帮助提醒 Linux 系统管理员他们经常使用但不够频繁而无法记住的命令的选项。

[**cheat.sh**](https://cheat.sh/) 将来自多个来源（包括 tldr-pages）的备忘单聚合到 1 个统一界面中。

[**devhints Rico**](https://devhints.io/) 的备忘单不仅仅关注命令行，还包括大量与编程相关的其他备忘单。

[**eg**](https://github.com/srsudar/eg) 在命令行上提供了详细的示例和解释。示例来自存储库，但例如支持显示自定义示例和命令以及默认值。

[**kb**](https://github.com/gnebbia/kb) 是一个极简的命令行知识库管理器。 kb 可用于以极简主义和干净的方式组织您的笔记和备忘单。它还支持非文本文档。

[**navi**](https://github.com/denisidoro/navi) 是一个交互式备忘单工具，它允许您即时浏览特定示例或完整命令。

[bropages（已弃用）](http://bropages.org/)是对手册页的高度可读性补充。它显示了 Unix 命令的简明、常见示例。这些示例由用户群提交，可以投票赞成或反对；最好的条目是人们在查找命令时最先看到的内容。


# 写在最后

整体体验下来个人比较偏向cheat一点，类Unix系统安装官方的教程一个名称就可以完成，而tldr需要Node环境，同时因为Node更新速度就像喝汤容易导致Linux系统版本或者内核版本低而不支持的问题。
