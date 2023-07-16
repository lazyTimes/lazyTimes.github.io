---
title: Navicat如何复制数据库
subtitle: Navicat如何复制数据库一次实战笔记
url_suffix: navicatcopy
author: lazytime
tags:
  - 其他
categories:
  - 其他
keywords: navicat,数据库
description: Navicat如何复制数据库一次实战笔记
copyright: true
date: 2021-03-28 14:34:25
---

# Navicat如何复制数据库

## 前言：

​	这篇文章简单记录一下如何通过navicat备份一整个数据库留个记录，节省后面百度的时间。

​	注意，这里依照`postgres`数据库作为案例。

## 第一步：备份数据库

​	首先我们选择任意想要复制的数据库，选择备份按钮。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311155316.png)

​	接着，我们可以在常规里面进行基本的配置，，可以在对象选择里面进行基本的配置。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311155453.png)

​	点击备份之后，我们可以看到信息日志，navicat正在为我们备份整改数据库的内容。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311155534.png)

​	备份完成之后，在备份的列表里面，可以看到生成对应的日期时间戳生成的对应db文件。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311155721.png)

​	**关键的地方来了，到了这一步，说明整个库的数据和结构我们都备份好了，现在我们需要将其提取到sql里面，用做新的数据库导入**。右击需要导出sql文件的备份，点击`提取SQL`。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311161408.png)

​	接着我们选择`提取sql`，然后指定一个目录的位置，确定之后，navicat就开始将整个备份导出到sql。同样耐心等待信息日志。

> 这个过程可能会很长，请耐心等待

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311161528.png)



## 第二步：新建需要复制的数据

​	这个操作应该简单的不能再简单了，就是新建一个新的数据库。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311155906.png)

## 第三步：导入备份数据库

​	这一步也是非常简单的，右击选择数据库，，选择`运行sql文件`，找到刚刚提取出来的sql文件导入，然后耐心等待即可。

> 这个过程比较长，数据库大的话几分钟，甚至小时都是有可能的，这时候可以去干干别的事情，**进度条不动不一定是死机了，可能是数据过大没有加载进度条**

​	![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210311161722.png)

## 总结：

​	这次简单记录的一下如何快速的复制一个数据库，当然更快的方式是使用`pq_dump`(针对不同数据库有不同的备份命令)。速度要比navicat 快上好几倍。但是通常情况下我们连接数据库的服务器或者客户端都不在本地，这种方式备份和复制整个库是十分方便的。

​	同时养成良好的备份习惯有助于严重失误的时候进行回溯。

​	