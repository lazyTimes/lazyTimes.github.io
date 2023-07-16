---
title: Springboot 打包的一些学习
subtitle: Spring Boot 打包成的可执行 jar 为什么不能被其他项目依赖？
url_suffix: springbootpac
author: lazytime
tags:
  - SpringBoot
categories:
  - SpringBoot
keywords: Springboot打包
description: Springboot打包
copyright: true
date: 2020-07-26 14:23:01
---

https://blog.csdn.net/u012702547/article/details/95180256

https://blog.csdn.net/weixin_38187317/article/details/82688906

制作七牛-spring-boot-starter并上传中央仓库 https://blog.csdn.net/weixin_38187317/article/details/82723758

> 说明： 依照七牛云自己实现了一个springboot，已上传中央仓库 来源博客： https://blog.csdn.net/weixin_38187317/article/details/82688906 目的：学习一下如何springboot 引入另一个springboot 的service层，

# Spring Boot 打包成的可执行 jar ，为什么不能被其他项目依赖？

https://blog.csdn.net/u012702547/article/details/95180256

> 说明：这里解读了为什么不能使用springboot 的插件打包

# SpringBoot 如何手动引入本地的jar包 并利用maven成功打包

- 参考博客 https://www.javatt.com/p/84969
- 需要执行如下命令： `mvn install:install-file -Dfile=./qiniu-spring-boot-starter-0.2-RELEASE.jar -DgroupId=com.zxd -DartifactId=myquartz -Dversion=8 -Dpackaging=jar`
- Maven将自己的jar包引入本地库中 https://www.jianshu.com/p/cef1bc65584d

# Spring Boot 制作一个自己的 Starter

https://blog.csdn.net/wo18237095579/article/details/81197245#重点编写-autoconfigure-类

# Springboot 打Jar包，Maven完美解决本地Jar包自动打入Springboot Jar包中

https://www.sojson.com/blog/253.html

# Maven将自己的jar包引入本地库中

https://www.jianshu.com/p/cef1bc65584d

\#　制作SpringBoot的jar给其他项目使用需要注意的点 https://blog.csdn.net/weixin_38187317/article/details/82688906