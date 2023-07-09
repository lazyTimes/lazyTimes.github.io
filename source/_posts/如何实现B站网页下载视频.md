---
title: '''如何实现B站网页下载视频'''
subtitle: 切勿随意传播
url_suffix: bdownload
author: lazytime
tags:
  - 其他
categories:
  - 其他
keywords: 下载
description: B站下载视频
copyright: true
date: 2022-11-16 11:05:20
---

# 引言

此方法建议不要随意在网上传播，虽然本意想让更多人知道，但是确实比较怕菊爆党。

<!-- more -->

# 步骤

1. 先下载油猴插件（需要梯子插件搜 **Tampermonkey**）。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221116110048.png)

网站地址：[Tampermonkey • Userscript Sources](https://www.tampermonkey.net/scripts.php)

之后前往下面的搜索页面，搜索B站[空格]下载

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221109143839.png)

2. 找到排序的第一个插件：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221109143929.png)

3. 进入页面下载即可，详细介绍这里就这里不贴图了，低调低调，另外脚本的介绍页还藏了福利，不想错过请仔细阅读

4. 进入到自己想要下载视频的页面<s>先来个一键三连</s>，然后就可以看到下载视频的按钮，点击之：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221109144052.png)

5. 进入下面的页面需要做这几件事情：

- 安装下面提到的依赖软件
- 把RPC网址进行切换
- 设置自己的下载位置
- 全选批量下载，然后等待即可。

**默认RPC默认地址**
(1)、Motrix RPC默认地址：ws://localhost:16800/jsonrpc
(2)、Aria2 RPC默认地址：ws://localhost:6800/jsonrpc

# 依赖软件

## Motrix

简单好用的开源软件： [Motrix](https://motrix.app/zh-CN/)。有点像绿色版某雷。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221109150021.png)


安装之后的下载效果如下：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20221109140515.png)

# 下载

虽然B站下载器也可以做到，但是个人更喜欢网页无缝下载。另外插件介绍里面也提示了不单单是B站下载视频，个人没那么多心思去折腾其他网站，这里就留给读者自己尝试了
