---
title: 如何访问jar包下面的资源文件
subtitle: jar包的访问一定要知道内容访问机制
author: lazytime
tags:
  - jar
categories:
  - 技巧
keywords: 关于内嵌一些通用页面和功能的做法
description: 内嵌页面的做法
copyright: true
date: 2020-10-22 21:17:45
---

# 如何访问jar包下面的资源文件
## 前言

有些时候，我们需要通用一些功能页面，比如maven中公用的core的jar包内需要增加一些通用的功能，现在总结一下自己从中学到的一些东西

<!-- more -->

## 参考博客

https://blog.csdn.net/w302974215/article/details/49816385

https://my.oschina.net/zhaoyun1985/blog/479238?from=mail-notify

https://bbs.csdn.net/topics/380223539/

## 起步：

+ 创建一个springboot 项目
+ 在springboot resources 下面创建一个 META-INF 文件
+ 再创建一个resources 的内容，可以同步到reosurces下面

### 目录工程结构

+ resources
  + META-INF
    + resources
      + WEB-INF
      + js
    + jsp
    + jsp
    + jsp

当文件被打包为jar的之后

将会可以依照上下文进行访问

> 注意最终jar包是生成在 `lib`包下面


## Spring Boot 默认到访问规则
+ "classpath:/META-INF/resources/", 
+ "classpath:/resources/",
+ "classpath:/static/", 
+ "classpath:/public/" 

在静态资源文件夹中有几个文件名是默认的。
index：欢迎页
favicon.ico：顶端图标文件
只要将相关的文件放入目录中，我们输入路径后即可访问。