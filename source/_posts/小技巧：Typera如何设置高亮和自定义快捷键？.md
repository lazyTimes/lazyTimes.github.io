---
title: 小技巧：Typera如何设置高亮和自定义快捷键？
subtitle: Markdown 如何给文字加上高亮效果？
author: lazytime
url_suffix: note8
tags:
  - 其他
categories:
  - 其他
keywords: Markdown
description: Markdown 如何给文字加上高亮效果？
copyright: true
date: 2020-07-26 19:59:19
---

# Markdown 如何给文字加上高亮效果？

## 前置工作：

1. 最新版本的typera
2. NotePad++ （包括 sublime、atom 都可，但是不建议使用记事本，后面会说明原因）

<!-- more -->

## 其他：

其实官方目前已经有了案例（包括mac os X系统）

这里分享一下地址：如下

http://support.typora.io/Shortcut-Keys/#change-shortcut-keys

如果看不懂英文可以下载谷歌浏览器翻译查看

## 开始：

### 第一步：随便打开一个.md后缀的文件

我们可以打开typera的软件或者打开一个md 文件用typera 打开方式即可

### 第二步：选择文件,打开偏好设置

在打开的偏好设置里面，我们需要勾选**高亮**选项

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200519224209.png?ynotemdtimestamp=1595748679079)

### 第三步：点击下方的高级设置，打开一个名为`conf.user.json`的文件

下面我截取了部分配置，只要参考设置快捷键即可

```
// Custom key binding, which will override the default ones.
  "keyBinding": {
    // for example: 
    //"Always on Top": "Ctrl+Shift+P",
    // 设置高亮快捷键
	"Highlight": "Ctrl+Shift+H"
  },
```

### 第四步：注意：需要重新启动typera才能生效

我们按下Ctrl+Shift+H就可以实现文字高亮了，是不是特别方便

重启typera之后，我们可以体验高亮功能了，高亮功能还是十分有用的，不仅在文字里面高亮，在目录里面也有对应到展示，非常直观方便，让我们抓住重点

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200519224657.png?ynotemdtimestamp=1595748679079)