---
title: Obsidian Day-Planner 插件新版本使用（0.7.X以上版本）
subtitle: 新版本插件介绍
author: lazytime
url_suffix: obsidiandayplan
tags:
  - 其他
categories:
  - 其他
keywords: Obsidian
description: Obsidian 使用
copyright: true
date: 2023-09-06 11:10:21
---
# 引言

这几天把 **Obsidian** 和**Day-Planner**插件做了升级，突然发现原来的**时间视图**不见了。

起初个人认为是新版本的Obsidian和插件不兼容，于是选择换成旧版本，后来看了一眼**Day-Planner**更新日志，才发现原来作者基本把插件重做了。

更新之后的**Day-Planner**插件和旧版本差别挺大，个人花了不少时间熟悉新版本，本文将介绍如何快速使用和适应新版本的**Day-Planner**插件。

<!-- more -->

# Github地址

[GitHub - ivan-lednev/obsidian-day-planner: An Obsidian plugin for day planning and managing pomodoro timers from a task list in a Markdown note.](https://github.com/ivan-lednev/obsidian-day-planner)

## 简单说明

> 0.7.0 significantly changes what the plugin looks like and what it does. If you like to have some of the old behaviors back, [consider creating an issue](https://github.com/ivan-lednev/obsidian-day-planner/issues).
> 
> If for some reason you still want to use the old version, there are community forks, which you can use via [BRAT](https://github.com/TfTHacker/obsidian42-brat). [Here is one such fork](https://github.com/ebullient/obsidian-day-planner-og).

Github的README部分说的很清楚了，作者在 0.7.X 之后对于 Day-Planner 插件的功能和界面进行**重做**，基本可以当做一个新插件使用。

如果习惯老版本的使用方式，可以进入下面的网站下载之后进行手动安装。

> 这也意味着旧版本做任何扩展和维护

[GitHub - ebullient/obsidian-day-planner-og: An Obsidian plugin for day planning and managing pomodoro timers from a task list in a Markdown note.](https://github.com/ebullient/obsidian-day-planner-og)

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905105427.png)

# 新版本如何使用？

下面来看下新版本插件的使用流程。
## 快速上手

新版本的Day-Planner通过左侧的菜单栏访问：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905120656.png)

点击进入，发现这是一个“周计划”视图：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905120752.png)

功能设计有点类似 IOS系统的 的日历视图，在任意的时间轴可以做自己的时间规划，比如我们选择在 “**9月4日的8点设置出门**”：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905121029.png)

点击对应的时间区间，插件会在选中的区间生成“item”。

> 注意下面的界面“New Item”可以上下拖动。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905121153.png)

它的时间范围对应如下，作者的设计思路是：“**一个Item为半小时**”，在周计划界面拖动 Item 也是**以固定半小时的间隔动态调整**。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905121231.png)

注意，它并没有（个人期望的）通过拖动“一长段”时间设置1到两个小时或者更长时间的规划，希望作者后续更新可以加入这项功能。

以上就是对于新版本插件的简单快速上手，说实话半小时的设置操作不是很方便，个人做了一些直接可以复制粘贴的**每日规划模板**，只需要按照所需的日志一键粘贴就可以快速上手内容，个人习惯是一小时为“一段”，分割一天为24份，具体效果可以继续阅读下文。
## 个人模板

### 旧版本

```
#日记

## 每日记录模板

- [ ] 06:00 - 07:00 
- [ ] 07:00 - 08:00
- [ ] 08:00 - 09:00
- [ ] 09:00 - 10:00
- [ ] 10:00 - 11:00
- [ ] 11:00 - 12:00
- [ ] 12:00 - 13:00
- [ ] 13:00 - 14:00
- [ ] 14:00 - 15:00
- [ ] 15:00 - 16:00
- [ ] 16:00 - 17:00
- [ ] 17:00 - 18:00
- [ ] 18:00 - 19:00
- [ ] 19:00 - 20:00
- [ ] 20:00 - 21:00
- [ ] 21:00 - 22:00
- [ ] 22:00 - 23:00

# 规划

# 输入

# 输出

# 专注详情
```
### 新版本

新版本的出现导致旧模板也需要做更新，对应新版本的插件做了下面的微调：

```
#日记
# Day planner

-  06:00 - 07:00 
-  07:00 - 08:00
-  08:00 - 09:00
-  09:00 - 10:00
- 10:00 - 11:00
-  11:00 - 12:00
- 12:00 - 13:00
-  13:00 - 14:00
- 14:00 - 15:00
- 15:00 - 16:00
- 16:00 - 17:00
- 17:00 - 18:00
- 18:00 - 19:00
- 19:00 - 20:00
- 20:00 - 21:00
- 21:00 - 22:00
-  22:00 - 23:00

# 规划

# 输入

# 输出

# 专注详情
```

上面的内容复制黏贴到对应日期文件，就可以在对应的日期指定一个“基础模板”，当然从截图看效果怪怪的，其实只要根据自己的需要填写内容即可正常显示：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905120752.png)

## 注意点

需要注意，如果读者想修改上面提供自定义模板内容，顶部的`# Day planner`这样的一级标题是必须要保留的，**不能做任何调整**。

为了方便理解，直接看下面两张图对比：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905115513.png)

`# Day planner`可以修改为多级标题，但是`# Day planner`这个“关键词”的本身的文本不能做任何变更，否则就变成下面这样“诡异”的情况：


![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905115437.png)

这样的设计也可以理解，顺着作者的设计规划自己的模板即可。
## Ctrl + P 变化

如果使用过旧版本Day-Planner，我们可以在`Obsidian`中使用`ctrl + P`的快捷按钮做快捷操作，没错，新版本插件这里的功能按钮也完全不一样了。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905111854.png)

下面按照从上往下的顺序，介绍截图每一项快捷指令的作用。
### 1. 展示本周的时间视图规划。

- 优点：和点击左边的按钮效果相同，优点是不需要动用鼠标靠键盘就可以快速访问 =v=。
- 缺点：会覆盖当前`Tab`，个人更希望新开`Tab`展示。
### 2. 进入当天的Day-Planner

- PS：旧版本是在 Day Planners 的文件夹下面每日自动创建。
- 新版本：如果发现对应日期的md文件不存在，默认会在`Obsidian`的仓库 **根目录** 创建一个新的文件，命名格式如下：

![2. 进入当天的Dat-Planner](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905112419.png)
### 3. 展示时间线

第三个选项是时间线展示，选择之后`Obsidian`右边的视窗多一个“选项”，这样就可以看到**当天的时间线**（新版本外观重做了）：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905113547.png)

左右箭头是切换当前日期的上一天和下一天，右上角的按钮可以选择：

- **调节当前时间线的间隔**。比如个人设置了Zoom的等级为1，整个时间线的间隔会窄一些。

- **是否自动滚动到当前的时间点**，假设现在7点50，就会自动定位到7点-8点这个时间段。

> 选择自动滚动，时间线会随着当前时间自动滚动定位：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905113723.png)
### 4. 插入 Day planner

效果就是在**当前的光标位置插入一个“Day planner” 的一级标题**。

比如下面这样的效果：

```
#Day planner
```

个人认为这个选项没什么用，模板赋值粘贴更为方便。
### 5. 进入当天的日期视图

最后一项就是进入到当天的日期视图，也是一个快捷操作。

## 设置变化

新版本插件的设置基本全默认即可，个人的调整为：变更 Zoom Level=1，把第一项做了选中，其他基本保持原样。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905122240.png)

## 小结

新版本的 Day-Planner 整体上比较容易上手，比起旧版本要实用不少，希望作者后续继续更新。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905114923.png)

# 写在最后

与时俱进，个人认为旧版本**记录每一天的时间流**不利于做个人规划，很容易变成流水账，个人还需要额外借助**时间序**或者**滴答清单**这样的软件，做一周或者更长时间的时间规划。

新版本把整个插件重做，贴合了目前大部分时间管理APP的风格，整个界面十分的简洁直观，当然目前的插件不是很完善，希望作者后续可以继续改进操作。

从个人角度来看，本次插件“换代”最大意义是，类似滴答清单这样需要付费高级会员才能体验“日历视图”，在 Day-Planner 是完全免费的，白嫖党表示非常Nice。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230905105639.png)

总而言之，这次更新非常香。