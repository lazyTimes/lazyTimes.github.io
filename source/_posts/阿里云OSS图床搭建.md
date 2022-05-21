---
title: 阿里云OSS图床搭建
subtitle: 到了最后还是得掏钱，唉
author: 阿东
url_suffix: abioss
tags:
  - 其他
categories:
  - 其他
keywords: OSS
description: 图床处理
copyright: true
date: 2022-05-21 18:50:06
---

# 阿里云OSS图床搭建
# 介绍
过去介绍过Gitee的图床搭建，用了一年多都挺稳定了，访问速度尚可，但是2022年开始发现不是很稳定，并且部分网站会发现“图片消失”的情况，F12看一下发现是很多外国网站给直接**302临时重定向**了。

所以为了保证数据安全，这里无奈只能掏钱买阿里云的 OSS做图床了，查了下价钱也还能接受，标准LRS存储一年也就10块钱。

本文就根据OSS配置再结合个人常用的几个软件来总结阿里云OSS相关配置和应用。

<!-- more -->

## 计费方式
在了解具体的使用之前，这里简要介绍一下计费的方式。

阿里云有目前有两种计费方式，如果你不想买资源包等等操作，那么默认开通OSS之后就可以直接拿来用，直接**按量计费**的方式即可，先使用，后付费。

- 按量收费计费公式：OSS的使用费用每小时结算一次，计算公式为：**费用=实际资源使用量×对应资源每小时单价**。

- 资源包：预先购买针对不同的计费项推出的优惠资源包，在费用结算时，优先从资源包抵扣用量，先购买，后抵扣，适用于业务用量相对稳定的场景。

> 注意：资源包一定要根据自己创建和使用的Bucket进行购买，比如标准存储就买标准存储的，低频存储的就买低频存储的，**千万不要买错了，买错了不能反悔**。当然标准的LRS比较便宜，但是粗略看了一下其他几个Burket选项都挺贵的，购买之前一定要确认清楚自己的需求和使用的Bucket类型。
> 如果对于费用计费有顾虑，可以阅读“资源包管理“中的”购买了资源包为什么还会欠费？“，里面都有相关解释。
![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032203859.png)

这里假定读者都是给个人学习或者简单使用的情况，所以使用的是购买资源包计费方式。

## 存储类型
官方做了一张表，其实一般使用“标准类型”就够了，可能有的读者还想图便宜会想要买一个“低频访问”来玩玩，但是只要你去看一下低频访问资源包价格就会发现你会被文字游戏给坑了。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032207275.png)

## 新人优惠
新人优惠很重要，查了一下很多文章不会说这个东西，这里有必要强调一下，阿里云的OSS对于新人来说有送3个月免费100GB流量，基本相当于让你免费用3个月，这一点还是挺香的（<s>到期了薅完阿里薅腾讯云的</s>），所以创建OSS之后先不要急着用，先把免费的资源包领一下：

地址：[对象存储OSS_云存储服务_企业数据管理_存储-阿里云 (aliyun.com)](https://www.aliyun.com/product/oss?spm=5176.21213303.J_6704733920.11.402753c97AYB84#J_3332243220)。

滚动条下拉选择套餐即可。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032156464.png)

地域选择大陆通用即可：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032157443.png)

购买之后我们创建相关的Burket然后上传的时候就会从资源包扣费了。
注意你创建的Burket一定要和你的资源包匹配！
注意你创建的Burket一定要和你的资源包匹配！
注意你创建的Burket一定要和你的资源包匹配！

很重要，否则不明不白的额外扣费让人恼火和后悔。

# 阿里云 OSS基础配置
进入官网：[阿里云-上云就上阿里云 (aliyun.com)](https://www.aliyun.com/?spm=5176.21213303.J_3207526240.1.288153c9TDGktx)，在产品中选择 OSS，如果从来没开通过OSS，阿里云这里会给一份协议确认然后确认用户开通，这里就不截图了。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032023825.png)

进入管理页面，刚进去看不知道要干啥，所以直接点击右边的OSS新手入门来了解也是一种方式。
[OSS阿里云_ OSS是什么意思_对象储存OSS_阿里云OSS学习路径图_OSS Learning Path - 阿里云 (aliyun.com)](https://help.aliyun.com/learn/learningpath/oss.html?spm=5176.8465980.guide.1.41231450mh5Ll7)

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032036671.png)

我们切换到B**ucket**列表，选择“创建Bucket”。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032105135.png)

进入页面之后，我们填写下面的内容：
- Bucket名称，唯一命名，起个自己喜欢的名字即可
- 地域：选择和自己所在城市比较近的城市，国内选择国内的区域和节点即可。
- Endpoint：需要的话可以记录一下，比如我我选择的是：oss-cn-shenzhen.aliyuncs.com
- 存储类型：如果仅仅作为备份使用，低频访问比较合适，但是如果是对外使用不管流量多少还是建议用标准的，归档存储一般用于永久存储备份重要数据。
- 同城冗余和版本控制没啥必要，不用开
- 读写权限：如果我们作为图床，需要用“公共读”允许匿名用户访问数据。（和PicGo配置有关）
- 实时日志和定期备份个人认为如果是自己用也是没有必要，骗钱玩意。

## 购买资源包
创建完成之后，我们先不急着操作，我们先买个资源包，这里个人使用了“新人优惠”不给买了，这里就不演示购买操作了。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032215220.png)

## AccessKey管理
接着是用户配置部分，我们需要在OSS中配置允许对外访问的AccessKey，这里我们点击右上角“头像”的"AccessKey管理“，这里其实用户体验不是很好。
![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032249369.png)


由于图床会对外访问，所以建议不要使用主账户的AccessToken进行操作，而是使用子账户方式进行操作处理，这里进入之后可以看到RAM用户管理，点击“创建用户”。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032253953.png)

“创建用户”其他都可以自由操作，但是一定要选择“Open API”，这里需要进行安全验证。创建完整之后，我们便拥有了**AccessKey ID，AccessKey Secret**这两个关键配置，注意这两个配置只能查看一次，建议复制到自己本地存储后面需要使用到。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204032254752.png)

**Burke授权**
创建用户之后，我们需要给创建的子用户授权，在管理页面选择新建子商户之后选择给商户添加权限。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204041251396.png)

然后回到刚刚创建的bucket，在文件管理内给新建的用户授权。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204041300935.png)



## 配置PicGo
PicGo是什么这里就略过了，我们直接来看PicGo的配置

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204041303779.png)

关键部分：在软件中我们选择“阿里云OSS配置”然后根据参数填写下面的内容：
- keyId：这里用之前新建的**子用户**的 **AccessKey**。
- KeySecret：这里使用新建**子用户**的 **AcessSecret**。
- 存储空间名称：这里按照下图填写**Bucket域名**，注意这里只需要`.aliyuncs.com`需要删除。
- 默认存储区域：这里按照下图填写**Endpoint**，注意这里只需要`.aliyuncs.com`需要删除。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/202204041306584.png)

最后个人的配置如下：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204041315333.png)


## 配置Typera
安装Typera这里就跳过了，我们打开软件之后选择“图像”，然后切换到PicGo，验证一把之后会提示成功信息，之后我们修改上面“插入图片时....”的操作改为“上传图片”，以后图片都会往PicGo进行上传，不会出现在本地的一个临时路径了。

> 这里不是很建议直接执行上传操作，更建议先放到一个指定文件夹然后确认无误之后进行复制粘贴的上传替换，当然OSS流量基本够用。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204041345412.png)

## obsidian配置
具体可以看作者的文章，基本上安装一个插件之后“Enable”即可直接使用，直接往Obsidian进行粘贴就会直接委托PicGo上传，然后出翔相关路径。
[在Obsidian中使用图床实现“一次上传、多次搬运”省心又省力 - 经验分享 - Obsidian 中文论坛](https://forum-zh.obsidian.md/t/topic/388)

最后，我们截图粘贴查看是否触发PicGo上传，最终截图的路径如下：

https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204041333383.png

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204041333383.png)

# 写在最后
gitee的图床个人目前已经不再进行上传，后续都将会改用OSS，另外个人建议定期给图床做一下本地备份，虽然可能并没有特别大的意义。

另外如果对于图片的重要性不大，可以直接使用免费的图床。