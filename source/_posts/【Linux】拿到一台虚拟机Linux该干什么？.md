---
title: 【Linux】拿到一台虚拟机Linux该干什么？
subtitle: 简单记录一波
author: 阿东
url_suffix: linuxshould
tags:
  - 服务端-linux
categories:
  - 服务端-linux
keywords: linux,xuniij
description: 拿到一台虚拟机Linux该干什么？
copyright: true
date: 2022-09-07 21:23:04
---
# 【Linux】拿到一台虚拟机Linux该干什么？

# 引言

很多时候我们喜欢在自己电脑上装一台Linux虚拟机玩，但是每次装好之后基本都是两眼无神，不知道下一步干啥，所以这篇文章主要就是解决安装好Linux之后，建议做的一些操作，帮助快速构建本地可用环境。

本文演示的Linux版本为CenterOs7.9，使用的镜像是官方7.9的Miniual版本（也就是最小体积版本），VM Tool 的版本为16.2.3 build-19376536。整个过程步骤十分简单，比较适合新手使用。

<!-- more -->

## 一、步骤

## 1.1 虚拟机连接

通过虚拟机登录到LInux，切换Root角色，使用`sudo vi /etc/ssh/sshd_config`修改文件，注意不要改错文件。

```shell
sudo vi /etc/ssh/sshd_config
# 修改端口
Port 10022
# 由于是新的Linux虚拟机，建议还是先保留22端口，等能正常用10022登陆再去掉，万一10022登不上就嗝屁了（并不会）
Port 22
# 不允许 ROOT 登陆，不成文规定
PermitRootLogin: no
```

## 1.2 禁止root登陆

执行命令`vi /etc/ssh/sshd_config`，找到`PermitRootLogin`，将后面的yes改为no。

记得把前面的注释 `#` 取消，这样root就不能远程登录了！通常用普通账号登录进去，要用到root的时候进入系统再使用命令`su root`。

也可以构建 **给予sudo权限的用户** 操作自己的虚拟机（下文介绍），总之就是不要Root直接登录。

`vi` 命令是没有颜色提示的，所以如果想要更好的配置阅读体验，通常需要安装`vim`，命令`yum install vim -y`。

再次强调拿到虚拟机之后第一手操作是**关闭Root登录**，不管是否为本地LInux服务。

```
# 不允许 ROOT 登陆
PermitRootLogin: no
```

修改完成之后的效果图：

![改端口禁ROOT](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220906164214.png)

> 如果找不到配置，检查是否修改的是**ssh_config**，而不是**sshd_config**文件！

 ## 1.3 改登录端口（ssh）

临时新增端口方法不建议使用，这里记录改文件永久生效的办法。

因为是自己本地的虚拟机，所以直接图省事，把防火墙关闭，这样也不要老是去配端口。

```shell
vi /etc/ssh/sshd_config
```

根据要求加上自己需要的端口，将`#Port 22`的注释去掉并且换行加入`Port 10022`，如果是不是增加而是想直接修改端口的话，这里**建议先保留22端口，等新端口可以登录再去掉**。

另外再介绍一下，443是https使用的端口，3128是squid使用的端口，一万以内的端口使用频率很高。

如果是自己使用建议使用大端口，比如10000~65535以上，基本不会有中间件和其他的冲突问题。

对于一些自己程序使用的端口，也是数建议千位数字前面加一个1,，基本可以保证不冲突。

修改完成之后一定要记得 **重启ssh服务**：`systemctl restart sshd.service`，或者直接重启虚拟机Linux系统。


## 1.4 关闭linux内部防火墙

**检测自己添加了多少开放的端口**  ：`firewall-cmd --zone=public --list-ports`。当然我这里演示是直接关掉防火墙，对外是畅通无阻的。

临时新增方法（不建议使用）：`firewall-cmd --zone=public --add-port=12280/tcp --permanent` ，`--permanent`就是让端口永久生效

不建议使用原因，第一个是不知道端口加来干嘛用的，后面容易忘，第二个是这个操作只能**临时生效**，重新启动又会还原。

> 不建议使用的其他原因是执行此命令**会把文件的所有注释清空**！！

注意改端口之后尝试外部连接是失效的，因为还有selinux和防火墙需要处理，这里依然图省事一并给他关了。

所以这里firewalld 的基本使用如下：

     启动：`systemctl start firewalld`
     关闭： `systemctl stop firewalld`
     查看状态：`systemctl status firewalld` 
     开机禁用  ： `systemctl disable firewalld`
     开机启用  ： `systemctl enable firewalld`

我们需要关闭防火墙，当然这里只能在自己的虚拟机这么用，主要是减少自己捣鼓学习的时候避免各种不必要的麻烦，在真实的生产环境实际上更多情况是开启的。

此外如果是云服务器提供商，这个配置通常也是关闭的，取而代之的是在外部做了一个安全网。

**步骤**

使用`systemctl status firewalld`检查状态。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220523114235.png)

关闭`systemctl status firewalld.serivce`。

> 这里有个挺蛋疼的踩坑点，感觉这块像是两个人写的（怪怪的），使用`systemctl stop firewalld`是临时关闭，重启之后防火墙又会自动打开，`systemctl status firewalld.serivce`是**永久关闭防火墙服务**。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220906162311.png)


## 1.5 关闭 SeLinux

**简易解释**

关闭防火墙之后，接着是关闭 **SeLinux** 。

```shell
# 注意需要ROOT 权限
 vi /etc/selinux/config
#　将 SELINUX=disabled 表示关闭
SELINUX=disabled
```

操作完成之后使用 `:` 加上`-x` 保存，之后建议`reboot`一下。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220523112457.png)

>  **x 和 wq 的区别？**
       x 执行操作完毕，如果修改了文件，文件的最后修改时间会产生变化，没有，则不变化
       wq 执行操作完毕，不管文件有没有改动，最后修改时间都会产生变化

**保姆解释**

**永久生效**的方法，执行命令 `vi /etc/selinux/config` 【需要**ROOT**权限】，出现如下文本。

```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

默认情况下SELINUX是`enforcing`的，我们只需要修改这一部分：

```
SELINUX=disabled
```

最后`reboot`一下。

## 1.6 Sudo权限用户构建

使用Root添加新用户，配置密码：

```shell
# 添加新用户
useradd 想要添加的用户名（英文）

# 修改密码
passwd 想要添加的用户名（英文）
# 之后提示输入密码
```

让新用户具备SUDO的权限，`vi /etc/sudoers`，或者给这个文件赋予写入的权限` chmod u+w /etc/sudoers`（直接Root操作更方便）。

```shell
## Next comes the main part: which users can run what software on 
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
zxd     ALL=(ALL)       ALL

```

后续使用者加上sudo 命令， 就可以具有root权限了，但是会提示输入密码。

这样的用户既可以外部登录，操作的时候也相对安全一些，虽然有Sudo权限，但是实际上要比Root管的范围要小一点，Root依然是Linux的最高权限管理员。

**另一种方法：错误示范**

下面的方式通用是给普通用户穿一件Root的衣服，但是这样的话登录会被识别为Root登录，**和禁用Root发生冲突**，直白的说就是登不进系统（尴尬）。

> 不建议从用户管理的角度做这种操作，因为本质上相当于复制了一个Root。

创建一个新用户，但是给予root同等的权限，我们称之为伪Root的普通用户，换句话说叫做系统管理员。

我们需要执行下面的步骤：

```shell
# 添加新用户
useradd 想要添加的用户名（英文）

# 修改密码
passwd 想要添加的用户名（英文）
# 之后提示输入密码
```

接下来是配置用户的Root权限，这里要使用Root身份进行操作：

```shell
vim /etc/passwd
```

第一行是root身份，所以我们直接把相关配置赋给新用户。

```
#第一行内容
root:x:0:0:root:/root:/bin/bash
# 新增用户（通常新增用户的最底部）
zxd:x:1000:1000:zxd:/home/zxd:/bin/bash

# 进行修改操作
# 修改之前（通常新增用户的最底部）
zxd:x:1000:1000:zxd:/home/zxd:/bin/bash

# 修改之后
# 删除掉
# zxd:x:1000:1000:zxd:/home/zxd:/bin/bash
# 新增下面这一行
新增用户名:x:0:0:root:/root:/bin/bash
```

最后验证一下，如果`su 新建用户名`之后前面显示的内容为Root则说明伪装成功：

```
[root@localhost zxd]# su zxd
```

## 1.7 验证

这里直接使用**Xshell**进行测试，使用10022登录，22端口无法登录，无法用root登录等均验证通过。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220907101420.png)


# 二、扩展思考

弄完基础配置的Linux系统之后，我们可以从下面的方面入手，当然后半部分基本为扩展学习建议：

- 关闭 selinux 。
- SSH 改端口（ssh），1024以内的端口需要root启动的。比较常见的端口一般禁止占用：
	- 0 - 65535 ；
	- ssl 443 ；
	- 22 ；
	- 8080 ；
	- 80 ；
	- 1433 ；
	- 3306； 
	- 10022（改完之后的登录端口） ；
	- 5022；
	- 禁止root登陆； 
- 新建用户和组，给予目录的权限。
- 开放linux内部防火墙（iptables 6（不维护）、firewalld 7） 。
- 挂载硬盘 。
- NFS文件共享 。
- 局域网拷贝 。
- 文件自动同步 。
- 检测服务器磁盘空间 。
- shell 自动清理磁盘 。
- 构建软链接，硬链接 （windows,linux）。
- yum本地源。
- 挂载光驱 （【无法联网的情况，学会可以使用本地源】）
-  时间设置，时间同步，修改时区。 
- 常用的命令熟悉。
- 常用软件安装。
- 根据所学解决甲方安全测评等单位，给出的服务器安全整改报告，或者自己设置一些难度比较高的挑战目标（如果有可能的话）。

# 三、小结

轻松简单的文章，希望这篇文章对于读者有帮助。

