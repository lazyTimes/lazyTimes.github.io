---
title: Jmeter压力测试回顾(三)
subtitle: 压力测试回顾第三部分
url_suffix: jmeter-test3
author: lazytime
tags:
  - 测试
categories:
  - 测试
keywords: 压力测试回顾第三部分
description: 压力测试回顾第三部分
copyright: true
date: 2021-01-18 23:35:12
---

# 回顾Jmeter压力测试（三）

# GUI实战部分

通过菜单了解Jmeter大致的内容之后，我们依靠实战来更深入的理解各种功能

<!-- more -->

## 线程组的基本使用：

准备部分：

1. 准备一个springBoot项目，或者找一个可以访问的接口，当然最好不要访问一些外网IP，容易误认为攻击封IP
2. 比如个人写了一个简单的DEMO：

```java
@RestController
@RequestMapping("/jmeter")
public class JmeterTest {

    @RequestMapping("/runTest1")
    public Object testThread(){
        return "hello-world";
    }
}

```



1 添加->threads->线程组（控制总体并发）

案例：如下图所示，添加一个20个线程，循环1次的线程组，在2秒内完成

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109174815.png)

+ 线程数：虚拟用户数。一个虚拟用户占用一个进程或线程
+ 准备时长（Ramp-Up Period(in seconds)）：全部线程启动的时长，比如100个线程，20秒，则表示20秒内100个线程都要启动完成，每秒启动5个线程
+ 循环次数：每个线程发送的次数，假如值为5，100个线程，则会发送500次请求，可以勾选永远循环	



2 线程组 -> 添加-> Sampler(采样器) -> Http请求

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109193233.png)

一个线程组下面可以增加几个Sampler

+ 名称：采样器名称
+ 注释：对这个采样器的描述
+ web服务器：
  + 默认协议是http
  + 默认端口是80
  + 服务器名称或IP ：请求的目标服务器名称或IP地址
+ 路径：服务器URL		
+ **Use multipart/from-data for HTTP POST** ：当发送POST请求时，使用`Use multipart/from-data方法发送，默认不选中。

> 吐槽：感觉中间的选框太小了



3 请求-查看测试结果

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109193752.png)

如之前介绍菜单所说，还需要增加一个监听器，收集请求的最终结果。

操作如下：线程组->添加->监听器->察看结果树

如上图所示，最终的测试结果都是正确，两秒内开启20个线程并发访问，压力还是比较小的。



## 断言的基本使用

上面的案例是一个最简单的线程组的使用，有点类似postMan请求接口。下面了解一下断言是如何使用的。

1. 增加断言: 线程组 -> 添加 -> 断言 -> 响应断言

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109194054.png)

下面增加一个相应的断言，响应断言里面的选择内容如下：

测试字段：根据测试字段匹配结果，

中间：apply to(应用范围):
	`Main sample only:` 仅当前父取样器 进行断言，一般一个请求，如果发一个请求会触发多个，否则就使用`sub sample`（比较少用） 

匹配规则：一些逻辑的匹配规则，比如包含，匹配，相等

这一块的组合形式比较丰富，下面简单举几个例子：

+ 响应文本：即响应的数据，比如json等文本
+ 响应代码：http的响应状态码，比如200，302，404这些
+ 响应信息：http响应代码对应的响应信息，例如：OK, Found
+ Response Header: 响应头

模式匹配规则：

+ 包括：包含在里面就成功
+ 匹配：响应内容完全匹配，不区分大小写
+ equals：完全匹配，区分大小写

断言结果监听器: 线程组-> 添加 -> 监听器 -> 断言结果

里面的内容是sampler采样器的名称
断言失败，查看结果树任务结果颜色标红(通过结果数里面双击不通过的记录，可以看到错误信息)

2、响应断言案例：

给出一个请求的信息内容：

```
Thread Name:20线程2秒内容完成 1-1
Sample Start:2021-01-09 19:57:43 CST
Load time:1
Connect Time:0
Latency:1
Size in bytes:173
Sent bytes:134
Headers size in bytes:162
Body size in bytes:11
Sample Count:1
Error Count:0
Data type ("text"|"bin"|""):text
Response code:200
Response message:
```

下面创建一个响应断言：

我们指定响应文本为：Hello-world，测试字段为响应文本，在下方输入`hello-world`用于增加断言的信息。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109200104.png)

在查看結果树里面就可以看到对应的信息内容了

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110135332.png)

大小断言：可以断言响应请求里面的size大小，单位为`Byte`

断言的功能相对比较简单，这里仅仅列出了一些简单案例

## 聚合报告分析

操作方式：新增聚合报告：线程组->添加->监听器->聚合报告（Aggregate Report）

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110135654.png)

介绍一下英文版的内容对应列的含义

+ lable: sampler的名称
+ Samples: 一共发出去多少请求,例如10个用户，循环10次，则是 100
+ Average: 平均响应时间
+ Median: 中位数，也就是 50％ 用户的响应时间
+ 90% Line : 90％ 用户的响应不会超过该时间 （90% of the samples took no more than this time. The remaining samples at least as long as this）
+ 95% Line : 95％ 用户的响应不会超过该时间
+ 99% Line : 99％ 用户的响应不会超过该时间
+ min : 最小响应时间
+ max : 最大响应时间
+ Error%：错误的请求的数量/请求的总数
+ **Throughput： 吞吐量——默认情况下表示每秒完成的请求数（Request per Second) 可类比为qps
  		KB/Sec: 每秒接收数据量**

新增聚合报告之后，就可以在聚合报告里面看到对应的结果，注意聚合报告的内容都是累加的



## 用户自定义变量

如果想要设置一些常用的变量配置放到当前的线程组里面，可以使用如下的方式

作用：

很多变量在全局中都有使用，或者测试数据更改，可以在一处定义，四处使用(比如服务器地址)

操作方法：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110165045.png)

线程组->add -> Config Element(配置原件)-> User Definde Variable（用户定义的变量）

如上图配置完成之后，引用方式如下：${XXX}，在接口中变量中使用，如下所示：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110165240.png)

##  CSV可变参数

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110170707.png)

1、线程组->add -> Config Element(配置原件)-> CSV data set config (CSV数据文件设置)

我们在电脑的任意位置配置txt文件，配置方式如下：

新建一个txt文件，同时注意使用utf-8的编码进行保存。

2、在读取的配置文件里面，同时使用多个自定义参数

```txt
username,password,gender
zhangsan,test001,1
lisi,test002,2
wanglaowu,test003,2
laoliu,test004,1
```

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110183037.png)

接着我们可以唉csv可变参数里面按照要求进行配置：

+ 文件名：csv文件的名字
+ 文件编码：当前文件的编码格式
+ 变量名称：对应标头，用逗号分隔
+ 忽略首行：蛀牙要设置变量名称的时候才会生效
+ 分隔符：默认分隔符为逗号
+ 是否带有引号：(未验证)
+ 遇到文件结束符之后再次循环：意味着是否需要重复的读取文件的内容
+ 遇到文件结束符终止线程：只读一遍文件
+ 线程共享模式：默认应用当前**所有**线程组

## JDBC request压测Mysql

### 简单应用入门：

在正式进行使用之前，需要随意建立一个数据库或表。

```mysql
CREATE TABLE `admin`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `account` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `password` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `power` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 52 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

```

使用Jdbc进行压力测试，需要依赖jar包实现，不同的版本jar包压测的结果可能会有出入

1. Thread Group -> add -> sampler -> jdbc request

这时候进行请求是肯定无效的，我们只是建立的一个采样器：

2. jar包添加  mysql-connector-java-5.1.30.jar 

我们需要在配置元件里面，对于个人的JDBC进行配置

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110191818.png)

3. JDBC connection Configuration 配置

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110195206.png)

Max number of connections：最大连接数

Time-between-eviction-runs-millis：指定空闲连接检查、废弃连接清理、空闲连接池大小调整之间的操作时间间隔

variable name for created pool：创建池的变量名

Auto commit：是否自动提交

Transaction Isolation：事务隔离级别（注意Mysql默认级别为读已提交）

Preinit Pool：预初始化池

Test while idle：空闲线程测试

Soft Min Evictable Idle Time：软最小可收回空闲时间

Validation Query：验证查询

最下面的部分为JDBC连接，最后一项是连接的参数配置。

DataBase URL : 数据库连接地址 jdbc:mysql://127.0.0.1:3306/blog
JDBC Driver Class : 数据库驱动，选择对应的mysql
username：数据库用户名
password：数据库密码

4. 将`mysql-connector-java.xxx.jar`拷贝到jmeter的classpath目录：（核心步骤）

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110200116.png)

5. 注意一下配置一定要准确，否则请求会抛出错误，请求失败也不要慌，多看看返回的提示信息，个人在实验的过程中也不是一次成功的：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110200938.png)

6. 发起测试请求：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110201135.png)

下面的数据为随便录入的一些数据，可以参考自己的测试结果

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210110201155.png)

### 查询参数：

GUI在查询参数这一部分不是十分好，下面说下如果需要使用？这一类形式替换要如何处理

如果使用占位符，在jmeter的GUI界面，需要使用括号进行替换，（1）代表取第一个参数。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210111001343.png)

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210111001434.png)

下面为填入的内容介绍：

**Variable name：**这里写入数据库连接池的名字（和JDBC Connection Configuration名字保持一致 ）
**Query：**里面填入查询数据库数据的SQL语句（填写的SQL语句末尾不要加“；”）
**parameter valus：**数据的参数值
**parameter types：**数据的参数类型
**cariable names：**保存SQL语句返回结果的变量名
**result cariable name：**创建一个对象变量，保存所有返回结果
**query timeout：**查询超时时间
**handle result set：**定义如何处理由callable statements语句返回的结果

## 分布式压测

官网教程： http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html

+ 普通压测：单台机可以对目标机器产生的压力比较小，受限因素包括CPU，网络，IO等
+ 分布式压测：利用多台机器向目标机器产生压力，模拟几万用户并发访问

由于分布式的压测一般对于服务器的配置要求较高，个人一个弱鸡服务器就不献丑了，这里推荐一篇分布式压测文章推荐：

简书文章：https://www.jianshu.com/p/5c04792c31b6

官方文档：https://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html

Jmeter分布式压测原理(了解)

1、总控机器的节点master，其他产生压力的机器叫“肉鸡” server

2、master会把压测脚本发送到 server上面

3、执行的时候，server上只需要把jmeter-server打开就可以了，不用启动jmeter

4、结束后，server会把压测数据回传给master,然后master汇总输出报告