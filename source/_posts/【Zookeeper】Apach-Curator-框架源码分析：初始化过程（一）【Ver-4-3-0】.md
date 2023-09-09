---
title: 【Zookeeper】Apach Curator 框架源码分析：初始化过程（一）【Ver 4.3.0】
subtitle: Curator源码分析第一篇
author: lazytime
url_suffix: curatorsource1
tags:
  - Zookeeper
categories:
  - Zookeeper
keywords: Curator
description: Zookeeper的客户端Curator使用
copyright: true
date: 2023-08-10 16:57:00
---
# 介绍

Curator是netflix公司开源的一套zookeeper客户端，目前是Apache的顶级项目。

和ZK的原生客户端相比，Curator的抽象层次要更高，同时简化了ZK的常用功能开发量，比如Curator自带连接重试、反复注册Watcher、NodeExistsException 异常处理等等。

根据官方的介绍，我们可以了解到它是一个用于分布式的Java客户端API工具。它基于`high-level API`，拥有它可以更简单易懂的指挥Zookeeper实现分布式安全应用程序开发。

Curator由一系列的模块构成，对于一般开发者而言，常用的是**curator-framework**和**curator-recipes**，以及广为熟知的 **分布式锁**。

Curator 当然也包括许多扩展，比如**服务发现**和**基于Java 8异步DSL**。

```txt
Apache Curator is a Java/JVM client library for [Apache ZooKeeper](https://zookeeper.apache.org/), a distributed coordination service.

Apache Curator includes a high-level API framework and utilities to make using Apache ZooKeeper much easier and more reliable. It also includes recipes for common use cases and extensions such as service discovery and a Java 8 asynchronous DSL.
```

> 用官方的介绍来说就是：guava之于java就像curator之于zookeeper 

<!-- more -->

# ZK 版本支持

Curator 目前最新的版本为 5.X 的版本，已经不支持 ZK 的 3.4.X 以及之前的版本，阅读源码之前经过认真考虑，最终选择了 ZK的 **3.5.10** 版本。

> 5.X 对于 Curator 做了不少破坏性的改动，不兼容的原因如下：
> 
> - 旧的ListenerContainer类已经被移除，以避免Guava类泄漏。
> - ConnectionHandlingPolicy和相关类已被删除
> - Reaper和ChildReaper类/recipes已被删除。您应该改用 ZooKeeper 容器节点。
> - newPersistentEphemeralNode()和newPathChildrenCache()已从GroupMember中移除。
> - ServiceCacheBuilder< T> executorService(CloseableExecutorService executorService)已从ServiceCacheBuilder中移除。
> - ServiceProviderBuilder< T> executorService(CloseableExecutorService executorService)已从ServiceProviderBuilder中移除。
> - static boolean shouldRetry(int rc)已从RetryLoop中移除。
> - static boolean isRetryException(Throwable exception)已从RetryLoop中移除。

# 官网地址

[Apache Curator](https://curator.apache.org/)

## 下载地址

Curator Maven 相关地址：[https://mvnrepository.com/artifact/org.apache.curator](https://mvnrepository.com/artifact/org.apache.curator)

Curator jar包下载地址：[https://cwiki.apache.org/confluence/display/CURATOR/Releases](https://cwiki.apache.org/confluence/display/CURATOR/Releases)

# 快速开始

## ZK 集群部署

学习之前需要使用ZK搭建集群环境，方便Debug的时候调试代码。这部分搭建过程放到另一篇文章：

[[【Zookeeper】基于3台linux虚拟机搭建zookeeper集群]]

## Maven依赖引入

下面是对应的Zookeeper和Curator的版本选择。

```xml
<curator.version>4.3.0</curator.version>  
<zookeeper.version>3.5.10</zookeeper.version>
```

```xml
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
	<version>${curator.version}</version>
	<exclusions>
		<exclusion>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-recipes</artifactId>
	<version>${curator.version}</version>
	<exclusions>
		<exclusion>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>${zookeeper.version}</version>
</dependency>
```

## 构建入门实例

Curator 最为核心和强大并且常用功能是分布式锁。

在入门demo中可以看到整个 Curator 依靠 **CuratorFrameworkFactory** 构建，使用 Curator 进行分布式加锁解锁操作，只需要为所连接的ZooKeeper集群提供一个CuratorFramework对象即可。

```java
CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy)
```

上面的方法将会使用默认值创建与ZooKeeper集群的连接，调用放只需要关注使用到的重试策略。

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3)
CuratorFramework client = CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy);
client.start();
```

从参数值可以大致了解到，这里使用的策略是指数递增间隔的方式尝试重试时间，并且指定重试三次。

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);  
CuratorFramework client =  
        CuratorFrameworkFactory.newClient("192.168.0.1;192.168.0.2;192.168.0.3", retryPolicy);  
client.start();  
// 此处就获取到 zk的一个连接实例。  
//.....
```

拥有了 **CuratorFramework** 实例之后，就可以直接通过 API 调用操作ZK。

```java
client.create().forPath("/my/path", myData)
```

> 这样的直接调用还有个好处是client实例如果碰到网络抖动等情况会自动重试，重试过程不需要开发者自己实现。

## 可重入锁（公平锁）案例代码

下面是官网可重入锁的Demo使用代码。

```java
InterProcessMutex lock = new InterProcessMutex(client, lockPath);
if ( lock.acquire(maxWait, waitUnit) ) 
{
    try 
    {
        // do some work inside of the critical section here
    }
    finally
    {
        lock.release();
    }
}
```

这里改造一下即可简单使用。

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);  
CuratorFramework client =  
        CuratorFrameworkFactory.newClient("192.168.0.1,192.168.0.2,192.168.0.3", retryPolicy);  
client.start();  
// 此处就获取到 zk的一个连接实例。  
//.....  
client.create().forPath("/my/path", "Test".getBytes());  
InterProcessMutex lock = new InterProcessMutex(client, "/test/myLock");  
lock.acquire();  
try {  
    // do some work inside of the critical section here  
    Thread.sleep(3000);  
} finally {  
    lock.release();  
}
```

整个Demo案例代码比较简单，下面直接开始介绍初始化过程。

本文主要介绍和**Curator初始化**、内部的**通知机制**以及**会话管理**部分。

# 初始化过程流程图

初始化过程流程图全图如下。下面将会一步步拆解这幅图是如何拼凑的。

![Curator 源码分析.drawio.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/Curator%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.drawio.png)

> Drawio 源文件和图片地址如下：
> 链接：[https://pan.baidu.com/s/18PoMjkp11LztmNB3XgZ0qw?pwd=4bug](https://pan.baidu.com/s/18PoMjkp11LztmNB3XgZ0qw?pwd=4bug )
	提取码：4bug 

# 初始化源码分析

## CuratorFramework 初始化过程

### 初始化过程流程图

CuratorFramework 初始化过程为下面截图这一部分，红色部分为个人认为相对比较重要的对象和变量。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230708120150.png)


### CuratorFrameworkFactory.newClient() 代码分析

下面通过`CuratorFrameworkFactory.newClient()`一步步探究整个初始化过程。

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);  
CuratorFramework client =  
        CuratorFrameworkFactory.newClient("192.168.19.100:2181,192.168.19.101:2181,192.168.19.102:2181", retryPolicy);

```

在获取分布式锁之前，我们需要先连接ZK集群，整个过程通过两行代码完成。

首先，我们需要确定连接ZK的重试策略，接着通过`CuratorFrameworkFactory`构建`Curator` 实例，`Curator `内部根据ZK原生客户端做了一层封装，开发者使用过程中不需要关注。

```java
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);  
CuratorFramework client =  CuratorFrameworkFactory.newClient("192.168.0.1,192.168.0.2,192.168.0.3", retryPolicy);
```

上面是简单的模板代码。**ExponentialBackoffRetry** 构建重试策略为按照指数增长重试时间，比如第一次1秒，第二次2秒，第三次4秒，第四次8秒..... 

接着是利用`CuratorFrameworkFactory`构建实例。

```java
return newClient(connectString, DEFAULT_SESSION_TIMEOUT_MS, DEFAULT_CONNECTION_TIMEOUT_MS, retryPolicy);
```

这里强调一下两个常量 **DEFAULT_SESSION_TIMEOUT_MS** （默认的会话超时时间）、**DEFAULT_CONNECTION_TIMEOUT_MS**（默认的连接超时时间），作用是传入重试策略时候填写默认参数。

```java
private static final int DEFAULT_SESSION_TIMEOUT_MS
    = Integer.getInteger("curator-default-session-timeout", 60 * 1000)
```

```java
private static final int DEFAULT_CONNECTION_TIMEOUT_MS = Integer.getInteger("curator-default-connection-timeout", 15 * 1000);
```

我们进一步进入构造方法，这里用了建造者模式。

```java
return builder().  
    connectString(connectString).  
    sessionTimeoutMs(sessionTimeoutMs).  
    connectionTimeoutMs(connectionTimeoutMs).  
    retryPolicy(retryPolicy).  
    build();
```

`build()`工作完成之后，后续的调用实际上调用的是**CuratorFrameworkImpl**实例，注意这里把**CuratorFrameworkFactory**的**this**引用逸出给**CuratorFrameworkImpl**对象。

```java
return new CuratorFrameworkImpl(this);
```

`CuratorFrameworkImpl` 构造方法的内容比较多，这里在源码对于相对重要的组件进行标注，这里的**CuratorZookeeperClient**这个对象，相当于ZK原生客户端的封装对象，Curator的很多质量都是由它来完成调用的。

```java
  
public CuratorFrameworkImpl(CuratorFrameworkFactory.Builder builder)  
{  
    ZookeeperFactory localZookeeperFactory = makeZookeeperFactory(builder.getZookeeperFactory());  
    this.client = new CuratorZookeeperClient  
        (  
            localZookeeperFactory,  
            builder.getEnsembleProvider(),  
            builder.getSessionTimeoutMs(),  
            builder.getConnectionTimeoutMs(),  
            builder.getWaitForShutdownTimeoutMs(),  
            new Watcher()  
            {  
                @Override  
                public void process(WatchedEvent watchedEvent)  
                {  
                    CuratorEvent event = new CuratorEventImpl(CuratorFrameworkImpl.this, CuratorEventType.WATCHED, watchedEvent.getState().getIntValue(), unfixForNamespace(watchedEvent.getPath()), null, null, null, null, null, watchedEvent, null, null);  
                    processEvent(event);  
                }  
            },  
            builder.getRetryPolicy(),  
            builder.canBeReadOnly(),  
            builder.getConnectionHandlingPolicy()  
        );  
  //用于判断连接断开和连接超时的状态，设置curator的连接状态，并通过connectionStateManager触发连接事件状态通知
    internalConnectionHandler = new StandardInternalConnectionHandler();
     
    //接收事件的通知。后台线程操作事件和连接状态事件会触发 
    listeners = new ListenerContainer<CuratorListener>();  
    
    //当后台线程发生异常或者handler发生异常的时候会触发
    unhandledErrorListeners = new ListenerContainer<UnhandledErrorListener>();  
    //后台线程执行的操作队列
    backgroundOperations = new DelayQueue<OperationAndData<?>>();  
    forcedSleepOperations = new LinkedBlockingQueue<>();  
    //命名空间
    namespace = new NamespaceImpl(this, builder.getNamespace());  

//线程工厂方法，初始化后台线程池时会使用
    threadFactory = getThreadFactory(builder);  

maxCloseWaitMs = builder.getMaxCloseWaitMs();  

//负责连接状态变化时的通知
    connectionStateManager = new ConnectionStateManager(this, builder.getThreadFactory(), builder.getSessionTimeoutMs(), builder.getConnectionHandlingPolicy().getSimulatedSessionExpirationPercent(), builder.getConnectionStateListenerDecorator());  
    compressionProvider = builder.getCompressionProvider();  
    aclProvider = builder.getAclProvider();  
    
    //CuratorFrameworkImpl的状态，调用start方法之前为 LATENT，调用start方法之后为 STARTED ,调用close()方法之后为STOPPEDstate = new AtomicReference<CuratorFrameworkState>(CuratorFrameworkState.LATENT);  
    useContainerParentsIfAvailable = builder.useContainerParentsIfAvailable(); 
    //错误连接策略 
    connectionStateErrorPolicy = Preconditions.checkNotNull(builder.getConnectionStateErrorPolicy(), "errorPolicy cannot be null");  
    schemaSet = Preconditions.checkNotNull(builder.getSchemaSet(), "schemaSet cannot be null");  
    zk34CompatibilityMode = builder.isZk34CompatibilityMode();  
  
    byte[] builderDefaultData = builder.getDefaultData();  
    defaultData = (builderDefaultData != null) ? Arrays.copyOf(builderDefaultData, builderDefaultData.length) : new byte[0];  
    authInfos = buildAuths(builder);  
    
  //有保障的执行删除操作，其实是不断尝试直到删除成功，通过递归调用实现
    failedDeleteManager = new FailedDeleteManager(this);  
    
    //有保障的执行删除watch操作
    failedRemoveWatcherManager = new FailedRemoveWatchManager(this);  
    
    namespaceFacadeCache = new NamespaceFacadeCache(this);  
    
  //服务端可用节点的检测器，第一次连接和重连成功之后都会触发重新获取服务端列表
    ensembleTracker = zk34CompatibilityMode ? null : new EnsembleTracker(this, builder.getEnsembleProvider());  
  
    runSafeService = makeRunSafeService(builder);
```

`newClient`的目的是构建ZK连接实例，包括一系列附加核心组件：后台操作、连接事件、异常监控、容器，命名空间、负载均衡等等。

## CuratorZookeeperClient 初始化过程

### CuratorZookeeperClient 初始化过程流程图

CuratorZookeeperClient 初始化过程图如下：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230708120426.png)

### CuratorZookeeperClient 初始化代码分析

上面提到，`CuratorFrameworkImp`l的初始化过程中，有一段比较重要的`CuratorZookeeperClient`客户端初始化过程，下面就来看看这个初始化过程干了啥。

```java
public CuratorZookeeperClient(ZookeeperFactory zookeeperFactory, EnsembleProvider ensembleProvider,
            int sessionTimeoutMs, int connectionTimeoutMs, int waitForShutdownTimeoutMs, Watcher watcher,
            RetryPolicy retryPolicy, boolean canBeReadOnly, ConnectionHandlingPolicy connectionHandlingPolicy)
    {

		// StandardConnectionHandler当收到Disconnect事件后，如果在规定时间内没有重连到服务器，则会主动触发Expired事件
        this.connectionHandlingPolicy = connectionHandlingPolicy;
        if ( sessionTimeoutMs < connectionTimeoutMs )
        {
            log.warn(String.format("session timeout [%d] is less than connection timeout [%d]", sessionTimeoutMs, connectionTimeoutMs));
        }
		// 重连策略
        retryPolicy = Preconditions.checkNotNull(retryPolicy, "retryPolicy cannot be null");
        ensembleProvider = Preconditions.checkNotNull(ensembleProvider, "ensembleProvider cannot be null");

        this.connectionTimeoutMs = connectionTimeoutMs;
        this.waitForShutdownTimeoutMs = waitForShutdownTimeoutMs;
        // //curator注册到原生客户端上的defaultWatcher,会收到和连接状态有关的事件通知等，负责超时重连
        state = new ConnectionState(zookeeperFactory, ensembleProvider, sessionTimeoutMs, connectionTimeoutMs, watcher, tracer, canBeReadOnly, connectionHandlingPolicy);

		//  重试策略设置
        setRetryPolicy(retryPolicy);
    }
```

**ConnectionState**是`Curator`注册到原生客户端上的**defaultWatcher**，它会收到和连接状态有关的事件通知等，负责超时重连操作等。

再来看下`ConnectionState`的构造方法。

```java
ConnectionState(ZookeeperFactory zookeeperFactory, EnsembleProvider ensembleProvider, int sessionTimeoutMs, int connectionTimeoutMs, Watcher parentWatcher, AtomicReference<TracerDriver> tracer, boolean canBeReadOnly, ConnectionHandlingPolicy connectionHandlingPolicy)  
{  
    this.ensembleProvider = ensembleProvider;  
    this.sessionTimeoutMs = sessionTimeoutMs;  
    this.connectionTimeoutMs = connectionTimeoutMs;  
    this.tracer = tracer;  
    this.connectionHandlingPolicy = connectionHandlingPolicy;  
    if ( parentWatcher != null )  
    {  
	    // 因为defaultWatcher只能有一个，通过parentWatchers可实现defaultWatcher接到事件通知时parentWatchers的回调
        parentWatchers.offer(parentWatcher);  
    }  
  
    handleHolder = new HandleHolder(zookeeperFactory, this, ensembleProvider, sessionTimeoutMs, canBeReadOnly);  
}
```

**parentWatchers** 使用了并发安全队列 **ConcurrentLinkedQueue**，这部分属于JDK并发编程的基础内容，这个队列的作用如下：

> **ConcurrentLinkedQueue**：一个基于链接节点的**无界线程安全队列**。此队列按照 FIFO（**先进先出**）原则对元素进行排序。队列的**头部** 是队列中**时间最长的元素**。队列的尾部 是队列中时间最短的元素。**新的元素插入到队列的尾部**，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue 是一个恰当的选择。此队列不允许使用 null 元素。

```java
private final Queue<Watcher> parentWatchers = new ConcurrentLinkedQueue<Watcher>();
```

## ConnectionStateManager 初始化过程

### ConnectionStateManager 初始化过程流程图

**ConnectionStateManager** 主要是持有`Client`引用，通过连接状态管理工程创建构建监听器，以及构建只允许一个线程执行的线程池。

> Curator 的设计记录是一个客户端永远只有一个线程负责工作。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230704210542.png)

### ConnectionStateManager 初始化代码分析

在`Curator`框架初始化代码中包含了 **ConnectionStateManager** 初始化，它主要负责状态维护和连接状态变更通知。

```java
//负责连接状态变化时的通知
connectionStateManager = new ConnectionStateManager(this, builder.getThreadFactory(), builder.getSessionTimeoutMs(), builder.getConnectionHandlingPolicy().getSimulatedSessionExpirationPercent(), builder.getConnectionStateListenerManagerFactory());
```

可以看到，如果要监听状态改变，需要注册一个监听器。相关的注册方式在“”部分进行详细介绍，这里先看下相关的成员变量以及初始化方法。

```java
//连接状态事件通知队列
private final BlockingQueue<ConnectionState> eventQueue = new ArrayBlockingQueue<ConnectionState>(QUEUE_SIZE);

//需要通知的listeners 
private final UnaryListenerManager<ConnectionStateListener> listeners;

//ConnectionStateManager的运行状态 
private final AtomicReference<State> state = new AtomicReference<State>(State.LATENT);
```

**ConnectionStateManager#ConnectionStateManager**

```java
/**
Params:
client – the client 

threadFactory – thread factory to use or null for a default 

sessionTimeoutMs – the ZK session timeout in milliseconds 

sessionExpirationPercent – percentage of negotiated session timeout to use when simulating a session timeout. 0 means don't simulate at all 

managerFactory – manager factory to use
*/
public ConnectionStateManager(CuratorFramework client, ThreadFactory threadFactory, int sessionTimeoutMs, int sessionExpirationPercent, ConnectionStateListenerManagerFactory managerFactory)  
{  
    this.client = client;  
    this.sessionTimeoutMs = sessionTimeoutMs;  
    this.sessionExpirationPercent = sessionExpirationPercent;  
    if ( threadFactory == null )  
    {  
        threadFactory = ThreadUtils.newThreadFactory("ConnectionStateManager");  
    }  
    //事件队列处理线程池
    service = Executors.newSingleThreadExecutor(threadFactory);  
    // 构建监听器队列
    listeners = managerFactory.newManager(client);  
}
```

## CuratorFrameworkImpl 启动过程

`CuratorFrameworkImpl`启动过程的主要工作如下：

1. 启动 **ConnectionStateManager**，同时负责连接事件的通知准备。
2. 启动 **CuratorZookeeperClient** ，建立服务端会话连接。
3. 启动一个单线程线程池，这个线程负责监听执行后台任务队列，不断从任务队列取出元素并且执行。

### CuratorFrameworkImpl 启动过程流程图

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230705073357.png)

### 客户端连接 client.start();

调用`start `：

```java
client.start();  
```

`client.start(); `内部逻辑如下，这个方法的代码都比较简单，具体可以参考注释理解。

```java
public void start()  
{  
    log.info("Starting");  
    // 使用CAS把当前的运行状态切换为 STARTED，状态切换之后不可逆
    // LATENT:CuratorFramework.start() has not yet been called
    // STARTED: CuratorFramework.start() has been called
    if ( !state.compareAndSet(CuratorFrameworkState.LATENT, CuratorFrameworkState.STARTED) )  
    {  
        throw new IllegalStateException("Cannot be started more than once");  
    }  
  
    try  
    {  
	    // ordering dependency - must be called before client.start()  
	    // 顺序依赖 - 必须在 client.start()之前调用。 
        connectionStateManager.start(); 
		// 构建连接监听器，监听异常连接状态
        final ConnectionStateListener listener = new ConnectionStateListener()  
        {  
            @Override  
            public void stateChanged(CuratorFramework client, ConnectionState newState)  
            {  
	            // CONNECTED：为第一次成功连接到服务器而发送。注意：对于任何一个CuratorFramework实例只会收到其中一条信息。
	            
	            // RECONNECTED：一个暂停的、丢失的或只读的连接已被重新建立
	            // RECONNECTED：A suspended, lost, or read-only connection has been re-established
	            // 如果已经连接或者正在重连
                if ( ConnectionState.CONNECTED == newState || ConnectionState.RECONNECTED == newState )  
                {  
                    logAsErrorConnectionErrors.set(true);  
                }  
            }  
  
            @Override  
            public boolean doNotDecorate()  
            {  
                return true;  
            }  
        };  
		// 注册监听器
        this.getConnectionStateListenable().addListener(listener);  
		// 全局启动开发设置为true，ConnectionState 状态更新
        client.start();  
		// 构建线程池
        executorService = Executors.newSingleThreadScheduledExecutor(threadFactory);         // 执行具备返回值的Callable 任务
        executorService.submit(new Callable<Object>()  
        {  
            @Override  
            public Object call() throws Exception  
            {  
	            // 关键部分：挂起后台操作
                backgroundOperationsLoop();  
                return null;            
			}  
        });  
		  
        if ( ensembleTracker != null )  
        {  
            ensembleTracker.start();  
        }  
  
        log.info(schemaSet.toDocumentation());  
    }  
    catch ( Exception e )  
    {  
        ThreadUtils.checkInterrupted(e);  
        handleBackgroundOperationException(null, e);  
    }  
}
```

通过CAS操作将当前状态更新为 **STARTED**，同时根据`if`逻辑可以得知`start()`方法不允许重复调用，这和 JDK的 Thread 设计思路比较相似，Thread 同样只允许执行一次`start()`方法。

CAS 操作成功则构建连接监听器监听异常连接状态，监听器中会判断当前客户端是否已经连接或者正在重连，如果是则设置**logAsErrorConnectionErrors=true**。

我们继续看关键部分`backgroundOperationsLoop(); `。

### 后台轮询操作指令 `backgroundOperationsLoop()`

`backgroundOperationsLoop()`方法，根据名称得知这是一个后台循环，后台任务的整体流程如下：

```java
private void backgroundOperationsLoop()  
{  
    try  
    {  
        while ( state.get() == CuratorFrameworkState.STARTED )  
        {  
            OperationAndData<?> operationAndData;  
            try            
            {  
                operationAndData = backgroundOperations.take();  
                if ( debugListener != null )  
                {  
                    debugListener.listen(operationAndData);  
                }  
                // 执行后台操作
                performBackgroundOperation(operationAndData);  
            }  
            catch ( InterruptedException e )  
            {  
	            // 在这里中断异常会被吞掉。
                // swallow the interrupt as it's only possible from either a background  
                // operation and, thus, doesn't apply to this loop or the instance                // is being closed in which case the while test will get it            }  
        }  
    }  
    finally  
    {  
        log.info("backgroundOperationsLoop exiting");  
    }  
}
```

> `OperationAndData` 实现了 Delayed 接口用于实现阻塞队列延迟重试。

上面的处理逻辑如下：

1. 判断当前是否为`STARTED`状态，一直循环。
2. 从阻塞队列**BlockingQueue**当中弹出操作指令对象，在初始化代码中可以得知是一个`DelayQueue` 延迟并发安全阻塞队列，`OperationAndData` 对象毫无疑问实现了`Delayed`接口。

```java
backgroundOperations = new DelayQueue<OperationAndData<?>>();
```

3. 判断Debug 监听器是否存在，如果存在则监听`OperationAndData`。
4. 执行后台操作`performBackgroundOperation`，它的工作是从阻塞队列不断获取数据操作`OperationAndData` 对象调用`callPerformBackgroundOperation`方法执行。
5. 如果无法正常连接ZK集群，此时会走else分支并且进入重连判断逻辑。如果符合条件，则添加到阻塞队列的当中等待下一次重试。（注意这里是**主动重试，同步操作**）

```java
void performBackgroundOperation(OperationAndData<?> operationAndData)
    {
        try
        {
            if ( !operationAndData.isConnectionRequired() || client.isConnected() )
            {
                operationAndData.callPerformBackgroundOperation();
            }
            else
            {
	            // 允许重连或者超时这样的情况发生
                client.getZooKeeper();  // important - allow connection resets, timeouts, etc. to occur
		
				// 如果连接超时，则跑出 CuratorConnectionLossException 异常
                if ( operationAndData.getElapsedTimeMs() >= client.getConnectionTimeoutMs() )
                {
                    throw new CuratorConnectionLossException();
                }
                // 如果没有超时，则推入到 forcedSleepOperations 强制睡眠后等待重连
                sleepAndQueueOperation(operationAndData);
            }
        }
        catch ( Throwable e )
        {
	        // 检查线程中断
            ThreadUtils.checkInterrupted(e);

            /**
             * Fix edge case reported as CURATOR-52. ConnectionState.checkTimeouts() throws KeeperException.ConnectionLossException
             * when the initial (or previously failed) connection cannot be re-established. This needs to be run through the retry policy
             * and callbacks need to get invoked, etc.
             */
             /*
             修复报告为CURATOR-52的边缘案例。当初始（或之前失败的）连接无法重新建立时，ConnectionState.checkTimeouts()会抛出KeeperException.ConnectionLossException。这需要通过重试策略运行，回调需要被调用，等等。
             */
             // 连接丢失异常处理
            if ( e instanceof CuratorConnectionLossException )
            {
                WatchedEvent watchedEvent = new WatchedEvent(Watcher.Event.EventType.None, Watcher.Event.KeeperState.Disconnected, null);
                CuratorEvent event = new CuratorEventImpl(this, CuratorEventType.WATCHED, KeeperException.Code.CONNECTIONLOSS.intValue(), null, null, operationAndData.getContext(), null, null, null, watchedEvent, null, null);
                // 如果重连次数
                if ( checkBackgroundRetry(operationAndData, event) )
                {
	                // 推送到backgroundOperations队列尝试重连
                    queueOperation(operationAndData);
                }
                else
                {
	                // 放弃重连
                    logError("Background retry gave up", e);
                }
            }
            else
            {
	            // 否则需要处理后台操作异常
                handleBackgroundOperationException(operationAndData, e);
            }
        }
    }
```

这里顺带介绍下后台决定是否重试的判断逻辑，主要是根据用户传输的重试策略执行对应的重试逻辑判断，比较经典的**策略模式**实现。

```java
client.getRetryPolicy().allowRetry(operationAndData.getThenIncrementRetryCount(), operationAndData.getElapsedTimeMs(), operationAndData)
```

### operationAndData.callPerformBackgroundOperation() 后台任务执行

**operationAndData** 继承了**DelayQueue**，运用多态特性拥有不同实现，内部只有一行代码：

```java
void callPerformBackgroundOperation() throws Exception  
{  
    operation.performBackgroundOperation(this);  
}
```

> operation.performBackgroundOperation(this);  对应 **BackgroundOperation#performBackgroundOperation**

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230708125752.png)

**BackgroundOperation** 后台操作有很多具体的实现，对应了ZK常见操作。传递的`this`就是 `operationAndData` 对象。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230703204542.png)

## 会话管理

1. Client 连接状态都是通过 **ConnectionState** 进行管理的，它会负责尝试超时重连的操作。
2. **ConnectionStateManager** 会负责连接状态的改变和通知。
3. **ConnectionHandlingPolicy**  则对应了连接超时策略的触发。

在后台轮询队列操作指令对象过程中会在状态改变的时候尝试重连，客户端重连必然要通知到对应的监听器，那么 **Curator** 是如何进行客户端 **会话状态通知**以及**会话超时重连**的？

###  连接事件监听和状态变更 ConnectionState#process

从`ConnectionState#process`的代码可以得知，连接状态相关的事件类型为`Watcher.Event.EventType.None`，会通知到所有的Wathcer。

其中`ConnectionState`作为 **defaultWatcher** ，它的事件回调如下：

```java
public void process(WatchedEvent event)  
{  
    if ( LOG_EVENTS )  
    {  
        log.debug("ConnectState watcher: " + event);  
    }  
  
    if ( event.getType() == Watcher.Event.EventType.None )  
    {  
    //isConnected：客户当前的连接状态，true表示已连接（SyncConnected 和 ConnectedReadOnly 状态）
        boolean wasConnected = isConnected.get(); 
        // 根据 org.apache.zookeeper.Watcher.Event.KeeperState 进行状态判断。 
        boolean newIsConnected = checkState(event.getState(), wasConnected);  
        if ( newIsConnected != wasConnected )  
        {  
	        // /如果连接状态发生改变，则更新
            isConnected.set(newIsConnected);  
            connectionStartMs = System.currentTimeMillis();  
            if ( newIsConnected )  
            {  
                
			//重连，更新会话超时协商时间
			// NegotiatedSessionTimeoutMs（协商会话超时）。
			                lastNegotiatedSessionTimeoutMs.set(handleHolder.getNegotiatedSessionTimeoutMs());  
                log.debug("Negotiated session timeout: " + lastNegotiatedSessionTimeoutMs.get());  
            }  
        }  
    }  

	// 通知parentWatchers, 注意初始化的时候其实传入了一个parentWatcher,会调用CuratorFrameworkImpl.processEvent
    for ( Watcher parentWatcher : parentWatchers )  
    {  
        OperationTrace trace = new OperationTrace("connection-state-parent-process", tracer.get(), getSessionId());  
        parentWatcher.process(event);  
        trace.commit();  
    }  
}
```

最后一段注释提到可以看到遍历`parentWatchers`并且调用`process`方法。这里实际上默认会有个Watcher，那就是在初始化的时候默认会注册一个Watch作为parentWatcher传入。

```java
  this.client = new CuratorZookeeperClient  
        (  
            localZookeeperFactory,  
            builder.getEnsembleProvider(),  
            builder.getSessionTimeoutMs(),  
            builder.getConnectionTimeoutMs(),  
            builder.getWaitForShutdownTimeoutMs(),  
            new Watcher()  
            {  
                @Override  
                public void process(WatchedEvent watchedEvent)  
                {  
                    CuratorEvent event = new CuratorEventImpl(CuratorFrameworkImpl.this, CuratorEventType.WATCHED, watchedEvent.getState().getIntValue(), unfixForNamespace(watchedEvent.getPath()), null, null, null, null, null, watchedEvent, null, null);  
                    // 注意初始化的时候其实传入了一个parentWatcher,会调用CuratorFrameworkImpl.processEvent
                    processEvent(event);  
                }  
            },  
            builder.getRetryPolicy(),  
            builder.canBeReadOnly(),  
            builder.getConnectionHandlingPolicy()  
        ); 
```

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230706215011.png)

这部分通知事件回调在下文会再次提到，这里简单有关印象即可。

### 连接状态检查和处理 ConnectionState#checkState

连接状态检查和处理在`ConnectionState#checkState`方法中进行。

```java
boolean newIsConnected = checkState(event.getState(), wasConnected); 
```

```java
private boolean checkState(Event.KeeperState state, boolean wasConnected)  
{  
    boolean isConnected = wasConnected;  
    boolean checkNewConnectionString = true;  
    switch ( state )  
    {  
    default:  
    case Disconnected:  
    {  
        isConnected = false;  
        break;    }  
  
    case SyncConnected:  
    case ConnectedReadOnly:  
    {  
        isConnected = true;  
        break;    }  
	// 访问权限异常
    case AuthFailed:  
    {  
        isConnected = false;  
        log.error("Authentication failed");  
        break;    }  
  
    case Expired:  
    {  
        isConnected = false;  
        checkNewConnectionString = false;  
        handleExpiredSession();  
        break;    }  
  
    case SaslAuthenticated:  
    {  
        // NOP  
        break;  
    }  
    }  
    // the session expired is logged in handleExpiredSession, so not log here  
    // 会话过期被记录在handleExpiredSession中，所以不记录在这里。 
    if (state != Event.KeeperState.Expired) {  
        new EventTrace(state.toString(), tracer.get(), getSessionId()).commit();  
    }  
  
    if ( checkNewConnectionString )  
    {  
	    //如果服务端列表发生变化，则更新
        String newConnectionString = handleHolder.getNewConnectionString();  
        if ( newConnectionString != null )  
        {  
            handleNewConnectionString(newConnectionString);  
        }  
    }  
  
    return isConnected;  
}
```

上面根据不同连接状态判断连接是否异常， 返回结果为**true**则表示连接是正常的，当会话超时过期`Expired`时，会调用`handleExpiredSession`进行`reset`操作（会话**被动重连**），这里对于非连接超时的状态进行时间追踪。

> 注意重连策略 **RetryPolicy**这个策略在主动和被动重连中均会调用。

### parentWatchers 注册和回调

发生状态变更的方法最后部分是通知所有的parentWatchers，下面来看看这个循环干了什么事情。

再次强调初始化的时候传入了一个 **parentWatcher**，会调用`CuratorFrameworkImpl.processEvent` 方法，现在来看看这部分是如何注册和回调的。

```java
// 通知parentWatchers,注意初始化的时候其实传入了一个parentWatcher,会调用CuratorFrameworkImpl.processEvent
    for ( Watcher parentWatcher : parentWatchers )  
    {  
        OperationTrace trace = new OperationTrace("connection-state-parent-process", tracer.get(), getSessionId());  
        parentWatcher.process(event);  
        trace.commit();  
    }  
```

我们直接看看这个默认的Watcher回调`CuratorFrameworkImpl#processEvent(event)` 相关代码逻辑。

```java
new Watcher()  
{  
    @Override  
    public void process(WatchedEvent watchedEvent)  
    {  
        CuratorEvent event = new CuratorEventImpl(CuratorFrameworkImpl.this, CuratorEventType.WATCHED, watchedEvent.getState().getIntValue(), unfixForNamespace(watchedEvent.getPath()), null, null, null, null, null, watchedEvent, null, null);
        // 处理事件  
        processEvent(event);  
    }  
},
```

`processEvent(event)`相关逻辑如下，首先对于状态变更判断，状态如果出现变更则通知到所有注册在 **CuratorListener** 上的监听器。

```java
private void processEvent(final CuratorEvent curatorEvent)  
{  
    if ( curatorEvent.getType() == CuratorEventType.WATCHED )  
    {  
	    //状态转换
        validateConnection(curatorEvent.getWatchedEvent().getState());  
    }  
	  //通知所有注册的CuratorListener
    listeners.forEach(new Function<CuratorListener, Void>()  
    {  
        @Override  
        public Void apply(CuratorListener listener)  
        {  
            try  
            {  
                OperationTrace trace = client.startAdvancedTracer("EventListener");  
                // 接收回调事件
                listener.eventReceived(CuratorFrameworkImpl.this, curatorEvent);  
                trace.commit();  
            }  
            catch ( Exception e )  
            {  
                ThreadUtils.checkInterrupted(e);  
                logError("Event listener threw exception", e);  
            }  
            return null;  
        }  
    });  
}
```

其中`validateConnection` 负责连接状态的转换代码。

**CuratorFrameworkImpl#validateConnection**

```java
void validateConnection(Watcher.Event.KeeperState state)  
{  
    if ( state == Watcher.Event.KeeperState.Disconnected )  
    {  
        internalConnectionHandler.suspendConnection(this);  
    }  
    else if ( state == Watcher.Event.KeeperState.Expired )  
    {  
        connectionStateManager.addStateChange(ConnectionState.LOST);  
    }  
    else if ( state == Watcher.Event.KeeperState.SyncConnected )  
    {  
        internalConnectionHandler.checkNewConnection(this);  
        connectionStateManager.addStateChange(ConnectionState.RECONNECTED);  
        unSleepBackgroundOperations();  
    }  
    else if ( state == Watcher.Event.KeeperState.ConnectedReadOnly )  
    {  
        internalConnectionHandler.checkNewConnection(this);  
        connectionStateManager.addStateChange(ConnectionState.READ_ONLY);  
    }  
}
```

可以看到实际的状态变更是依靠 **ConnectionStateManager** 组件负责的，**ZK的原生客户端状态和Curator包装的状态对应**表如下：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230706220450.png)

此外还需要注意每一个 `if` 判断的最后一行代码中有一个添加 **ConnectionState** 的操作，这个操作的意义是通知所有注册到 `listeners`的`ConnectionStateListener`。

```java
connectionStateManager.addStateChange(ConnectionState.READ_ONLY);
```

> 至于怎么通知的会在下文介绍。

## 通知机制

通知是干什么？其实就是在事件发生的时候，及时回调注册的**Listenrner监听器**对应的回调函数。Curator 针对不同组件设计了不同的监听器注册和回调。

```java
// 自定义监听器 CuratorListener
client.getCuratorListenable().addListener((_fk, e) -> {
	if (e.getType().equals(CuratorEventType.WATCHED)) {
		log.info("测试");
	}

});
ConnectionStateListener connectionStateListener = (client1, newState) -> {
	//Some details
	log.info("newState => "+ newState);
};
```

可以注册的监听器方式如下：
- 一次性 Watch 通知
-  注册 CuratorListener 通知
- 注册 ConnectionStateListener 通知
- 注册 UnhandledErrorListener 通知
- 后台线程操作完成时的回调通知
- 缓存机制，多次注册

### 一次性 Watch 通知

每次都需要反复通过下面的方法重新注册。这里涉及到 NodeCache 的相关组件，由于目前并没有介绍相关的前置代码，这里暂时跳过介绍。

```java
client.checkExists().creatingParentContainersIfNeeded().usingWatcher(watcher).inBackground(backgroundCallback).forPath(path);
```

### 注册 CuratorListener 通知

实现方式很简单，就是把监听器注册到`CuratorFrameworkImpl.listeners`这个容器当中，后台线程完成操作通知该监听器容器的所有监听器。

比如异步的方式在ZK上面创建路径会触发**CuratorEventType.CREATE**事件，还有就是连接状态事件触发的时候**parentWatcher**也会回调这些listeners，比如下面的代码：

```java
/**
 * connect ZK, register watchers
 */
public CuratorFramework mkClient(Map conf, List<String> servers, Object port,
                                 String root, final WatcherCallBack watcher) {

    CuratorFramework fk = Utils.newCurator(conf, servers, port, root);

	// 自定义监听器 CuratorListener
    fk.getCuratorListenable().addListener(new CuratorListener() {
        @Override
        public void eventReceived(CuratorFramework _fk, CuratorEvent e) throws Exception {
            if (e.getType().equals(CuratorEventType.WATCHED)) {
                WatchedEvent event = e.getWatchedEvent();

                watcher.execute(event.getState(), event.getType(), event.getPath());
            }

        }
    });

    fk.start();
    return fk;
}
```

**CuratorFrameworkImpl#processEvent**

`processEvent` 方法总会进行注册的 **CuratorListener** 回调操作。

```java
private void processEvent(final CuratorEvent curatorEvent)
    {
        if ( curatorEvent.getType() == CuratorEventType.WATCHED )
        {
            validateConnection(curatorEvent.getWatchedEvent().getState());
        }

        listeners.forEach(new Function<CuratorListener, Void>()
        {
            @Override
            public Void apply(CuratorListener listener)
            {
                try
                {
                    OperationTrace trace = client.startAdvancedTracer("EventListener");
                    listener.eventReceived(CuratorFrameworkImpl.this, curatorEvent);
                    trace.commit();
                }
                catch ( Exception e )
                {
                    ThreadUtils.checkInterrupted(e);
                    logError("Event listener threw exception", e);
                }
                return null;
            }
        });
    }
```

具体回调则是有各种执行构建实现器完成的，这一块深究比较复杂，这里有个概念后续有需要查看相关实现即可。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230708111456.png)

### 注册 ConnectionStateListener 通知

如果添加 **ConnectionStateListener** 监听器，则在连接状态发生改变时，会收到通知。

```java
ConnectionStateListener connectionStateListener = new ConnectionStateListener()
    {
        @Override
        public void stateChanged(CuratorFramework client, ConnectionState newState)
        {
          //Some details
        }
    };
client.getConnectionStateListenable().addListener(connectionStateListener);
```

ConnectionStateListener 监听器的事件回调发生在**ConnectionStateManager**当中，但是前面我们只介绍了如何初始化，下面扩展介绍回调`ConnectionStateListener`的部分

#### ConnectionStateManager 如何回调 ConnectionStateListener？

**CuratorFrameworkImpl#validateConnection**

上面讲解**会话机制**的时候，提到了最后有一个添加 **ConnectionState** 的操作，这里将介绍收到 **ConnectionState** 变更之后如何回调注册在自己身上的监听器。

```java
void validateConnection(Watcher.Event.KeeperState state)  
{  
	// ......
    else if ( state == Watcher.Event.KeeperState.Expired )  
    {  
        connectionStateManager.addStateChange(ConnectionState.LOST);  
    }  
    else if ( state == Watcher.Event.KeeperState.SyncConnected )  
    {  
        unSleepBackgroundOperations();  
    }  
    else if ( state == Watcher.Event.KeeperState.ConnectedReadOnly )  
    {  
        connectionStateManager.addStateChange(ConnectionState.READ_ONLY);  
    }  
}
```

具体处理在下面这个方法中完成。

**ConnectionStateManager#processEvents**

```java
private void processEvents()
    {
        while ( state.get() == State.STARTED )
        {
            try
            {
                int useSessionTimeoutMs = getUseSessionTimeoutMs();
                long elapsedMs = startOfSuspendedEpoch == 0 ? useSessionTimeoutMs / 2 : System.currentTimeMillis() - startOfSuspendedEpoch;
                long pollMaxMs = useSessionTimeoutMs - elapsedMs;

                final ConnectionState newState = eventQueue.poll(pollMaxMs, TimeUnit.MILLISECONDS);
                if ( newState != null )
                {
                    if ( listeners.isEmpty() )
                    {
                        log.warn("There are no ConnectionStateListeners registered.");
                    }
					// 关键部分，当出现状态变更进行回调监听器通知
                    listeners.forEach(listener -> listener.stateChanged(client, newState));
                }
                else if ( sessionExpirationPercent > 0 )
                {
                    synchronized(this)
                    {
                        checkSessionExpiration();
                    }
                }
            }
            catch ( InterruptedException e )
            {
                // swallow the interrupt as it's only possible from either a background
                //  吞下中断，因为它只可能来自后台操作
                
                // operation and, thus, doesn't apply to this loop or the instance
                // is being closed in which case the while test will get it
                // 如果实例在关闭有可能走到这一块代码
            }
        }
    }
```

上面内容重要的其实就一行代码：

```java
listeners.forEach(listener -> listener.stateChanged(client, newState));
```

这个**processEvents**是怎么回调的？其实在之前画的 **CuratorFrameworkImpl** 启动过程流程图中就有展示。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230706222022.png)

**ConnectionStateManager** 当中有一个 **ExecutorService** 线程池，翻看代码可以得知他的实现是 **SingleThreadScheduledExecutor**，这里含义明显就是单独开启一个线程轮询这一段代码检查 **listener**，状态变更通知注册在 **ConnectionStateManager** 上的监听器。

### 注册 UnhandledErrorListener 通知

同理注册到`CuratorFrameworkImpl.unhandledErrorListeners`当中，当后台线程操作发生异常或者handler发生异常的时候会触发。

**注册方式**

```java

/**
 * connect ZK, register watchers
 */
public CuratorFramework mkClient(Map conf, List<String> servers, Object port,
                                 String root, final WatcherCallBack watcher) {

    CuratorFramework fk = Utils.newCurator(conf, servers, port, root);

	// 自定义监听器 UnhandledErrorListener
   fk.getUnhandledErrorListenable().addListener(new UnhandledErrorListener() {
        @Override
        public void unhandledError(String msg, Throwable error) {
            String errmsg = "Unrecoverable zookeeper error, halting process: " + msg;
            LOG.error(errmsg, error);
            JStormUtils.halt_process(1, "Unrecoverable zookeeper error");

        }
    });

   
    fk.start();
    return fk;
}

```

**如何触发？**

触发的相关代码在`CuratorFrameworkImpl#logError`方法中，注意这里的`apply`方法处理。

```java
void logError(String reason, final Throwable e)  
{  
	// 省略其他无关代码
    unhandledErrorListeners.forEach(new Function<UnhandledErrorListener, Void>()  
    {  
        @Override  
        public Void apply(UnhandledErrorListener listener)  
        {  
            listener.unhandledError(localReason, e);  
            return null;        
            }  
    });  

	// 省略无关代码
}
```

### 后台线程操作完成时的回调通知

对于不同操作比如 `setData`，可以通过链式调用的方式传入回调函数 callback，操作完成之后会执行回调函数完成回调操作。

```java
 public static void setDataAsyncWithCallback(CuratorFramework client, BackgroundCallback callback, String path, byte[] payload) throws Exception {
        // this is another method of getting notification of an async completion
        client.setData().inBackground(callback).forPath(path, payload);
    }
```

### 缓存机制，多次注册

Curator的缓存机制是一块比较大的部头，Curator 的缓存方式包括：

- Path Cache
- Node Cache 
- Tree Cache

缓存在使用之前会和服务端的节点数据进行对比，当数据不一致时，会通过watch机制触发回调刷新本地缓存，同时再次注册Watch，每次重连会注册新的 Watcher，保证 Watcher永远不丢失。

## 小结

通过通知机制和会话管理两个部分，我们了解到：
- **客户端通知**是同步完成。
- `connectionStateManager.listeners`是由**内部的线程池**做异步通知
- `CuratorFrameworkImpl.listeners` 对于连接状态的通知，与watcher通知线程为**同步**，由后台线程通知时为**异步**。
- watcher注册过多可能导致重连之后watcher丢失。


## 回顾初始化过程

Curator框架实现CuratorFrameworkImpl启动时，首先启动连接状态管理器**ConnectionStateManager**， 然后再启动客户端**CuratorZookeeperClient**。

构造Curator框架实现CuratorFrameworkImpl初始化动客户端CuratorZookeeperClient，注意在这里会默认传入一个Watcher，用于处理CuratorEvent。)。 

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230717113221.png)

CuratorZookeeperClient启动过程，关键点是在启动连接状态ConnectionState（在构造CuratorZookeeperClient，初始化连接状态，并将内部Watcher传给连接状态）。 

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230717112918.png)

连接状态实现了观察者Watcher，在连接状态建立时，调用客户端CuratorZookeeperClient传入的Watcher，处理相关事件。而这个Watcher是在现CuratorFrameworkImpl初始化动客户端CuratorZookeeperClient时 传入的。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230717113503.png)

**客户端观察者的实际处理业务逻辑在CuratorFrameworkImpl实现**，也就是`processEvent`方法，processEvent主要处理逻辑为，遍历CuratorFrameworkImpl内部的监听器容器内的监听器处理相关CuratorEvent 事件。这个CuratorEvent事件，是由原生WatchedEvent事件包装而来。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230717113536.png)

启动连接状态管理器**ConnectionStateManager**，主要是使用连接状态监听器容器（UnaryListenerManager< ConnectionStateListener>）**Listenabler**（之前版本叫 **ListenerContainer**）中的监听器。

**ConnectionStateManager**中监听器触发具体工作是消费连接状态事件队列**BlockingQueue**中事件。这里**BlockingQueue**里面存放的是ConnectionState状态变更之后【offer】的节点。

这部分又回到【注册 ConnectionStateListener 通知】部分，状态变更之后最后一段有一个`connectionStateManager.addStateChange(XXXX);  `的小动作。

```java
void validateConnection(Watcher.Event.KeeperState state)  
{  
	// ......
    else if ( state == Watcher.Event.KeeperState.Expired )  
    {  
        connectionStateManager.addStateChange(ConnectionState.LOST);  
    }  
    else if ( state == Watcher.Event.KeeperState.SyncConnected )  
    {  
        unSleepBackgroundOperations();  
    }  
    else if ( state == Watcher.Event.KeeperState.ConnectedReadOnly )  
    {  
        connectionStateManager.addStateChange(ConnectionState.READ_ONLY);  
    }  
}
```

通过代码下探，最终回到下面的部分：

```java
 private void postState(ConnectionState state)
    {
        log.info("State change: " + state);

        notifyAll();

        while ( !eventQueue.offer(state) )
        {
            eventQueue.poll();
            log.warn("ConnectionStateManager queue full - dropping events to make room");
        }
    }
```

> @since 4.2.0 return type has changed from ListenerContainer to Listenable


# 写到最后

本节介绍了Curator的基础使用，从源码角度分析了Curator 组件的初始化过程，并且简单分析会话管理和通知机制的相关源码调用。

下面是本文涉及到的源码讲解汇总的一副总图。个人源码分析过程如果有存在错误或者疑问欢迎反馈和讨论。

![Curator 源码分析.drawio.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/Curator%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.drawio.png)

最后是整个demo代码：

```java

@Slf4j
public class CuratorTestExample {

    public static void main(String[] args) throws Exception {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        CuratorFramework client =
                CuratorFrameworkFactory.newClient("192.168.19.100:2181,192.168.19.101:2181,192.168.19.102:2181", retryPolicy);

        // 连接ZK,开启连接

        // 自定义监听器 CuratorListener
        client.getCuratorListenable().addListener((_fk, e) -> {
            if (e.getType().equals(CuratorEventType.WATCHED)) {
                log.info("测试");
            }

        });
        ConnectionStateListener connectionStateListener = (client1, newState) -> {
            //Some details
            log.info("newState => "+ newState);
        };
        // 11:31:17.026 [Curator-ConnectionStateManager-0] INFO com.zxd.interview.zkcurator.CuratorTestExample - newState => CONNECTED
        client.getConnectionStateListenable().addListener(connectionStateListener);
        client.start();
        // 此处就获取到 zk的一个连接实例。
        //.....
        // 创建znode，如果有必要需要创建父目录
        client.create().creatingParentsIfNeeded().withProtection().forPath("/my/path", "Test".getBytes());
        InterProcessMutex lock = new InterProcessMutex(client, "/my/path");
        lock.acquire();
        try {
            // do some work inside of the critical section here
            Thread.sleep(1000);
        } finally {
            lock.release();
        }

    }
}
```

# 推荐阅读

[ZK客户端Curator使用详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/135334760)

[https://cloud.tencent.com/developer/article/1648976?areaSource=106005.14](https://cloud.tencent.com/developer/article/1648976?areaSource=106005.14)

[Curator目录监听 | Ravitn Blog (donaldhan.github.io)](https://donaldhan.github.io/zookeeper/2018/06/29/curator%E7%9B%AE%E5%BD%95%E7%9B%91%E5%90%AC.html)