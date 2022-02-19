---
title: Hexo 迁移备份
subtitle: 创建Git分支将Hexo博客迁移到其它电脑
author: lazytime
url_suffix: hexoback
tags:
  - Hexo
categories:
  - 其他
keywords: Hexo备份
description: 创建Git分支将Hexo博客迁移到其它电脑
copyright: true
date: 2020-07-26 17:26:28
---



# 创建Git分支将Hexo博客迁移到其它电脑

## 迁移前准备：安装hexo博客必要的软件

+ 下载安装Git客户端
+ 安装node js
+ 从git 仓库拉去原来的项目

<!-- more -->

## 采取方式

1. 采取新建仓库的方式
2. 分支存放源代码

> git clone https://github.com/lazyTimes/lazyTimes.github.io.git



## 必备文件

| 文件(夹)     | 说明                 |
| :----------- | :------------------- |
| scaffolds/   | 博客文章模板         |
| source/      | 所有的博客文章       |
| themes/      | 网站主题             |
| .gitignore   | push时需忽略的文件   |
| _config.yml  | 站点配置文件         |
| package.json | 依赖包的名称和版本号 |

## 备份流程

### 1. 拉取已经部署上去的项目

```
git clone https://github.com/lazyTimes/lazyTimes.github.io.git
```

### 2. 拷贝需要备份的`元数据`

具体查看上方的必备文件

参考截图:

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200726154723.png)

### 3. 删除主题的.git 配置

执行如下命令删除不必要的内容

`rm -rf thems/next/.git*`

### 4. 创建名为hexo的分支

`git checkout -b hexo`

### 5. 把文件存放到暂存区

```
git add --all
```

### 6. 提交变更

先提交所有的改动内容

```
git commit -m "hexo-2"
```

然后使用如下命令把内容推送到分支

```
git push --set-upstream origin hexo
```

> 如果没有在Git config 设置用户名和密码，推送的时候会提示设置，根据提示设置用户名和密码即可

### 7. 源码推到分支上

```
$ git add .
$ git commit -m "xxxx"
$ git push origin hexo
```

## 更加推荐的方式

为了保证我们的源码的一些敏感配置不泄露，建议使用私有仓库进行存储，接下来说一下私有仓库的配置方式





