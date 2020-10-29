---
title: RocketMq消息队列中间件安装(linux)
subtitle: RocketMq消息队列中间件安装(linux)
author: lazytime
url_suffix: note23
tags:
  - RocketMq
categories:
  - 消息中间件
keywords: RocketMq消息队列中间件安装(linux)
description: 消息中间件
copyright: true
date: 2020-08-29 22:57:53
---

# 安装

## 1. 下载maven的tar.gz包

## 2. 设置/etc/profile文件加入MAVEN_HOME

> ## 安装工具
>
> ## yum install curl-devel expat-devel gettext-devel

```
> unzip rocketmq-all-4.3.0-source-release.zip
> cd rocketmq-all-4.3.0/
> mvn -Prelease-all -DskipTests clean install -U
> cd distribution/target/apache-rocketmq
```

## 3.启动

```
> nohup sh bin/mqnamesrv &
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
  
   > nohup sh bin/mqbroker -n localhost:9876 &
  > tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
```

## 4.启动消费者

```
> export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

#### 5.关闭

```
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

# 可视化工具下载与安装

RoekerMQ4.x可视化控制台讲解

```
1、下载 https://github.com/apache/rocketmq-externals
	2、编译打包  mvn clean package -Dmaven.test.skip=true
	3、target目录 通过java -jar的方式运行
	
	4、无法连接获取broker信息
		1）修改配置文件,名称路由地址为 namesrvAddr，例如我本机为
		2）src/main/resources/application.properties
			rocketmq.config.namesrvAddr=192.168.0.101:9876
	
	5、默认端口 localhost:8080
	
	6、注意：
		在阿里云，腾讯云或者虚拟机，记得检查端口号和防火墙是否启动
```