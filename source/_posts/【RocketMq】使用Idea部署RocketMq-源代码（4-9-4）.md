---
title: 【RocketMq】使用Idea部署RocketMq 源代码（4.9.4）
subtitle: IDEA的部署和使用
author: lazytime
url_suffix: rocketjianhua
tags:
  - RocketMq
categories:
  - RocketMq
keywords: RocketMq部署和简化
description: 部署和使用RocketMq
copyright: true
date: 2023-03-11 07:24:00
---
#rocketmq  

# 一、介绍

笔记为主，Idea部署RocketMq的简化流程。

https://github.com/apache/rocketmq


# 二、提示
## 2.1 IDEA版本
个人使用的Idea版本。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826144547.png)

<!-- more -->

## 2.2 RocketMq源码版本

需要注意下载源代码之前，先检查一下自己的java版本，最低要求JDK1.8以上。

个人拉取的版本为 **4.9.4**，因为时效性未来版本有些代码可能会被改进，所以要注意版本问题。

```java
java version "1.8.0_341"
Java(TM) SE Runtime Environment (build 1.8.0_341-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.341-b10, mixed mode)
```

拉源码，这一步不需要过多介绍，直接点击主页右上角即可。

> 这里从官方fork 了一下项目

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826151618.png)

github 现在墙越来越厚，拉代码经常失败，没办法只能再套一层，用 gitee fork了一遍，双层套娃属于是。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826152001.png)

最后终于成功拉代码到本地（真不容易）。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826151917.png)

# 三、操作步骤

## 3.1 下载源代码

个人fork版本（4.9.9）：[https://gitee.com/lazyTimes/rocketmq-adong.git](https://gitee.com/lazyTimes/rocketmq-adong.git)

官方gituhb（最新版本）：[https://github.com/apache/rocketmq.git](https://github.com/apache/rocketmq.git)


**源码目录结构**：

-   **broker**: broker 模块（broke 启动进程）
-   **client** ：消息客户端，包含消息生产者、消息消费者相关类
-   **common** ：公共包
-   **dev** ：开发者信息（非源代码）
-   **distribution** ：部署实例文件夹（非源代码）
-   **example**: RocketMQ 例代码
-   **filter** ：消息过滤相关基础类
-   **filtersrv**：消息过滤服务器实现相关类（Filter启动进程）
-   **logappender**：日志实现相关类
-   **namesrv**：NameServer实现相关类（NameServer启动进程）
-   **openmessageing**：消息开放标准
-   **remoting**：远程通信模块，基于Netty
-   **srcutil**：服务工具类
-   **store**：消息存储实现相关类
-   **style**：checkstyle相关实现
-   **test**：测试相关类
-   **tools**：工具类，监控命令相关实现类

## 3.2 检查Maven配置

拉下来之后不要急着调配置，我们先检查idea里面的Maven配置是否被重置，如果被重置了就改回来，另外最好检查一下当前使用的项目编译版本， IDEA 经常会用内置的JDK版本对项目编译，个人也因为这个问题浪费了不少时间。

总而言之，拉下项目做下面三件事：
- 编译，检查是否报错
- 检查JDK版本
- 检查Maven配置，以及Maven所用JDK打包版本（在setting里可以看到）。
- 反复检查，确保无误。

## 3.3 设置NameServ

首先是设置命名服务，这是Producer发送消息必要服务之一。

这一步核心步骤就两个：
- 启动参数设置 `ROCKETMQ_HOME` ，如果不会设置就直接项目的根路径，但还是建议设置一个单独目录里面方便后续观察。
- 设置`logback_namesrv.xml`，它所在的路径必须跟 `ROCKETMQ_HOME` 相同，否则 `NameServ` 启动会报错。

设置`ROCKETMQ`运行主目录，Mac系统和Window设置出来的路径不一样，个人使用Windows作为案例，所以设置的路径为：`E:\adongstack\project\selfUp\rocketmq-adong`

1. 接着我们进入 NameSer 这个项目，直接跑`org.apache.rocketmq.namesrv.NamesrvStartup#main()`方法，不出意外会给一个提示：

> Please set the ROCKETMQ_HOME variable in your environment to match the location of the RocketMQ installation

日志非常简洁明了，要求设置 `ROCKETMQ_HOME`，但是要设置在哪里，怎么设置？

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826155828.png)


2. 运行过一次`org.apache.rocketmq.namesrv.NamesrvStartup#main()`方法之后，Idea会有历史启动记录保留，这时候可以对于启动参数进行配置：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826160028.png)

3. 点击启动参数Env.....，然后在里面配置前文提到的`ROCKETMQ_HOME`，这里个人设置了独立路径，建议读者尝试的时候用一个单独的空文件目录，方便后期查找。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826160215.png)

4. 注意新版本IDEA的按钮位置变了，点击“+”号之后，填充参数。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826160352.png)

5. 确认之后，会发现下面的效果。然后继续运行`Main()`方法。

![路径配置结果](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826160453.png)

但是很遗憾，还是报错，日志提示需要`logback_namesrv.xml`，实际上就是logback的日志配置文件，我们可以从源代码的`E:\adongstack\project\selfUp\rocketmq-adong\distribution\conf\logback_namesrv.xml`（个人路径为例）找到这个文件的配置。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826164323.png)

6. 因为不知道文件放在哪里，我们尝试把项目的`logback_namesrv.xml`配置放到下面这个位置，看看什么效果。

> logback的配置这里就不介绍了，可以自行查阅官方文档或者上网查查资料了解配置含义。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826164712.png)

然而这种配置方式**是错的**，实际上我们要移动配置到 **ROCKETMQ_HOME** 所在目录中。日志报错会引导我们进行正确改进。

7. 最后尝试执行`org.apache.rocketmq.namesrv.NamesrvStartup#main()`方法，成功！打印内容如下。

```console
	The Name Server boot success. serializeType=JSON
```

## 3.4 设置Broker

Broker负责持久化队列消息，以及从NameServ 中寻找路由信息，Producer 借助NameServ 找到Broker之后会推送消息，Broker成功收到消息后通知Producer消息发送成功。

这一步设置比较关键，仔细观察配置。

设置Broker主要考虑两点，第一点是`broker.conf`配置，第二点是配置Broker的logback的xml日志文件。

接着我们设置 Broker ，Broker的启动入口在`src/main/java/org/apache/rocketmq/broker/BrokerStartup.java`，一上来我们先不急着改启动环境的参数，而是先把 Broker 的关键配置改了，**主要设置 NameServer 的地址、Broker 的名称等相关属性**，这些依赖配置文件`broker.conf`。

首先我们要先查自己本地回环网卡的IP地址（就是s你上网那张网卡的IP），比如我的IP是`192.168.0.6`。

有了IP之后，我们可以先备份一份`distribution/conf/broker.conf`文件，然后修改里面的配置：

> 也可以直接复制一份`distribution/conf/broker.conf`配置到 **ROCKETMQ_HOME**然后再进行改动，这里仅仅是个人习惯问题，方便日后直接在项目里面立马可以看到自己定义的配置。

`broker.conf `官方给的默认配置：

```
brokerClusterName = DefaultCluster  
brokerName = broker-a  
brokerId = 0  
deleteWhen = 04  
fileReservedTime = 48  
brokerRole = ASYNC_MASTER  
flushDiskType = ASYNC_FLUSH
```

`broker.conf`修改之后的配置，注意有新增内容，个人命名为`broker-back.conf`：

> 这些配置都可以从源代码的配置文件里面找到对应的参数属性。

```
# 使用如下配置文件  
brokerClusterName = DefaultCluster  
brokerName = broker-a  
brokerId = 0  
deleteWhen = 04  
fileReservedTime = 48  
brokerRole = ASYNC_MASTER  
flushDiskType = ASYNC_FLUSH  
storePathRootDir=E:\\adongstack\\project\\selfUp\\rocketmq-adong\\tmp\\run-dev\\store  
storePathCommitLog=E:\\adongstack\\project\\selfUp\\rocketmq-adong\\tmp\\run-dev\\store\\commitlog  
# 重点内容
namesrvAddr=127.0.0.1:9876  
brokerIP1=192.168.0.6  
brokerIP2=192.168.0.6  
autoCreateTopicEnable=true
```

> 为啥是 store 和 store\commitlog，一个是Broker的持久化仓库地址，一个是提交日志存储地址，这里建议分开，RocketMq的commitlog使用的是追加写入的方式。

注意网上有一些教程的配置文件内容可能比这份文件要多一些，其实只是更加细化的配置而已，这些简单的配置足够我们调试最简单的源码。

修改完成之后，我们同样把**distribution/conf/logback_brokerxml** 以及 **broker.conf**拷贝到 `ROCKETMQ_HOME`当中。经过这么一番操作配置，最终结果如下：

> 这里把所有的配置放到一个文件夹里面，方便管理，截图所属文件的根路径是`E:\adongstack\project\selfUp\rocketmq-adong\tmp\run-dev`


![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826173454.png)

解剖路径如下：
```properties
# `E:\adongstack\project\selfUp\rocketmq-adong\tmp\run-dev`//.....
- store
	- commitlog
- broker.conf
- logback_broker.xml
- logback_namesrv.xml
```

最后，我们在`cc`的启动参数进行如下配置：

`-c 启动参数` 对应conf配置文件。
```java
-c E:\\adongstack\\project\\selfUp\\rocketmq-adong\\tmp\\run-dev\\broker.conf
```

![BrokerStartup启动参数](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826205432.png)

最后再运行一遍`BrokerStartup`的main方法，出现下面的内容说明配置成功：

```
The broker[broker-a, 192.168.0.103:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```

## 3.5 运行测试

首先我们把前面两个小节配置的NameServ和Broker的配置进行启动：

Broker 启动打印：
```java
The broker[broker-a, 127.0.0.1:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```

NameServ 启动打印：
```java
The Name Server boot success. serializeType=JSON
```

接着是使用Producer做测试，这里使用的是`org.apache.rocketmq.example.quickstart.Producer#main()`方法，注意这个案例是不能够直接拿来用的，因为我们使用debug模式，**需要放开一行注释代码**。

因为上面的所有配置都是使用官方推荐的默认配置，所以需要只需要放开这一段注释，默认连接的NameServ 端口是 9876。

> // Uncomment the following line while debugging, namesrvAddr should be set to your local address  
  // 调试时取消注释以下行，namesrvAddr 应设置为您的本地地址，读者第一次看到这里应该是注释过的 
  producer.setNamesrvAddr(DEFAULT_NAMESRVADDR);

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20220826215646.png)

Producer的最终打印结果如下：

```java
SendResult [sendStatus=SEND_OK, msgId=7F000001858818B4AAC28569CA6C0000, offsetMsgId=7F00000100002A9F00000000000000EF, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=0], queueOffset=1]
21:45:02.846 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[127.0.0.1:9876] result: true
21:45:02.848 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[127.0.0.1:10911] result: true
```

# 小结
IDEA 部署源代码整体流程比较简单，比较麻烦的反而是一些运行参数的配置上。另外github现在连接十分不稳定，建议用国内的一些代码管理网站进行同步，gitee、coding 都可以。

需要注意进行测试的时候一定要把注释放开，否则会一直出现Producer连不上的报错异常。

