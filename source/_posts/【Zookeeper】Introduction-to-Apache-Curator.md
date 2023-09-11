---
title: 【Zookeeper】Introduction to Apache Curator
subtitle: curator介绍和使用
author: lazytime
url_suffix: zkapachcurator
tags:
  - Zookeeper
categories:
  - Zookeeper
keywords: Zookeeper,curator
description: Zookeeper使用
copyright: true
date: 2023-09-11 10:23:18
---

#zookeeper 
# 原文

[https://www.baeldung.com/apache-curator](https://www.baeldung.com/apache-curator)

## [**1. Introduction**](https://www.baeldung.com/apache-curator#introduction)

[Apache Curator](https://curator.apache.org/) is a Java client for [Apache Zookeeper](https://zookeeper.apache.org/), the popular coordination service for distributed applications.

[Apache Curator](https://curator.apache.org/) 是 [Apache Zookeeper](https://zookeeper.apache.org/) 的 Java 客户端，后者是分布式应用程序的流行协调服务。

In this tutorial, we'll introduce some of the most relevant features provided by Curator:

- Connection Management – managing connections and retry policies
- Async – enhancing existing client by adding async capabilities and the use of Java 8 lambdas
- Configuration Management – having a centralized configuration for the system
- Strongly-Typed Models – working with typed models
- Recipes – implementing leader election, distributed locks or counters

在本文中，我们讲介绍一些Curator提供的最实用的功能实现：

- 连接管理：管理连接和重试策略
- 异步：通过添加异步功能和使用 Java 8 lambda 来增强现有客户端
- 配置管理：系统集中配置。
- 强类型模型：作用于类型模型上
- Recipes：实现领导选举，分布式锁或者计数器



<!-- more -->

## [**2. Prerequisites**](https://www.baeldung.com/apache-curator#prerequisites)

To start with, it’s recommended to take a quick look at the [Apache Zookeeper](https://zookeeper.apache.org/) and its features.

首先，建议快速浏览一下 [Apache Zookeeper](https://zookeeper.apache.org/) 及其功能。

For this tutorial, we assume that there's already a standalone Zookeeper instance running on _127.0.0.1:2181_; [here are](https://zookeeper.apache.org/doc/current/zookeeperStarted.html) instructions on how to install and run it, if you're just getting started.

对于本教程，我们假设已经有一个独立的 Zookeeper 实例在 _127.0.0.1:2181_ 上运行；[这里](https://zookeeper.apache.org/doc/current/zookeeperStarted.html) 会介绍如何安装并且运行

> [[【Zookeeper】基于3台linux虚拟机搭建zookeeper集群]]

First, we’ll need to add the [curator-x-async](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.curator%22%20AND%20a%3A%22curator-x-async%22) dependency to our _pom.xml_:

首先，我们需要添加 [curator-x-async](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.curator%22%20AND% 20a%3A%22curator-x-async%22) 对我们的 _pom.xml_ 的依赖：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-x-async</artifactId>
    <version>4.0.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**The latest version of Apache Curator 4.X.X has a hard dependency with Zookeeper 3.5.X** which is still in beta right now.

**最新版本的 Apache Curator 4.X.X 与 Zookeeper 3.5.X 具有硬依赖性**。

And so, in this article, we’re going to use the currently latest stable [Zookeeper 3.4.11](https://zookeeper.apache.org/doc/r3.4.11/index.html) instead.

在本文中会使用文章编写时候的最后一个稳定版本 [Zookeeper 3.4.11](https://zookeeper.apache.org/doc/r3.4.11/index.html) 替代

So we need to exclude the Zookeeper dependency and add [the dependency for our Zookeeper version](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.zookeeper%22%20AND%20a%3A%22zookeeper%22) to our _pom.xml_:

因此，我们需要排除 Zookeeper 依赖项并添加[我们的 Zookeeper 版本的依赖项](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.zookeeper%22% 20AND%20a%3A%22zookeeper%22) 到我们的 _pom.xml_：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

For more information about compatibility, please refer to [this link](https://curator.apache.org/zk-compatibility-34.html).

更多信息可以查看： [this link](https://curator.apache.org/zk-compatibility-34.html)
## [**3. Connection Management**](https://www.baeldung.com/apache-curator#connection_management)

**The basic use case of Apache Curator is connecting to a running Apache Zookeeper instance**.

最基础的使用案例，是使用Curator连接并且运行一个Apach Zk 实例。

The tool provides a factory to build connections to Zookeeper using retry policies:

工具提供了一个工厂构建ZK连接，并且需要手动指定重试器。

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy = new RetryNTimes(
  maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
  .newClient("127.0.0.1:2181", retryPolicy);
client.start();
 
assertThat(client.checkExists().forPath("/")).isNotNull();
```

In this quick example, we'll retry 3 times and will wait 100 ms between retries in case of connectivity issues.

在这个快速使用案例中，我们指定了重试三次，并且每次重试的间隔等待100ms。

Once connected to Zookeeper using the _CuratorFramework_ client, we can now browse paths, get/set data and essentially interact with the server.

使用 _CuratorFramework_ 客户端连接到 Zookeeper 后，我们现在可以浏览路径、获取/设置数据并与服务器进行交互。
## [**4. Async**](https://www.baeldung.com/apache-curator#async)

**The Curator Async module wraps the above _CuratorFramework_ client to provide non-blocking capabilities** using [the CompletionStage Java 8 API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletionStage.html).

**Curator Async 模块使用[CompletionStage Java 8 API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletionStage.html) 对上述 _CuratorFramework_ 客户端进行封装，以提供非阻塞功能**。

Let's see how the previous example looks like using the Async wrapper:

让我们看看使用 Async 封装器的前一个示例是怎样的：

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy 
  = new RetryNTimes(maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
  .newClient("127.0.0.1:2181", retryPolicy);

client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);

AtomicBoolean exists = new AtomicBoolean(false);

async.checkExists()
  .forPath("/")
  .thenAcceptAsync(s -> exists.set(s != null));

await().until(() -> assertThat(exists.get()).isTrue());
```

Now, the _checkExists()_ operation works in asynchronous mode, not blocking the main thread. We can also chain actions one after each other using the _thenAcceptAsync()_ method instead, which uses the [CompletionStage API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletionStage.html).

现在，_checkExists()_ 操作以异步模式运行，不会阻塞主线程。我们还可以使用_thenAcceptAsync()_方法（该方法使用 [CompletionStage API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletionStage.html)）将操作一个接一个地串联起来。
## [**5. Configuration Management**](https://www.baeldung.com/apache-curator#configuration_management)

In a distributed environment, one of the most common challenges is to manage shared configuration among many applications. **We can use Zookeeper as a data store where to keep our configuration.**

在分布式环境中，最常见的挑战之一就是管理多个应用程序之间的共享配置。 **我们可以使用 Zookeeper 作为保存配置的数据存储。

Let's see an example using Apache Curator to get and set data:

让我们来看一个使用 Apache Curator 获取和设置数据的示例：

```java
CuratorFramework client = newClient();
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

client.create().forPath(key);

async.setData()
  .forPath(key, expected.getBytes());

AtomicBoolean isEquals = new AtomicBoolean();
async.getData()
  .forPath(key)
  .thenAccept(data -> isEquals.set(new String(data).equals(expected)));

await().until(() -> assertThat(isEquals.get()).isTrue());
```

In this example, we create the node path, set the data in Zookeeper, and then we recover it checking that the value is the same. The _key_ field could be a node path like _/config/dev/my_key_.

在这个示例中，我们创建节点路径，在 Zookeeper 中设置数据，然后检查值是否相同，再恢复数据。_key_ 字段可以是一个节点路径，如 _/config/dev/my_key_。
### **5.1. Watchers**[](https://www.baeldung.com/apache-curator#1-watchers)

Another interesting feature in Zookeeper is the ability to watch keys or nodes. **It allows us to listen to changes in the configuration and update our applications without needing to redeploy**.

Zookeeper 的另一个有趣功能是监视键或节点。 **它允许我们监听配置中的变化并更新应用程序，而无需重新部署**。

Let's see how the above example looks like when using watchers:

让我们看看上面的示例在使用Watch时是什么样子的：

```java
CuratorFramework client = newClient()
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

async.create().forPath(key);

List<String> changes = new ArrayList<>();

async.watched()
  .getData()
  .forPath(key)
  .event()
  .thenAccept(watchedEvent -> {
    try {
        changes.add(new String(client.getData()
          .forPath(watchedEvent.getPath())));
    } catch (Exception e) {
        // fail ...
    }});

// Set data value for our key
async.setData()
  .forPath(key, expected.getBytes());

await()
  .until(() -> assertThat(changes.size()).isEqualTo(1));
```

We configure the watcher, set the data, and then confirm the watched event was triggered. We can watch one node or a set of nodes at once.

我们配置监视器、设置数据，然后确认被监视事件已触发。我们可以同时观察一个节点或一组节点。

## [**6. Strongly Typed Models**](https://www.baeldung.com/apache-curator#strongly_typed_models)

Zookeeper primarily works with byte arrays, so we need to serialize and deserialize our data. This allows us some flexibility to work with any serializable instance, but it can be hard to maintain.

Zookeeper 主要使用字节数组，因此我们需要对数据进行序列化和反序列化。这让我们可以灵活地使用任何可序列化的实例，但也很难维护。

To help here, Curator adds the concept of [typed models](https://curator.apache.org/curator-x-async/modeled.html) which **delegates the serialization/deserialization and allows us to work with our types directly**. Let's see how that works.

为了在这方面提供帮助，Curator 添加了[类型化模型](https://curator.apache.org/curator-x-async/modeled.html)的概念，它**了序列化/反序列化，并允许我们直接**我们的类型。让我们看看它是如何工作的。

First, we need a serializer framework. Curator recommends using the Jackson implementation, so let's add [the Jackson dependency](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) to our _pom.xml_:

首先，我们需要一个序列化框架。Curator 推荐使用 Jackson 实现，因此让我们在 _pom.xml_ 中添加[Jackson 依赖项](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

Now, let's try to persist our custom class _HostConfig_:

现在，让我们尝试持久化自定义类 _HostConfig_：

```java
public class HostConfig {
    private String hostname;
    private int port;

    // getters and setters
}
```

We need to provide the model specification mapping from the _HostConfig_ class to a path, and use the modeled framework wrapper provided by Apache Curator:

我们需要提供从 _HostConfig_ 类到路径的模型规范映射，并使用 Apache Curator 提供的建模框架包装器：

```java
ModelSpec<HostConfig> mySpec = ModelSpec.builder(
  ZPath.parseWithIds("/config/dev"), 
  JacksonModelSerializer.build(HostConfig.class))
  .build();

CuratorFramework client = newClient();
client.start();

AsyncCuratorFramework async 
  = AsyncCuratorFramework.wrap(client);
ModeledFramework<HostConfig> modeledClient 
  = ModeledFramework.wrap(async, mySpec);

modeledClient.set(new HostConfig("host-name", 8080));

modeledClient.read()
  .whenComplete((value, e) -> {
     if (e != null) {
          fail("Cannot read host config", e);
     } else {
          assertThat(value).isNotNull();
          assertThat(value.getHostname()).isEqualTo("host-name");
          assertThat(value.getPort()).isEqualTo(8080);
     }
   });
```

The _whenComplete()_ method when reading the path _/config/dev_ will return the _HostConfig_ instance in Zookeeper.

读取 _/config/dev_ 路径时，_whenComplete()_ 方法将返回 Zookeeper 中的 _HostConfig_ 实例。

## [**7. Recipes**](https://www.baeldung.com/apache-curator#recipes)

Zookeeper provides [this guideline](https://zookeeper.apache.org/doc/current/recipes.html) to implement **high-level solutions or recipes such as leader election, distributed locks or shared counters.**

Zookeeper 提供了[本指南](https://zookeeper.apache.org/doc/current/recipes.html)来实现**高层次的解决方案或秘诀，如领导者选举、分布式锁或共享计数器。**

Apache Curator provides an implementation for most of these recipes. To see the full list, visit [the Curator Recipes documentation](https://curator.apache.org/curator-recipes/index.html).

Apache Curator 提供了其中大部分方案的实现方法。要查看完整列表，请访问[Curator Recipes 文档](https://curator.apache.org/curator-recipes/index.html)。

All of these recipes are available in a separate module:

所有这些API都可以在单独的模块中找到：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
</dependency>
```

Let's jump right in and start understanding these with some simple examples.

让我们通过一些简单的例子，直接开始了解这些知识。
### [**7.1. Leader Election**](https://www.baeldung.com/apache-curator#1-leader-election)

In a distributed environment, we may need one master or leader node to coordinate a complex job.

在分布式环境中，我们可能需要一个主节点或领导节点来协调复杂的工作。

This is how the usage of [the Leader Election recipe](https://curator.apache.org/curator-recipes/leader-election.html) in Curator looks like:

这就是 [领导人选举配方](https://curator.apache.org/curator-recipes/leader-election.html) 在 Curator 中的用法：

```java
CuratorFramework client = newClient();
client.start();
LeaderSelector leaderSelector = new LeaderSelector(client, 
  "/mutex/select/leader/for/job/A", 
  new LeaderSelectorListener() {
      @Override
      public void stateChanged(
        CuratorFramework client, 
        ConnectionState newState) {
      }

      @Override
      public void takeLeadership(
        CuratorFramework client) throws Exception {
      }
  });

// join the members group
leaderSelector.start();

// wait until the job A is done among all members
leaderSelector.close();
```

When we start the leader selector, our node joins a members group within the path _/mutex/select/leader/for/job/A_. Once our node becomes the leader, the _takeLeadership_ method will be invoked, and we as leaders can resume the job.

当我们启动领导者选择器时，我们的节点会加入 _/mutex/select/leader/for/job/A_ 路径下的成员组。一旦我们的节点成为领导者，_takeLeadership_ 方法就会被调用，作为领导者的我们就可以继续执行任务。
### [**7.2. Shared Locks**](https://www.baeldung.com/apache-curator#2-shared-locks)

[The Shared Lock recipe](https://curator.apache.org/curator-recipes/shared-lock.html) is about having a fully distributed lock:

[共享锁配方](https://curator.apache.org/curator-recipes/shared-lock.html) 是关于完全分布式锁的：

```java
CuratorFramework client = newClient();
client.start();
InterProcessSemaphoreMutex sharedLock = new InterProcessSemaphoreMutex(
  client, "/mutex/process/A");

sharedLock.acquire();

// do process A

sharedLock.release();
```

When we acquire the lock, Zookeeper ensures that there's no other application acquiring the same lock at the same time.

当我们获取锁时，Zookeeper 会确保没有其他应用程序同时获取相同的锁。
### [**7.3. Counters**](https://www.baeldung.com/apache-curator#3-counters)

[The Counters recipe](https://curator.apache.org/curator-recipes/shared-counter.html) coordinates a shared _Integer_ among all the clients:

[计数器配方](https://curator.apache.org/curator-recipes/shared-counter.html) 协调所有客户端共享的_整数：

```java
CuratorFramework client = newClient();
client.start();

SharedCount counter = new SharedCount(client, "/counters/A", 0);
counter.start();

counter.setCount(counter.getCount() + 1);

assertThat(counter.getCount()).isEqualTo(1);
```

In this example, Zookeeper stores the _Integer_ value in the path _/counters/A_ and initializes the value to _0_ if the path has not been created yet.

在此示例中，Zookeeper 将 _Integer_ 值存储在 _/counters/A_ 路径中，如果路径尚未创建，则将该值初始化为 _0_。
## **8. Conclusion**[](https://www.baeldung.com/apache-curator#conclusion)

In this article, we’ve seen how to use Apache Curator to connect to Apache Zookeeper and take benefit of its main features.

在这篇文章中，我们介绍了如何使用Apach Curator连接Apach Zookeeper，并利用其主要功能。

We’ve also introduced a few of the main recipes in Curator.

我们还介绍了 Curator 中的一些主要不见。

As usual, sources can be found [over on GitHub](https://github.com/eugenp/tutorials/tree/master/apache-libraries).

与往常一样，您可以在 [GitHub](https://github.com/eugenp/tutorials/tree/master/apache-libraries) 上找到源代码。