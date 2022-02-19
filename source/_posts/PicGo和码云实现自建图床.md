---
title: PicGo和码云实现自建图床
subtitle: picgo搭建图床
author: lazytime
url_suffix: picgo
tags:
  - 其他
categories:
  - 其他
keywords: PicGo和码云实现自建图床
description: 自建图床
copyright: true
date: 2020-07-26 11:30:10
---

# PicGo和码云实现自建图床

## 最好的教程

知乎的一篇教程：https://zhuanlan.zhihu.com/p/102594554

## 测试图床链接：

链接1 ：

https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200318234353.png

链接2：

https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200318234621.png

链接3：

https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319094133.png

![最终结果1](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200318234353.png?ynotemdtimestamp=1595729096959)

![最终结果2](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200318234621.png?ynotemdtimestamp=1595729096959)

![最终结果3](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319094133.png?ynotemdtimestamp=1595729096959)

<!-- more -->

## 为什么要使用码云代替图床

- github毕竟是外国服务器，上传过程中因为网络问题有可能会无法使用的情况
- 码云作为国内的“github”，访问和上传速度更快，所以使用码云更为方便
- 目前`PicGo 2.0`之后已经有人集成了插件，使用和github一样简单好用

## 步骤：

### 1. 下载PicGo

官网地址：https://molunerfinn.com/PicGo/

Github 快捷网址：https://github.com/Molunerfinn/PicGo/releases/tag/v2.2.2 选择对应的版本下载即可

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319131933.png?ynotemdtimestamp=1595729096959)

更多信息访问Github：https://github.com/Molunerfinn/PicGo

### 2. 安装PicGo

安装过程就忽略了，安装完成之后打开软件

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319132027.png?ynotemdtimestamp=1595729096959)

### 3. PicGo 安装gitee 插件

前提条件：**注意PicGo安装的前提条件是2.0版本之后新增的插件功能！！！**

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319132225.png?ynotemdtimestamp=1595729096959)

将鼠标滚轮滚到最下面，会发现有一个插件设置

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319132358.png?ynotemdtimestamp=1595729096959)

选择安装 `gitee-uploader 1.1.2`这个插件

> **安装失败如何解决**？
>
> 注意该插件需要node.js 的环境，Node.js的安装如下
>
> http://nodejs.cn/download/
>
> 进入中文官网之后，安装对应的版本即可
>
> window 下面的exe程序安装之后会自动的配置环境变量，这时候我们可以使用命令查一下是否有Node环境
>
> 1. win 键 + R 打开**运行**窗口
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319132719.png?ynotemdtimestamp=1595729096959)
>
> 1. 输入命令`node -v`
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319132750.png?ynotemdtimestamp=1595729096959)
>
> 1. 如果出现如上所示的内容则证明安装成功

### 4. 码云搭建图床仓库

1. 进入自己的码云：https://gitee.com/lazyTimes/projects 这是我的码云地址

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319133220.png?ynotemdtimestamp=1595729096959)

1. 选择右上角有一个 `"+"`号，选择新建仓库

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319133327.png?ynotemdtimestamp=1595729096959)

1. 填写基本信息，页面翻到最下面，选择保存

![image-20200319133550209](http://null/)

1. 新建仓库成功

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319133836.png?ynotemdtimestamp=1595729096959)

### 5. 获取gitee私人令牌

和github一样，在最终设置之前我们需要获取一下gitee令牌用于上传

1. 登录gitee之后，右上角选择

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319214555.png?ynotemdtimestamp=1595729096959)

1. 选择“私人令牌”

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319214625.png?ynotemdtimestamp=1595729096959)

1. 选择“生成令牌”，进入到令牌到创建页面

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319214721.png?ynotemdtimestamp=1595729096959)

1. 勾选如图到内容，一般只需要用到前面几项就可以满足我们到需求。按要求输入密码即可生成成功

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319214905.png?ynotemdtimestamp=1595729096959)

1. 生成令牌成功，将令牌内容保存在自己到笔记，防止忘记

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319215102.png?ynotemdtimestamp=1595729096959)

经过如上步骤之后，我们基本就算是大功告成了，现在只需要在PicGo里面配置一下即可

### 6. PicGo 配置

- repo：必填，填写gitee上面的仓库名称
- branch：一般默认master 即可，有需要可以建立自己到分支
- tolken：要用到上一节所述到私人令牌，如果忘记了可以进入页面之后重新获取一下新令牌
- path：写上文件存放到位置，一般写上`img`即可
- customPath：定义传输到格式，一般可以不用管
- customUrl：自定义上传到链接

> 不知道怎么获取repo地址？
>
> 1. 进入自己到图床仓库到主页，复制地址栏到内容
> 2. 复制下方到用户和项目名称
>
> **注意中间到内容不要存在空格，删除注意不要多删除字符造成上传失败**
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200319215400.png?ynotemdtimestamp=1595729096959)

最终到配置结果如下所示

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200319131435.png?ynotemdtimestamp=1595729096959)

# 总结

由于github存在限制加上外网访问普遍较慢到问题，如果网速不给力并且没有科学上网工具建议使用gitee作为图形仓库。可以在1S内上传，非常方便。赶紧把gitee用起来把