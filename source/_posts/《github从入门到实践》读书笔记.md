---
title: 《github从入门到实践》读书笔记
subtitle: 这本书适合给完全不知道github是啥到人快速入门使用，其实都是随便百度都可以百度到到东西
author: 阿东
url_suffix: gitHub_rumen
tags:
  - 书籍
categories:
  - 读书笔记
keywords: github
description: 这本书适合给完全不知道github是啥到人快速入门使用，其实都是随便百度都可以百度到到东西
copyright: true
date: 2022-02-19 23:01:11
---

# 前言

  这本书适合给完全不知道github是啥到人快速入门使用，其实都是随便百度都可以百度到到东西，不过书到话会直接给你讲怎么用，没有其他到口水话。另外这本书完全不建议买实体书，完全不建议，真正有一点点用的就是前面的小半部分的命令和效果，后面大量的图片都是来凑数的。

# 推荐程度

  这个没啥好推荐的，个人是凑单买的这本书，大致翻了一下就翻完了。

# 个人评价

  本着看书就要编写读书笔记的原则还是写了，但是实际上没有啥做读书笔记的价值建议更多的是自己动手实操。

 <!-- more -->

# 内容概要：

  实践大于理论的书，跟着书的步骤瞧一瞧相关的命令熟悉一下github的基本操作挺好的，这里大概列一下前半部分的内容，后半部分凑数的。

  下面列举个人做的摘录笔记内容，大致浏览一边基础和常用的命令即可，具体的使用可以使用到的情况下在进行进一步学习。

```Git
基础的命令内容：
+ git add 添加文件到分支管理
+ git commit 提交内容到暂存区
+ git log xx 查看日志
+ git history 查看当前的git状态
+ git init 初始化
+ git diff 查看区别和对比
  + git diff HEAD
  
分支操作
+ git branch 分支预览
+ git checkout -b 创建或者切换分支
  + git branch 和 checkout 会为没有分支而创建分支

什么是特性分支？
  创建单一特性，除此之外不进行任何作业分支。可以理解为特性分支。
  + git checkout 上一个分支
合并两个分支

+ git reflog 当前仓库过去操作日志，可以将误操作回溯。
+ git checkout master 更新分支到master
+ git merge 合并分支
 + git merge -no-diff fix-B -no-diff 不重复
+ git log --graph 图表查看分支,比较实用的命令
+ 更高提交操作
  + git reset 回溯历史版本
  + git reset --hard 哈希值
建议：
先创建一个分支，然后回溯到上一个分支，这样可以保证回溯出错的时候可以重新开始。

合并冲突
  修改提交的信息：
+ git commit --amend 修改提交的内容
+ git commit -am 一次提交与添加的操作
+ git rebase -i 压缩历史，更改错误的提交记录
  + git rebase -i HEAD~2 
  ~: 前面两个提交，～2表示前面两个提交
+ git remote add 添加一个远程仓库
+ git push 推送
  $ git push -u origin master 设置推送到其他分支

拉取分支
+ git clone 获取分支
  + git checkout -b fex-D origin/feat-D 获取远程分支



其他学习：
1. pro git
2. learn gitbraching
3. try git

创建和维护仓库
+ git remote add upstream git://github......git
+ git fetch 设置远程仓库
接收pull-request 使用put 分支接收远程请求，合并到主分支。



```



# 写在最后

  小结一下之前的笔记内容。