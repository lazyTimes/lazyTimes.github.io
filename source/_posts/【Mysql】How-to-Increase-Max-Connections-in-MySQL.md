---
title: 【Mysql】How to Increase Max Connections in MySQL
subtitle: 如何扩大Max Connections
author: lazytime
url_suffix: mysqlconnection
tags:
  - Mysql
categories:
  - Mysql
keywords: maxconnection
description: 最大连接数
copyright: true
date: 2023-06-15 10:07:14
---

# 原文

[https://ubiq.co/database-blog/how-to-increase-max-connections-in-mysql/](https://ubiq.co/database-blog/how-to-increase-max-connections-in-mysql/)

# 引言

有时候连接Mysql时候我们会遇到“Too many connections”这样的报错，当然更多可能是在生产过程中碰到这样的问题，默认情况下Mysql的整体服务器连接数设置过低。

> Sometimes MySQL server may give “Too many connections” error message when you try to connect to a MySQL database. Here’s how to increase max connections in MySQL to fix this problem.

<!-- more -->

# Too many connections 的影响

这意味着所有可用的连接已经被各种客户端使用，你的MySQL服务器不能打开任何新的连接，直到任何现有的连接被关闭。

> By default, MySQL 5.5+ can handle up to 151 connections. This number is stored in server variable called _max_connections_. You can update max_connections variable to increase maximum supported connections in MySQL, provided your server has enough RAM to support the increased connections.

# max_connections默认值

> By default, MySQL 5.5+ can handle up to 151 connections. This number is stored in server variable called _max_connections_. You can update max_connections variable to increase maximum supported connections in MySQL, provided your server has enough RAM to support the increased connections.

在Mysql5.5+的版本中，这个值只有151，我们可以通过`show variables like "max_connections";`查看自己的Mysql服务器最大连接数。根据当前的机器配置和实际情况，我们可以拉大这个参数。

```mysql
mysql> show variables like "max_connections";
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151 |
+-----------------+-------+
1 row in set (0.01 sec)

```

# 如何扩大max_connections这个值？（docker 中的Mysql为例）

下面是基础步骤。

## 0. 如何查看？

首先我们需要了解如何查看当前的最大连接数。

```mysql
show variables like "max_connections";
```

navicat 可以在**命令行模式**下查看查看连接数（这里为个人实验修改后结果）：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230416195103.png)

有两种方法来增加MySQL连接。下面是通过MySQL命令行工具更新最大连接数为200的命令，无需重新启动。

## 1. 临时修改：命令

通过Mysql连接到客户端，可以使用下面的命令修改最大连接数：

```mysql
mysql> set global max_connections = 200;
```

但是需要注意这种修改方式**一旦重启就会失效**，所以这个命令通常是在出现类似的生产事故的时候救急使用。

如果想要永久生效，需要修改mysql的相关配置文件。

## 2. 永久生效：系统配置修改

要永久生效需要修改系统配置，

```sh
sudo vi /etc/my.cnf
```

需要注意根据Linux发行版和安装类型不同，my.cnf文件可能位于以下任何位置。如果是docker，则my.cnf 位于自定义映射的位置

-   /etc/my.cnf
-   /etc/mysql/my.cnf
-   $MYSQL_HOME/my.cnf
-   ~/.my.cnf


注意这个参数要配置到 **[mysqld]** 下面。

```sh
[mysqld]
max_connections = 10024
```

最后只需要重启Mysql服务即可。

# 重启Mysql

个人搭建的Docker只需要重启镜像即可，较为方便。

```sh
[root@localhost conf]# docker restart 58f
```

> docker 的 ID 支持模糊匹配。也可以自定义name之后进行重启。

# 写在最后

希望上述教程能帮助你了解如何增加MySQL的最大连接数。

> Hopefully, the above tutorial will help you increase max connections in MySQL.