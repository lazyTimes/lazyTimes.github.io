---
title: 《跟闪电侠学Netty》阅读笔记 - ChannelHandler 生命周期
subtitle: 生命周期相关方法
author: lazytime
url_suffix: netty_live
tags:
  - Netty
categories:
  - Netty
keywords: ChannelInboundHandler生命周期
description: Netty
copyright: true
date: 2023-07-09 21:29:04
---

# 引言

本文主要介绍ChannelHandler当中的ChannelInboundHandler。

# 思维导图

[https://www.mubu.com/doc/1lK922R14Bl](https://www.mubu.com/doc/1lK922R14Bl)

![《跟闪电侠学Netty》- ChannelHandler 生命周期.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/%E3%80%8A%E8%B7%9F%E9%97%AA%E7%94%B5%E4%BE%A0%E5%AD%A6Netty%E3%80%8B-%20ChannelHandler%20%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

<!-- more-->

# LifeCycleTestHandler 案例

首先来看一下案例，LifeCycleTestHandlerTest 利用适配器 **ChannelInboundHandlerAdapter** 重写，重写相关方法。

- public void handlerAdded(ChannelHandlerContext ctx) throws Exception  
- public void channelRegistered(ChannelHandlerContext ctx) throws Exception  
- public void channelActive(ChannelHandlerContext ctx) throws Exception  
- public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception  
- public void channelReadComplete(ChannelHandlerContext ctx) throws Exception  
- public void channelInactive(ChannelHandlerContext ctx) throws Exception  
- public void channelUnregistered(ChannelHandlerContext ctx) throws Exception  
- public void handlerRemoved(ChannelHandlerContext ctx) throws Exception

```java

/**
 * 测试netty channelHandler 的生命周期
 *
 * 对应的是 ChannelInboundHandlerAdapter
 */
@Slf4j
public class LifeCycleTestHandlerTest extends ChannelInboundHandlerAdapter {

    public static void main(String[] args) {
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        NioEventLoopGroup boss = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();

        serverBootstrap
                .group(boss, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringDecoder());
                        ch.pipeline().addLast(new LifeCycleTestHandlerTest());

                    }

                })
                .bind(8001);

    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        log.info("逻辑处理器被添加 handlerAdded() ");
        super.handlerAdded(ctx);
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        log.info("channel 绑定到线程 (NioEventLoop)：channelRegistered() ");
        super.channelRegistered(ctx);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        log.info("channel 准备就绪 channelActive()");
        super.channelActive(ctx);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        log.info("channel 某次数据读写完成 channelReadComplete() ");
        super.channelReadComplete(ctx);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        log.info("channel 有数据可读 channelRead() ");
        super.channelRead(ctx, msg);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        log.info("channel 被关闭 channelInactive()");
        super.channelInactive(ctx);
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        log.info("channel取消线程 (NioEventLoop) 的绑定：channelUnregistered() ");
        super.channelUnregistered(ctx);
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        log.info("逻辑处理器被删除 handlerRemoved() ");
        super.handlerRemoved(ctx);
    }
}
```

如果启动Client客户端并且进行连接，**ChannelHanlder** 的回调方法执行顺序如下：

1. `handlerAdded()`
2. `channelRegistered()`
3. `channelActive()`
4. `channelRead()`
5. `channelReadComplete()`

## Channel连接方法回调

根据上面的执行顺序，这里补充介绍回调方法的含义。

1. 逻辑处理器被添加：`handlerAdded()  `

	检测到新连接，调用`ch.pipleLine(new LifeCycleTestHandlerTest())` 之后的回调，表示**当前的Channel成功绑定一个逻辑处理器**。
   
2. channel 绑定到线程 (NioEventLoop)：`channelRegistered()  `

    表示当前Channel所有逻辑处理器和某个线程模型的线程绑定完成，比如绑定到NIO模型的NioEventLoop。此时会创建线程处理连接的读写，具体使用了JDK的线程池实现，通过线程池抓一个线程方式绑定。
    
3. channel 准备就绪： `channelActive()`  
   
    触发条件是所有的业务逻辑准备完成，也就是pipeline添加了所有的Handler ，以及绑定好一个NIO的线程之后，相当于是worker全部就位，就等boss一声令下开工。
    
4. channel 有数据可读 `channelRead()`

	客户端每次发数据都会以此进行回调。
   
5. channel 某次数据读写完成 `channelReadComplete()`

	服务端每次处理以此数据都回调此方法进行回调 。


# Channel 关闭方法回调

 1. channel 被关闭：`channelInactive() ` 

	TCP层已经转为非ESTABLISH模式。
    
 2. channel取消线程 (NioEventLoop) 的绑定：`channelUnregistered()`

	与连接线程的绑定全部取消时候触发。
    
3. 逻辑处理器被删除：`handlerRemoved()`

	移除连接对应的所有业务逻辑处理器触发。


# Channel连接和关闭回调流程图

![Channel连接和关闭回调流程图](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230709201836.png)


# ChannelHanlder用法举例

## ChannelInitializer

入门程序当中的 `.childHandler(new ChannelInitializer<NioSocketChannel>()`定义了抽象initChannel方法，抽象方法需要自行实现，通常是服务端启动流程的逻辑处理器中使用，添加Handler链到Pipeline。

`handlerAdded()` 方法和 `channelRegistered()`方法都会尝试调用 initChannel，initChannel被多次调用，利用putIfAbsent防止initChannel被多次调用。

```java
if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) { // Guard against re-entrance.  
```

> 思考：为什么既要防止被多次调用，Netty又要调用多次`initChannel`？  
	1. handlerAdded方法已经通过`Channel`进行绑定。
	2. 用户自行覆盖和重写`ChannelInitializer`的`handlerAdded`，导致`Channel`不触发绑定  
	3. `channelRegisted()` 中再触发一次绑定，并且本身不允许被重写。  
	`public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {`  
	4. 引导实现者只关注重写handlerAdded之后的逻辑处理  

## handlerAdded 与 channelInActive  

主要用做资源释放和申请。  
        
## channelActive 和 ChannelInActive  

应用的场景如下：

1. TCP建立和连接抽象  
2. 实现统计单机的连接数量  
	- 其中channelActive被调用，对应的每次调用连接+1
	- 其中channelInActive被调用，对应的每次调用连接-1
   
3. 实现客户端IP黑白名单的连接过滤  
        
## channelRead()  

这个方法和粘包和拆包问题相关，服务端根据自定义协议拆包，在这个方法中每次读取一定数据，都会累加到容器当中。  
        
## channelReadComplete  

每次向客户端写入数据都调用`writeAndFlush`不是很高效，  更好的实现是向客户端写数据只`write`，但是在`ChannelReadComplete`里面一次 **flush** ，所以这个方法相当于批量刷新机制，批量刷新追求更高性能。

# 总结

1. 本部分主要是介绍ChannelHandler的各种回调，以及连接建立关闭，执行回调是一个逆向过程。
2. 每一种回调都有各自用法，但是部分回调界限比较模糊，更多需要在实践中区分和使用。

# 写到最后

生命周期和回调在Netty中非常直观，本文更多是对于重要的方法进行罗列。