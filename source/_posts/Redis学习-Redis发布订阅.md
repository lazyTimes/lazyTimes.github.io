---
title: Redis学习 - Redis发布订阅
subtitle: redis发布订阅的一些应用以及一些命令的学习
url_suffix: note
author: lazytime
tags:
  - redis
categories:
  - redis专栏
keywords: redis
description: redis发布订阅的一些应用以及一些命令的学习
copyright: true
date: 2020-11-21 22:52:22
---

消息通信模式，发送者发送消息，接受者接受消息，微信，微博，关注系统

redis客户端可以订阅任意数量的频道

订阅/发布消息：

1. 消息发送者
2. 频道
3. 消息内容
4. 消息接受者

<!-- more -->

## 消息发布的原理

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201118223756.png)

订阅/发布消息需要的四个必要对象：

1. 消息发送者 publisher
2. 频道 - channel
3. 消息内容 - channel msg
4. 消息接受者 - subscriber

## 如何订阅频道

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201118224115.png)

使用**subscribe** 命令发送消息，订阅消息会显示 1）2）3）,同时显示订阅的序号，从1开始，这时候命令行会阻塞等待消息传递

```
127.0.0.1:16379> subscribe first second
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "first"
3) (integer) 1
1) "subscribe"
2) "second"
3) (integer) 2
```

## 发送消息

redis设计了 **publish** 命令，用于订阅频道，发送消息会返回成功收到消息的数量，如果没有收到则为0

我们需要在新开一个窗口，输入如下命令

```
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ ./redis-cli -p 16379
127.0.0.1:16379> publish channel1 message
(integer) 1
127.0.0.1:16379> 

```



## 模式匹配

功能说明：允许客户端订阅某个模式的频道

本质：其实就是可以通过使用通配符的模式批量订阅一批频道

具体的命令如下：

```
127.0.0.1:16379> PSUBSCRIBE chanel-*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "chanel-*"
3) (integer) 1

```

如果订阅了一批频道，那么发送给这个频道的消息将被客户端接收到两次，只不过这两条消息的类型不同，一个是message类型，一个是**pmessage**类型，但其内容相同。

```
127.0.0.1:16379> PSUBSCRIBE chanel-*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "chanel-*"
3) (integer) 1
1) "pmessage"
2) "chanel-*"
3) "chanel-111"
4) "213213"
1) "pmessage"
2) "chanel-*"
3) "chanel-22"
4) "213213"
```

## 取消订阅

Redis采用**UNSUBSCRIBE**和**PUNSUBSCRIBE**命令取消订阅

## redis发布订阅原理实现

https://juejin.im/post/6844904186534952968#heading-7

### subscribe的实现

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201119134454.png)

1. 维护一个client 和 一个 server 结构体，都存储pubsub_patterns
2. client存储的是以hash表来实现的，用键值对的形式，键为键表示订阅的频道，值为空。
3. 而server存储的是改服务器当中所有频道以及订阅这个频道的客户端，也是字典类型。插入节点的时候键为频道，值为订阅的所有客户端组成的链表

### psubscribe

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201119225433.png)

1. 大体实现和subscribe类似，client维护的内容是相似的
2. 在server当中,表示该服务端的所有频道以及订阅频道客户端。插入节点使用的是键为频道，值为订阅了所有客户端组成的链表

### PUBLISH

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201119225849.png)

会在redis_server当中遍历所有的pubsub_channel中管理的所有频道，找到对应的频道之后链表遍历所有的客户端，将消息发给客户端。

## redis发布订阅的应用

用于监听命令和数据的变化，当redis通过发布订阅进行写操作的时候，会有两条消息，一条是del mykey，另一条是mykey del。一个表示空间变化，一个是频道的改变。可以用于进行消息的实时推送。

## redis和activeMQ的比较

（1）ActiveMQ支持多种消息协议，包括AMQP，MQTT，Stomp等，并且支持JMS规范，但Redis没有提供对这些协议的支持； 

（2）ActiveMQ提供持久化功能，但Redis无法对消息持久化存储，一旦消息被发送，如果没有订阅者接收，那么消息就会丢失； 

（3）ActiveMQ提供了消息传输保障，当客户端连接超时或事务回滚等情况发生时，消息会被重新发送给客户端，Redis没有提供消息传输保障。

>  ActiveMQ所提供的功能远比Redis发布订阅要复杂，毕竟Redis不是专门做发布订阅的，但是如果系统中已经有了Redis，并且需要基本的发布订阅功能，就没有必要再安装ActiveMQ了，因为可能ActiveMQ提供的功能大部分都用不到，而Redis的发布订阅机制就能满足需求。