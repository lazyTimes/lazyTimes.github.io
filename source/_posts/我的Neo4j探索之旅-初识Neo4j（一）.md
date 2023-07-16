---
title: 我的Neo4j探索之旅 - 初识Neo4j（一）
subtitle: 学习neo4j的使用和基本语法
url_suffix: neo4j1
author: lazytime
tags:
  - 数据库
categories:
  - 数据库
keywords: neo4j,图数据库
description: 学习neo4j的使用和基本语法
copyright: true
date: 2020-11-03 00:03:27
---

# 我的Neo4j探索之旅 - 初识Neo4j（一）

## 目录

[TOC]

## 前言：

neo4j 这个东西在国内用的很少，目前能百度的资料也是很早之前的几篇了，我针对neo4j 3.5 的版本进行一次学习和记录，以及实际的工作需求我也遇到了，后续会开源一个剔除业务的开源项目，有兴趣的读者可以了解一下图数据库的中间件，还是蛮有意思的。

<!-- more -->

## 它是什么？

+ 百度百科定义： http://www.youmeek.com/my-learning-way/?tdsourcetag=s_pcqq_aiomsg 
+ 官方网站： https://neo4j.com/ 
+ 官方文档地址： https://neo4j.com/docs/ 
  + Java Driver API Docs
  + Neo4j Javadocs
  + Cypher Manual
  + Spring data neo4j 这个是spring 管理neo4j的框架
+ 维基百科介绍: https://en.wikipedia.org/wiki/Neo4j 

##  它的历史 

+ 1.0版于2010年2月发布。

+ Neo4j 2.0版于2013年12月发布。
+ Neo4j 3.0版于2016年4月发布。
+ 2016年11月，Neo4j成功获得了由Greenbridge Partners Ltd.牵头的3600万美元D轮融资[[15\]](https://en.wikipedia.org/wiki/Neo4j#cite_note-15)
+ 2018年11月，Neo4j成功获得了由One Peak Partners和Morgan Stanley Expansion Capital牵头的8000万美元E轮融资，其他投资者包括Creandum，Eight Roads和Greenbridge Partners参与了此次融资

## 应用场景

+  https://zhuanlan.zhihu.com/p/36404872  知乎介绍neo4j
+  https://juejin.im/post/5b559990518825196b01dfc2  掘金文章

## 同类产品比较

+  https://blog.csdn.net/belalds/article/details/87917900  主流图数据库Neo4J、ArangoDB、OrientDB综合对比：架构分析
+  https://www.ljjyy.com/archives/2019/09/100584.html  在实际的项目选型中使用

## 为什么学习它
- 公司业务需求，需要使用可视化拓扑图展示数据
- 标签库使用mysql展示图形结构比较困难，转而使用图数据库解决

## 为什么要使用neo4j
+ https://www.cnblogs.com/rubinorth/p/5846140.html 参考
+ https://zhidao.baidu.com/question/1052090251428505379.html 百度知道

## 哪些人不喜欢它
+ neo4j 属于老牌图数据库
+ neo4j 不支持分片，对分布式的系统支持不是很好，推荐单机部署

## 我要怎么做（按优先级从高到低排序）
+ 看文档：
    + 安装Neo4j desktop 
      
      + 启动，进入localhost: 7474
      + 参考desktop 的快速入门操作案例
      + 进入官网，选择DEVELOP-Document，阅读如下内容：
        +  Getting Started 简单的了解Neo4j,地址如下：
          + https://neo4j.com/docs/getting-started/current/
        + Cypher manaral 比较重要： CQL语言十分强大，好用
        + Neo4j 1.7 Driver Manual neo4j 驱动以及api使用
      + Java Driver API Docs java API
+ 自己写 Demo

  + 后续会将个人实验内容上传到github
+ 参考别人 Demo

  + 参考地址：https://github.com/IsFive/neo4j-vis.js.git git 地址
  + 后面根据该项目进行二次开发，目前已上线正式环境。
+ 项目场景模拟
    + 让业务去推动技术
    + 明确需求
+ 遇到问题

  + 科学上网到国外使用谷歌进行搜索，目前国内使用较少
  + 查看csdn 博客，有部分问题的解决办法
  + 关于关系型数据库 与 neo4j数据库的数据同步问题 
  + Neo4j 与 vis 的使用问题



# 如何安装neo4j社区版本（免费）（windows - 10）

## 1. 进入官网，下载软件

https://neo4j.com/graph-algorithms-book/?ref=home-banner

点击右上角：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827111934.png)

点击下载：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827142816.png)

进入到如下页面

输入对应信息，选择下载

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827143345.png)

>  PS： 外网软件的下载真的慢的想死，这里提供一个诀窍
>
>  ![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827143459.png)
>
>  1. 进入到如下页面，右击蓝色连接
>  2. 复制连接地址
>  3. 在迅雷里面，新建任务，然后粘贴地址进去
>  4. 迅雷会找到资源然后提示你下载
>  5. 下载，不出意外飞速下好软件包
>
>  （本迅雷为破解版，个人自己使用，不对外开放）
>
>  ![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827143602.png)



> 线上的版本使用 版本为**3.5.12** ，官方最新版已经有了4.0的版本了
>
> 这里我也提供了安装包，自己下的话需要翻墙加迅雷下才有可能拉下来
>
> Windows:
>
> 链接：https://pan.baidu.com/s/1yWkIdUUt-RzvSuke89nJCw 
> 提取码：3c21
>
> Linux:
>
> 链接：https://pan.baidu.com/s/1ljzS5DIYo5n9fCIzKkAMiw 
> 提取码：bnrf 

## 2.  安装JDK

这个请自行百度，教程烂大街，不过注意安装 **JDK1.8** 版本以上，否则是无法使用的

## 3. 配置Neo4j环境变量

将下好的包解压到对应的位置之后，我们可以配置环境变量

环境变量如下

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827144938.png)

> NEO4J_HOME
>
> D:\zxd\tool\neo4j-community-3.5.12-windows\neo4j-community-3.5.12

## 4. neo4j 启动

1. **管理员模式**打开CMD， `cd` 到 `D:\zxd\tool\neo4j-community-3.5.12-windows\neo4j-community-3.5.12\bin` 下面（记得切换盘符）

2. 请查看下面的常用命令，根据自己的实际情况选择

```
# 启动服务
neo4j(.bat) start
# 重启服务
neo4j(.bat) restart
# 停止服务
neo4j(.bat) stop
# 控制台模式启动
neo4j(.bat) console

```

3. 开启neo4j，看到 类似`successful`的字样就代表运行成功了
4. 进入到 http://localhost:7474/browser/

5. 进入主页面，neo4j安装成功

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827150440.png)

## 5. 安装有可能的问题

此部分是针对（4） 有可能失败的情况下进行尝试：

### 常见问题1

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827145703.png)

输入如下的命令，安装neo4j 的服务

```shell
# 安装neo4j 服务
neo4j install-service
# 卸载neo4j 服务
neo4j uninstall-service
```

### 常见问题2：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827145921.png)

这种情况可能是你安装服务的时候，neo4j默认已经给你启动了，尝试访问 http://localhost:7474 看下能不能访问，如果可以访问，证明没有出现问题

> 如果依然没有解决，请尝试 `neo4j.bat stop` 先关闭服务，或者重新安装一遍neo4j的服务

### 常见问题3：

下面这个问题是一个比较奇怪的问题，我之前在上线部署的时候遇到过一次

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201102104408.png)

解决办法：![img](file:///C:/Users/123/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)https://blog.csdn.net/together_cz/article/details/97918789

 修改bin下面，有一个文件：neo4j.ps1

需要把前缀的 $PSSCRIPT 改为你的安装路径，然后执行neo4j 的命令就不会报错了

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201102105121.png)





## 6. neo4j 实现外网访问：

在`conf` 里面的 `neo4j.conf` 中修改:

```
#dbms.connectors.default_listen_address=0.0.0.0
```



## 7. Neo4j 部分配置参数详解：

在`conf/neo4j.config`中有对应的内容：

```yml
其他配置项目从网上摘抄部分
修改相应配置如下：

# 修改第22行load csv时l路径，在前面加个#，可从任意路径读取文件
#dbms.directories.import=import

# 修改35行和36行，设置JVM初始堆内存和JVM最大堆内存
# 生产环境给的JVM最大堆内存越大越好，但是要小于机器的物理内存
dbms.memory.heap.initial_size=5g
dbms.memory.heap.max_size=10g

# 修改46行，可以认为这个是缓存，如果机器配置高，这个越大越好
dbms.memory.pagecache.size=10g

# 修改54行，去掉改行的#，可以远程通过ip访问neo4j数据库
dbms.connectors.default_listen_address=0.0.0.0

# 默认 bolt端口是7687，http端口是7474，https关口是7473，不修改下面3项也可以
# 修改71行，去掉#，设置http端口为7687，端口可以自定义，只要不和其他端口冲突就行
#dbms.connector.bolt.listen_address=:7687

# 修改75行，去掉#，设置http端口为7474，端口可以自定义，只要不和其他端口冲突就行
dbms.connector.http.listen_address=:7474

# 修改79行，去掉#，设置http端口为7473，端口可以自定义，只要不和其他端口冲突就行
dbms.connector.https.listen_address=:7473

# 修改227行，去掉#，允许从远程url来load csv
dbms.security.allow_csv_import_from_file_urls=true

# 修改246行，允许使用neo4j-shell，类似于mysql 命令行之类的
dbms.shell.enabled=true

# 修改235行，去掉#，设置连接neo4j-shell的端口，一般都是localhost或者127.0.0.1，这样安全，其他地址的话，一般使用https就行
dbms.shell.host=127.0.0.1

# 修改250行，去掉#，设置neo4j-shell端口，端口可以自定义，只要不和其他端口冲突就行
dbms.shell.port=1337

# 修改254行，设置neo4j可读可写
dbms.read_only=false

```

## 8. 修改neo4j可视化界面的超管用户密码]

在控制台输入`:server change-password` 进行修改

键入原密码及新密码，即可修改

注意冒号

## 9 . window版本的其他安装方式:

neo4j 在window平台有一个desktop 版本，实现了多实例创建图数据库的应用，有需要可以直接安装，个人直接下载window的Bin包进行单机的部署。

# 如何安装neo4j社区版本（免费）（linux - CenterOs7）

重复的内容请查看`window`安装方式，linux 的安装相对更加简单一些。

（1）准备`**neo4j-community-3.5.12-unix.tar.gz.gz**` ，使用目前最新的版本

> Linux:
>
> 链接：https://pan.baidu.com/s/1ljzS5DIYo5n9fCIzKkAMiw 
> 提取码：bnrf 

（2）解压放入到linux相应位置

（3）确保当前环境变量存在JDK，版本不能低于`JDK1.8`

（4）同样由于安全配置的原因，需要进入客户端配置一次用户名和密码，因为linux没有GUI，在`neo4j.conf`需要开启远程访问：

> #dbms.connectors.default_listen_address=0.0.0.0
>
> 把#拿掉就可以进行远程访问了

（5）请参考window对于用户名和密码进行自定义

（6）如果忘记了GUI页面的用户名和密码，可以使用删除db的方式对于图数据库进行重置

# 总结：

1. 介绍了Neo4J的基本理念，已经我为什么要使用到neo4j 这个库
2. Neo4j在linux上和windows上的安装，注意如果要用到项目上，请注意使用开源的社区版，企业版提供更多的功能以及更好性能，同时官方提供技术支持，商用版本需要授权
3. 下一篇文章将对neo4j 进行扩展

内容篇幅较长，感谢观看！希望能对读者有所帮助，如果对于博客有任何建议或者意见，欢迎讨论，如果文章内容有误，可以直接私信或者在评论区留言，我会及时答复并且修复

