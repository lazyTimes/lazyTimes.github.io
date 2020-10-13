---
title: 一篇帖子解决window.open中使用jquery操作父元素的问题
subtitle: 还是不喜欢搞前端
author: lazytime
tags:
  - 前端
categories:
  - 未分类
keywords: jquery
description: 一篇帖子解决window.open中使用jquery操作父元素的问题
copyright: true
date: 2020-10-11 14:30:52
---

首先看下例子：
http://www.cssrain.cn/demo/1/Window-open-Show/index.html

这是最近在项目中用到的，后来还在父窗口，增加了删除，然后前面的序号重新排列。

其实都还是比较简单。

具体遇到了哪些问题呢，总结下：

1，因为父窗口中引入了jquery包了，所以子窗口不需要引入了（注：在我这个例子中）。

2，window.opener.$("#showtable tr") 后面直接写 jquery的选择 器。

3，删除排序也很简单，就是删除一个后，重新获取所有tr，然后在each循环设置一下index

本篇文章来源于 cssrain.cn 原文链接：http://www.cssrain.cn/article.asp?id=869