---
title: 【Mysql】How to store UTF8 characters in MySQL
subtitle: 如何修改默认编码
author: lazytime
url_suffix: mysqlcharacter
tags:
  - Mysql
categories:
  - 数据库-mysql
keywords: 请输入关键字（英文逗号分隔多个关键字）
description: 请输入描述信息
copyright: true
date: 2023-06-15 10:59:40
---

# 原文

[How to store UTF8 characters in MySQL - Ubiq BI](https://ubiq.co/database-blog/how-to-store-utf8-characters-in-mysql/)

# 引言

关于 **《如何在MySQL中存储UTF8字符》** 的一篇英文博客阅读和个人实战笔记。注意个人实验过程中使用了docker的Mysql 5.7 版本，读者可以根据自身情况调整。

<!-- more -->

# 1. Shell 检查字符集

如果不习惯Linux的小黑框，最简单的方式是用navicat的命令行工具查询，注意用户需要具备对应的系统配置权限，建议用root操作：

首先我们右击navicat的数据库连接配置（已准备好），选择“命令列界面”：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230416175305.png)

之后是navicat仿照的Shell，等待用户输入：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230416175340.png)

运行下面的命令可以查看所有字符集配置：

```mysql
SHOW VARIABLES LIKE 'character_set%';
```

个人的实验结果如下：

```mysql
mysql> SHOW VARIABLES LIKE 'character_set%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.03 sec)
```

可以看到字符集并不统一。

# 2. 修改字符集为UTF-8

个人使用为docker的mysql自定义映射的配置文件位置，和原始的安装方式配置文件配置略有不同。不过基本的操作方式都是修改`my.cnf`的配置：

```sh
[client] 
default-character-set=utf8mb4 
[mysqld] 
character-set-server = utf8mb4
```

个人操作记录如下，定位到映射文件位置之后，直接`vim`修改配置文件：

```sh
[root@localhost conf]# vim /opt/mysql/conf/mysql.cnf 
```

在配置文件当中做如下改动：

```sh
[root@localhost conf]# cat /opt/mysql/conf/mysql.cnf 
# mysqld custom
[mysqld]
# 下面部分可以忽略
sort_buffer_size = 16M
read_rnd_buffer_size = 2M
max_connections = 10024

# set character to utf-8
character-set-server = utf8mb4

[client]
# 注意客户端连接也要一并设置
default-character-set=utf8mb4

```

修改之后重启docker的mysql镜像。

```sh
[root@localhost conf]# docker ps 
CONTAINER ID   IMAGE       COMMAND                  CREATED       STATUS       PORTS                                                    NAMES
58fa52f21404   mysql:5.7   "docker-entrypoint.s…"   3 hours ago   Up 2 hours   33060/tcp, 0.0.0.0:13306->3306/tcp, :::13306->3306/tcp   mysql57

```

重启之后万事大吉。

```sh
[root@localhost conf]# docker restart 58f
```

之后依旧是连接mysql数据库，继续使用`show variables like 'character%';`命令查看当前字符集。

修改之前：

```mysql
mysql> SHOW VARIABLES LIKE 'character_set%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

修改之后结果如下：

```mysql
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+

```

# 3. 数据表转换为UTF-8

运行下面的命令，将你的数据库的字符集和排序改为UTF8。用你的数据库名称替换下面的dbname。

```mysql
ALTER DATABASE dbname CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`
```

> dbname 修改为自己的数据库即可

上述命令将把你的数据库中的所有表转换为UTF8MB4 格式。
