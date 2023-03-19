---
title: 【RocketMq】Rocket 生产者相关知识点 - 消息发送者参数详解
subtitle: RocketMq生产者使用
author: lazytime
url_suffix: rocketshengchanzhe
tags:
  - RocketMq
categories:
  - RocketMq
description: rocketmq使用
copyright: true
date: 2023-03-11 07:27:49
---

#rocketmq

# 引言

首先注意本次讨论的RokcetMq源码版本为 **4.9.4**，距离5.0发布 的没有多久。

这一节针对RocketMq的生产者请求发送的部分细节进行阐述，主要包含了下面的内容：DefaultMQProducer 为生产者默认对象，这个对象继承自 ClientConfig，里面包含了请求者的通用配置，所以可以拆分为两个部分进行理解，第一部分为ClientConfig，第二部分为DefaultMQProducer。

# ClientConfig 部分

ClientConfig 定义了一些配置的获取方法，定义了命名空间等参数。无论是消息的发送者还是消费者都是通用的。

下面根据本次的版本的源代码介绍相关参数。

| 名称                            | 描述                                           | 参数类型      | 默认值                                                       | 有效值 | 重要性 |
| ------------------------------- | ---------------------------------------------- | ------------- | ------------------------------------------------------------ | ------ | ------ |
| namesrvAddr                     | NameServer的地址列表                           | String        | 从-D系统参数rocketmq.namesrv.addr或环境变量。NAMESRV_ADDR    |        |        |
| instanceName                    | 客户端实例名称                                 | String        | 从-D系统参数rocketmq.client.name获取，否则就是DEFAULT        |        |        |
| clientIP                        | 客户端IP                                       | String        | RemotingUtil.getLocalAddress()                               |        |        |
| namespace                       | 客户端命名空间                                 | String        |                                                              |        |        |
| accessChannel                   | 设置访问通道                                   | AccessChannel | LOCAL                                                        |        |        |
| clientCallbackExecutorThreads   | 客户端通信层接收到网络请求的时候，处理器的核数 | int           | Runtime.getRuntime().availableProcessors()                   |        |        |
| pollNameServerInterval          | 轮询从NameServer获取路由信息的时间间隔         | int           | 30000，单位毫秒                                              |        |        |
| heartbeatBrokerInterval         | 定期发送注册心跳到broker的间隔                 | int           | 30000，单位毫秒                                              |        |        |
| persistConsumerOffsetInterval   | 作用于Consumer，持久化消费进度的间隔           | int           | 默认值5000，单位毫秒                                         |        |        |
| pullTimeDelayMillsWhenException | 拉取消息出现异常的延迟时间设置                 | long          | 1000，单位毫秒                                               |        |        |
| unitName                        | 单位名称                                       | String        |                                                              |        |        |
| unitMode                        | 单位模式                                       | boolean       | false                                                        |        |        |
| vipChannelEnabled               | 是否启用vip netty通道以发送消息                | boolean       | 从-D com.rocketmq.sendMessageWithVIPChannel参数的值，若无则是true |        |        |
| useTLS                          | 是否使用安全传输。                             | boolean       | 从-D系统参数tls.enable获取，否则就是false                    |        |        |
| mqClientApiTimeout              | mq客户端api超时设置                            | int           | 3000，单位毫秒                                               |        |        |
| language                        | 客户端实现语言                                 | LanguageCode  | LanguageCode.*JAVA*                                          |        |        |

<!-- more -->

## namesrvAddr

NameServer 的地址列表。

## clientIp

```java
private String clientIP = RemotingUtil.getLocalAddress();
```

从代码中可以看到，使用`RemotingUtil#getLocalAddress` 获取IP信息，在当前版本中默认返回不是`127.0`或者`192.168`开头的 IPV4地址，否则尝试获取IPV6的地址，如果都找不到就用LocalHost地址。

## instanceName

```java
private String instanceName = System.getProperty("rocketmq.client.name", "DEFAULT");
```

`instanceName`主要获取当前默认的系统参数客户端实例名称，它是客户端标识 CID 的组成部分


## unitName 单元名称

也是CID的组成部分之一，如果获取 NameServer 的地址是通过 URL 进行动态更新的话，会通过这个单元名称进行附加，用来区分不同的NameServer地址服务。

## clientCallbackExecutorThreads 回调线程池数量

表示public回调线程池的数量，默认为CPU的核数，通常这个值直接根据JVM获取的结果为基准即可。

```java
private int clientCallbackExecutorThreads = Runtime.getRuntime().availableProcessors();
```
## namespace 命名空间

4.5.1 之后才加入的新机制。主要适用场景为全链路压测的时候可以利用不同的命名空间划分出真实消息和压测消息，使得线上业务正常执行的情况下同步处理测试流程。

## pollNameServerInterval NameServer同步间隔

生产者客户端默认每隔出30S向NameServer 更新Topic的相关信息，注意这个参数在消费端同样存在相同的配置，这个配置通常不建议修改。

```java
/**  
 * Pulling topic information interval from the named server */
   private int pollNameServerInterval = 1000 * 30;
```

## heartbeatBrokerInterval Broker心跳间隔

客户端向 Broker 发送心跳包的时间间隔，默认为 30s，不建议修改该值。

```java
/**  
 * Heartbeat interval in microseconds with message broker */
   private int heartbeatBrokerInterval = 1000 * 30;
```

## persistConsumerOffsetInterval

客户端持久化消息消费进度的间隔，默认为 5s，该值不建议修改。

```java
/**  
 * Offset persistent interval for consumer */
   private int persistConsumerOffsetInterval = 1000 * 5;
```

# DefaultMQProducer 部分

这部分定义了日志和常见的使用消息队列方法，注意在类的开头定义了一个 **transient** 变量执行内部的保护方法。

官方文档中极少DefaultMQProducer配置如下：

| 名称                             | 描述                                                         | 参数类型        | 默认值                                     | 有效值 | 重要性 |
| -------------------------------- | ------------------------------------------------------------ | --------------- | ------------------------------------------ | ------ | ------ |
| producerGroup                    | 生产组的名称，一类Producer的标识                             | String          | DEFAULT_PRODUCER                           |        |        |
| createTopicKey                   | 发送消息的时候，如果没有找到topic，若想自动创建该topic，需要一个key topic，这个值即是key topic的值 | String          | TopicValidator.AUTO_CREATE_TOPIC_KEY_TOPIC |        |        |
| defaultTopicQueueNums            | 自动创建topic的话，默认queue数量是多少                       | int             | 4                                          |        |        |
| sendMsgTimeout                   | 默认的发送超时时间                                           | int             | 3000，单位毫秒                             |        |        |
| compressMsgBodyOverHowmuc        | 消息body需要压缩的阈值                                       | int             | 1024 * 4，4K                               |        |        |
| retryTimesWhenSendFailed         | 同步发送失败的话，rocketmq内部重试多少次                     | int             | 2                                          |        |        |
| retryTimesWhenSendAsyncFailed    | 异步发送失败的话，rocketmq内部重试多少次                     | int             | 2                                          |        |        |
| retryAnotherBrokerWhenNotStoreOK | 发送的结果如果不是SEND_OK状态，是否当作失败处理而尝试重发    | boolean         | false                                      |        |        |
| maxMessageSize                   | 客户端验证，允许发送的最大消息体大小                         | int             | 1024 *1024* 4，4M                          |        |        |
| traceDispatcher                  | 异步传输数据接口                                             | TraceDispatcher | null                                       |        |        |


## DefaultMQProducerImpl 内部对象

`defaultMQProducerImpl` 比较意思，因为此对象是 `DefaultMQProducerImpl` 整个实现类的实际调用者，这里用了受保护的内部对象完成所有方法调用，用final是规避旧版本多个线程初始化对象非原子性的问题，同时保证持有的内部对象不可变。

```java
/**  
 * Wrapping internal implementations for virtually all methods presented in this class. */
protected final transient DefaultMQProducerImpl defaultMQProducerImpl;
```

> 为什么这里要用 transient？
> transient 关键字确保对象被序列化之后不会泄漏 DefaultMQProducerImpl 对象。

## InternalLogger 日志对象

接着是日志对象，日志对象 InternalLogger 如下定义，内部实现比较简单，基本是一些info和debug日志打印。

```java
InternalLogger log = ClientLogger.getLog()
```

客户端日志的实现类存储路径时是：`${user.home}/logs/rocketmqlogs/rocketmq_client.log`，这个路径的获取细节在`org.apache.rocketmq.client.log.ClientLogger#createClientAppender`可以看到有关细节。使用`System.getProperty("user.home")`获取的路径在Unix系统中相当于用户的主目录。

> user.home 如果是 xxx 则是 /usr/home/xxx 为开始，比如个人的Mac电脑最终的存放地址为：`/Users/zxd/logs/rocketmqlogs/rocketmq_client.log`。

## producerGroup 消息组

表示发送者所属组定义如下，根据注释可以得知，gropu 可以实现生产者实例的聚合，主要用在事务的的时候需要使用到，而如果是非事务的消息，每一个进程都是唯一的，彼此没有关联。

有关事务的内容涉及需要用到Broker反查机制，这里不做过多牵扯，继续介绍。

```java
/**  
 * Producer group conceptually aggregates all producer instances of exactly same role, which is particularly * important when transactional messages are involved. </p>  
 *  
 * For non-transactional messages, it does not matter as long as it's unique per process. </p>  
 *  
 * See <a href="http://rocketmq.apache.org/docs/core-concept/">core concepts</a> for more discussion.  
 */
 private String producerGroup;
```

我们可以通过相关命令或者可视化工具查看发送者所属组的状态。注意默认的主题队列数量，RocketMq默认设置为4。

这里用了volatile保证多线程对于主题队列的数量时可见的，多个生产者实例观察的数量是一致的。

```java
/**  
 * Number of queues to create per default topic. */private volatile int defaultTopicQueueNums = 4;
```

## sendMsgTimeout 消息发送默认超时时间

消息默认发送的超时时间为3秒，

注意的是在 RocketMQ 4.3.0 版本之前由于存在重试机制，程序设置的设计为单次重试的超时时间，即如果设置重试次数为 3 次，则 `DefaultMQProducer#send` 方法可能会超过 9s 才返回。

```java
/**  
 * Timeout for sending messages. */
   private int sendMsgTimeout = 3000;
```

主要的改动点在`org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl` 这个对象里面

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202209280724815.png)

修复的方式比较简单粗暴，是增加一个纳秒值进行计算 ，如果请求时间超过发送请求的时间太久就抛出异常。下一次请求对应的扣除掉本次耗费的时间再进行重试，如果重试超过的总时间超过超时时间也同样抛出异常。

这就意味着如果超时次数设置10次，可能不到10次就会因为超时时间的判断抛出异常信息。

```java
long costTimeAsync = System.currentTimeMillis() - beginStartTime;  
if (timeout < costTimeAsync) {  
    throw new RemotingTooMuchRequestException("sendKernelImpl call timeout");  
}
```

##  compressMsgBodyOverHowmuch 压缩阈值

默认情况下，如果消息的长度超过4K，那么RocketMq默认会对于消息开启压缩，虽然会增加CPU的性能损耗，但是可以有效减少网络方便的开销。

```java
/**  
 * Compress message body threshold, namely, message body larger than 4k will be compressed on default. */
 // 压缩消息体阈值，即默认压缩大于4k的消息体。
   private int compressMsgBodyOverHowmuch = 1024 * 4;
```

```java
private boolean tryToCompressMessage(final Message msg) {  
	// 批量数据目前不支持压缩
    if (msg instanceof MessageBatch) {  
        //batch does not support compressing right now  
        return false;  
    }  
    byte[] body = msg.getBody();  
    if (body != null) {  
        if (body.length >= this.defaultMQProducer.getCompressMsgBodyOverHowmuch()) {  
            try {  
	            // 压缩之后的数据
                byte[] data = compressor.compress(body, compressLevel);  
                if (data != null) {  
                    msg.setBody(data);  
                    return true;                }  
            } catch (IOException e) {  
                log.error("tryToCompressMessage exception", e);  
                log.warn(msg.toString());  
            }  
        }  
    }  
  
    return false;  
}
```

## retryTimesWhenSendFailed 同步失败重试

同步消息发送重试次数。RocketMQ 客户端内部在消息发送失败时默认会重试 2 次。该参数与 `sendMsgTimeout` 联合生效，但是需要注意这个参数在SYNC模式下才会重试2次，如果是其他模式则默认是一次失败不再进行重试。

在SYNC模式只重试一次可以看下面代码：

```java
int timesTotal = communicationMode == CommunicationMode.SYNC ? 1 + this.defaultMQProducer.getRetryTimesWhenSendFailed() : 1;
```

## retryTimesWhenSendAsyncFailed 异步消息重试

见名知意，发送重试次数，如果是同步则默认为 2，即重试 2 次，一共有 3 次机会，如果是异步则只有一次机会，但是是写死1在判断处。

关键的代码在`org.apache.rocketmq.client.impl.MQClientAPIImpl#onExceptionImpl` 这个参数巨多的方法当中，简单判断当前的异步消息总的重试次数，如果重试多次超过次数则通过sendCallback回调发送异常。


```java
/**  
 * Maximum number of retry to perform internally before claiming sending failure in synchronous mode. </p>  
 *  
 * This may potentially cause message duplication which is up to application developers to resolve. */
   private int retryTimesWhenSendFailed = 2;
```

## retryAnotherBrokerWhenNotStoreOK 失败向其他Broker重试

根据方法的本意按照道理来说如果客户端收到的结果不是 SEND_OK，应该直接向另外一个 Broker 重试，但根据代码分析目前这个参数并不能按预期运作，官方一致也没有关注过这个问题。

```java
  
/**  
 * Maximum number of retry to perform internally before claiming sending failure in asynchronous mode. </p>  
 *  
 * This may potentially cause message duplication which is up to application developers to resolve. */
   private int retryTimesWhenSendAsyncFailed = 2;
```

## maxMessageSize 最大消息体

允许发送的最大消息体，默认为 4M，具体可以看下面的判断，注意Broker也有 maxMessageSize 这个参数的设置，故客户端的设置不能超过服务端的配置：

客户端的发送限制如下：

```java
/**  
 * Maximum allowed message body size in bytes. */
   private int maxMessageSize = 1024 * 1024 * 4; // 4M

...

if (msg.getBody().length > defaultMQProducer.getMaxMessageSize()) {  
    throw new MQClientException(ResponseCode.MESSAGE_ILLEGAL,  
        "the message body size over max value, MAX: " + defaultMQProducer.getMaxMessageSize());  
}
```

maxMessageSize 另一个使用地点是在RocketMq的轨迹消息长度判断中，不过这一块的代码在2022年的上半年被某位大神大改优化过，里面的优化代码比较值得学习，但是因为这一块牵扯的内容比较大部头需要先放放，我们看其他参数内容。

```java
// 轨迹消息中累计到3/4左右的时候就进行合并提交
if (currentMsgSize >= traceProducer.getMaxMessageSize() - 10 * 1000) {  
    List<TraceTransferBean> dataToSend = new ArrayList(traceTransferBeanList);  
    AsyncDataSendTask asyncDataSendTask = new AsyncDataSendTask(traceTopicName, regionId, dataToSend);  
    traceExecutor.submit(asyncDataSendTask);  
  
    this.clear();  
  
}
```

## sendLatencyFaultEnable 失败延迟规避

失败规避机制默认为false，它的含义是当Product向Broker发送消息失败之后，客户端的在内部重试的时候会规避掉上一次发送失败的Broker，并且一段时间内不会再向该Broker进行发送。

## notAvailableDuration 不可用延迟数组

不可用延迟数组，利用等比数列的时间发送消息，根据数组的设置在多少时间内不向Broker发送消息。从默认值可以看到这里是按照阶层的方式进行增长的。

```java
private long[] notAvailableDuration = {0L, 0L, 30000L, 60000L, 120000L, 180000L, 600000L};
```

## latencyMax  延迟最大值

设置消息发送的最大延迟级别，同样涉及了延迟推送机制。这里暂时略过。

```java
private long[] latencyMax = {50L, 100L, 550L, 1000L, 2000L, 3000L, 15000L};
```

# MqAdmin

定义了一些基础的规范接口，由于和我们平时写业务代码的Service Interface类似，这里不在过多展开介绍，而是简单罗列一些比较常用的接口：


```java

/**
-   String key：根据 key 查找 Broker，即新主题创建在哪些 Broker 上
-   String newTopic：主题名称
-   int queueNum：主题队列个数
-   int topicSysFlag：主题的系统参数
*/
void createTopic(String key, String newTopic, int queueNum, int topicSysFlag)
 
/**
	根据队列与时间戳，从消息消费队列中查找消息，返回消息的物理偏移量（在 commitlog 文件中的偏移量）。
	MessageQueue mq：消息消费队列
	long timestamp：时间戳
*/
long searchOffset(MessageQueue mq, long timestamp)

 /**
 查询消息消费队列当前最大的逻辑偏移量，在 consumequeue 文件中的偏移量。
 */
long maxOffset(final MessageQueue mq)
    
 /**
 查询消息消费队列当前最小的逻辑偏移量。
 */
long minOffset(final MessageQueue mq) 
    
/**
返回消息消费队列中第一条消息的存储时间戳。
*/
long earliestMsgStoreTime(MessageQueue mq)
    
/**
根据消息的物理偏移量查找消息
*/
MessageExt viewMessage(String offsetMsgId)
    
/**
根据主题与消息的全局唯一 ID 查找消息。
*/    
MessageExt viewMessage(String topic, String msgId)
    
/**
批量查询消息，其参数列表如下：

String topic：主题名称
String key：消息索引 Key
int maxNum：本次查询最大返回消息条数
long begin：开始时间戳
long end：结束时间戳
*/
QueryResult queryMessage(String topic, String key, int maxNum, long begin,long end)

```
# 写在最后

简单的进行一些API讲解，我们可以下具体使用到之后再来本文查阅会更有实际意义。