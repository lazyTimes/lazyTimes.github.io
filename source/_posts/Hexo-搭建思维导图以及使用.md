---
title: Hexo 搭建思维导图以及使用
subtitle: 这个人很懒，不想写副标题
author: lazytime
url_suffix: hexo-study1
tags:
  - Hexo
categories:
  - 其他
keywords: Hexo，搭建思维导图以及使用
description: 搭建思维导图以及使用
copyright: true
date: 2020-07-26 12:31:59
---

# Hexo 搭建思维导图以及使用

## 1. Hexo 如何搭建

- 详细查看Hexo学习博客：

## 2. 选择：

- 百度的kitmap
  - https://qsli.github.io/2017/01/01/markdown-mindmap/
- Hexo 思维导图插件
  - 
- 为什么使用百度的kityminder
  - 使用简单
  - 百度开发，使用国内插件
- 使用hexo思维导图插件的原因
  - 简单好用
- 什么是Kityminder
  - 分成两部分
    - kity-core
    - kity-editor
- 如何学习kityminder
  - https://qsli.github.io/2017/01/01/markdown-mindmap/
- 如何学习hexo 思维导图插件
  - https://zhuanlan.zhihu.com/p/75467441

## 3. 使用hexo 插件构建思维导图

- 跳转到hexo的目录下面，执行如下命令

  - `npm install hexo-simple-mindmap`

- 在博客内部加入如下

  ```
  {% pullquote mindmap mindmap-md %}
  [Hexo 的思维导图插件](https://hunterx.xyz/hexo-simple-mindmap-plugin-intro.html)
  - 前言
  - 使用方法
  {% endpullquote %}
  ```

## 4. 查看效果

进入文章，即可渲染出对应的思维导图

# 推荐其他的方式

- 使用xmind 话思维导图
  - 为什么？
    - 软件画图，可以一键生成图片
    - 样式可以调整，不需要切换
    - 简单好用，相对于Markdown编写更为直观
  - 怎么使用xmind
    - 百度搜索xmind 进入
    - 推荐使用xmind 的升级版本