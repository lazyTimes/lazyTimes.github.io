---
title: 《跟闪电侠学Netty》阅读笔记 - 聊天系统实现
subtitle: 聊天系统重要实现
author: lazytime
url_suffix: nettychatimpl
tags:
  - Netty
categories:
  - Netty
keywords: Netty,聊天
description: Netty聊天系统
copyright: true
date: 2023-07-11 16:50:48
---
# 引言

本部分整合聊天系统有关的章节，内容主要是介绍关键功能的实现逻辑和部分代码实现为主，建议读者先看看作者的博客项目，切换到不同的分支看看各个细节功能如何实现。这里仅仅记录一些个人学习过程的重点部分。

# 思维导图

[https://www.mubu.com/doc/1dunN_7Luzl](https://www.mubu.com/doc/1dunN_7Luzl)

![《跟闪电侠学Netty》 - 聊天系统实现.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/%E3%80%8A%E8%B7%9F%E9%97%AA%E7%94%B5%E4%BE%A0%E5%AD%A6Netty%E3%80%8B%20-%20%E8%81%8A%E5%A4%A9%E7%B3%BB%E7%BB%9F%E5%AE%9E%E7%8E%B0.png)


# 项目代码

作者的仓库代码地址：[https://github.com/lightningMan/flash-netty](https://github.com/lightningMan/flash-netty)

<!-- more -->

# 通信协议设计和自定义编解码实现

## 什么是通信协议？  

基于TCP通信均为二进制协议，底层都是通过字节进行传输的。在通信协议当中规定数据传输的每一个字节含义。
    
## 通信过程  

1. 客户端转换数据为二进制。  
2. 网络传输给服务端。
3. 服务端根据协议规则读取二进制数据。  
4. 服务端处理数据返回响应结果给客户端。  

## 聊天系统的通信协议数据对象设计

在聊天系统当中通信协议的设计如下。
    
### 4字节魔数  

比如Java的字节码`CafeBabe`，用于快速识别是否自定义协议，也可以方便快速提取数据。

```java
public static final int MAGIC_NUMBER = 0x12345678;
```

### 1 字节版本号  

类似TCP的IPV4还是IPV6。

```java
/**  
 * 协议版本  
 */  
@JSONField(deserialize = false, serialize = false)  
private Byte version = 1;
```

### 1 字节序列化算法  

使用1个字节来标识算法。

```java
/**  
 * 序列化算法定义  
 */  
public interface SerializerAlgorithm {  
    /**  
     * json 序列化  
     */  
    byte JSON = 1;  
}
```

### 1 字节指令  

一个字节最多表示256种指令。注意在设计上指令和版本号进行绑定关联，实现不同版本之间的指令兼容，提高程序的健壮性。

```java
@Data  
public abstract class Packet {  
    /**  
     * 协议版本  
     */  
    @JSONField(deserialize = false, serialize = false)  
    private Byte version = 1;  
  
  
    @JSONField(serialize = false)  
    public abstract Byte getCommand();  
}
```

### 4字节数据长度  

数据长度是必要的，主要用于字节流这种连续不断的数据形式进行切割。

```java
byteBuf.writeInt(bytes.length);
```

> int 基本数据类型在Java中默认占4个字节，这4个字节用来存储字节数组的长度。

### N字节数据  

数据部分。

## 如何实现JAVA对象二进制互相转化？

所谓互转对应了网络 Socket IO 的`input/output`中的数据转化部分，实体数据转为字节流这个过程我们通常叫做**编码**，反之则是解码。

无论是编码还是解码，都是依赖Netty自定义的 **MessageToMessageCodec**实现。聊天系统的编码和解码工作都是依赖 **PacketCodecHandler** 完成的。 

```java
@ChannelHandler.Sharable  
public class PacketCodecHandler extends MessageToMessageCodec<ByteBuf, Packet> {  
    public static final PacketCodecHandler INSTANCE = new PacketCodecHandler();  
  
    private PacketCodecHandler() {  
  
    }  
  
    @Override  
    protected void decode(ChannelHandlerContext ctx, ByteBuf byteBuf, List<Object> out) {  
        out.add(PacketCodec.INSTANCE.decode(byteBuf));  
    }  
  
    @Override  
    protected void encode(ChannelHandlerContext ctx, Packet packet, List<Object> out) {  
        ByteBuf byteBuf = ctx.channel().alloc().ioBuffer();  
        PacketCodec.INSTANCE.encode(byteBuf, packet);  
        out.add(byteBuf);  
    }  
}
```

自定义逻辑处理器，在 Netty Server 中需要注册到 **pipeline** 当中。

```java
public static void main(String[] args) {  
    NioEventLoopGroup boosGroup = new NioEventLoopGroup();  
    NioEventLoopGroup workerGroup = new NioEventLoopGroup();  
  
    final ServerBootstrap serverBootstrap = new ServerBootstrap();  
    serverBootstrap  
            .group(boosGroup, workerGroup)  
            .channel(NioServerSocketChannel.class)  
            .option(ChannelOption.SO_BACKLOG, 1024)  
            .childOption(ChannelOption.SO_KEEPALIVE, true)  
            .childOption(ChannelOption.TCP_NODELAY, true)  
            .childHandler(new ChannelInitializer<NioSocketChannel>() {  
                protected void initChannel(NioSocketChannel ch) {  
	                //......
                    ch.pipeline().addLast(PacketCodecHandler.INSTANCE); 
		            // ......
                }  
            });  
  
  
    bind(serverBootstrap, PORT);  
}
```

这里解释下为什么`PacketCodecHandler`要被注解标记为“**Sharable**”，因为编码和解码可能在多个`handler`中用到，为了提高效率，这里通过共享减少实例的创建。

> 下文也会介绍这个单例模式的优化点。

带着疑问我们再看看`@ChannelHandler.Sharable`这个注解的源码解释。

> Indicates that the same instance of the annotated ChannelHandler can be added to one or more ChannelPipelines multiple times without a race condition.
	If this annotation is not specified, you have to create a new handler instance every time you add it to a pipeline because it has unshared state such as member variables.
	This annotation is provided for documentation purpose, just like the JCIP annotations 

上面的内容翻译过来就是：

被注解的`Sharable`的同一个**ChannelHandler**实例，可以被多次添加到一个或多个`ChannelPipeline`中，并且可以确保不会出现竞争情况。如果没有指定这个注解，那么每次就创建新的Channel都需要使用新的Handler实例。在有不共享的状态，如成员变量时候，就不能用这个注解。

简单来说`@ChannelHandler.Sharable`实现了Netty中的"Bean"单例和共享。

## 实战部分  

### 数据编码过程（思路）  

下面是数据解码的基本编写思路。

1. 添加编码器。

```java
ch.pipeline().addLast(new PacketEncoder());
```

2. 往`ByteBuf`逐个写字段，实现编码过程。

```java
public class PacketEncoder extends MessageToByteEncoder<Packet> {  
  
    @Override  
    protected void encode(ChannelHandlerContext ctx, Packet packet, ByteBuf out) {  
        PacketCodec.INSTANCE.encode(out, packet);  
    }  
}
```

3. 完整的自定义协议：**PacketCodec#encode**。

```java
public void encode(ByteBuf byteBuf, Packet packet) {  
    // 1. 序列化 java 对象  
    byte[] bytes = Serializer.DEFAULT.serialize(packet);  
  
    // 2. 实际编码过程  
    byteBuf.writeInt(MAGIC_NUMBER);  
    byteBuf.writeByte(packet.getVersion());  
    byteBuf.writeByte(Serializer.DEFAULT.getSerializerAlgorithm());  
    byteBuf.writeByte(packet.getCommand());  
    byteBuf.writeInt(bytes.length);  
    byteBuf.writeBytes(bytes);  
}
```

### 解码数据过程（思路）  

下面是数据解码的基本编写思路：

1. 在handler当中添加自定义逻辑处理器。

```java
.handler(new ChannelInitializer<SocketChannel>() {  
    @Override  
    public void initChannel(SocketChannel ch) {
		ch.pipeline().addLast(new PacketDecoder());
	}  
});
```

2. 定义解码逻辑处理器。

```java
public class PacketDecoder extends MessageToMessageDecoder<ByteBuf> {  
  
    @Override  
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List out) {  
        out.add(PacketCodec.INSTANCE.decode(in));  
    }  
}
```

下面定义具体的解码过程：
- 跳过魔数。
- 跳过协议版本号  。
- 读取序列化算法。
- 读取指令，数据包，算法标识等自定义协议的基本内容。
- 根据数据长度。
- 取出数据。

**PacketCodec#decode**

```java
public Packet decode(ByteBuf byteBuf) {  
    // 跳过 magic number    byteBuf.skipBytes(4);  
  
    // 跳过版本号  
    byteBuf.skipBytes(1);  
  
    // 序列化算法  
    byte serializeAlgorithm = byteBuf.readByte();  
  
    // 指令  
    byte command = byteBuf.readByte();  
  
    // 数据包长度  
    int length = byteBuf.readInt();  
  
    byte[] bytes = new byte[length];  
    byteBuf.readBytes(bytes);  
  
    Class<? extends Packet> requestType = getRequestType(command);  
    Serializer serializer = getSerializer(serializeAlgorithm);  
  
    if (requestType != null && serializer != null) {  
        return serializer.deserialize(requestType, bytes);  
    }  
  
    return null;  
}
```


## 思考  

### JSON序列化方式之外其他序列化方式如何实现？  

#### Java原生序列化  

- 类实现 Serializable 接口  
- 具体底层由ObjectOutputStream和ObjectInputStream实现  
        
#### Hessian  

- Hessian 是动态类型、二进制、紧凑的，并且可跨语言移植的一种序列化框架  
- Hessian 协议要比 JDK、JSON 更加紧凑，性能上要比 JDK、JSON 序列化高效很多，而且生成的字节数也更小  
        
#### Protobuf  

- 谷歌实现的混合语言数据标准  
- 轻便、高效的结构化数据存储格式  
- 支持 Java、Python、C++、Go 等语言  
- 要求定义 IDL（Interface description language），并且使用对应语言的IDL生成序列化工具类
        
#### Thrift  

- Facebook于2007年开发的跨语言的rpc服框架  
- 通过Thrift的编译环境生成各种语言类型的接口文件

### 序列化和编码都是JAVA对象封装二进制过程，两者的联系和区别

总结起来就是一句话：**序列化是目标，编码是方法**。网上有一张图非常直观的展示了两者的区别。

![序列化和编码都是JAVA对象封装二进制过程，两者的联系和区别](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230629143232.png)

#### 两者的联系和区别

**编码**：信息从一种形式或格式转换为另一种形式的过程，目的是方便传输协议通信。

**序列化**：“序列化”其实本身也是“信息从一种形式或格式转换为另一种形式的过程”，只不过这个表现形式直观具体，序列化也常常用于表达一个对象的状态。

## 聊天系统的Netty细节优化

优化部分是聊天系统的精髓，也是使用Netty实践非常有价值的指导和参考，所以优先把优化部分放到前面介绍。

## 1. 使用共享Handler

### 问题分析

在旧版本代码中，每个新连接每次通过 **ChannelInitializer** 调用，都会产生9个指令对象都被new一遍操作，但是可以看到其实很多处理器内部是没有任何 "状态"的，**对于无状态的业务处理器就可以使用单例模式封装**。

```java
serverBootstrap
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    protected void initChannel(NioSocketChannel ch) {
                        ch.pipeline().addLast(new Spliter());
                        ch.pipeline().addLast(new PacketDecoder());
                        ch.pipeline().addLast(new LoginRequestHandler());
                        ch.pipeline().addLast(new AuthHandler());
                        ch.pipeline().addLast(new MessageRequestHandler());
                        ch.pipeline().addLast(new CreateGroupRequestHandler());
                        ch.pipeline().addLast(new JoinGroupRequestHandler());
                        ch.pipeline().addLast(new QuitGroupRequestHandler());
                        ch.pipeline().addLast(new ListGroupMembersRequestHandler());
                        ch.pipeline().addLast(new GroupMessageRequestHandler());
                        ch.pipeline().addLast(new LogoutRequestHandler());
                        ch.pipeline().addLast(new PacketEncoder());
                    }
                });

```

### 优化手段

- 通过加入注解 `@ChannelHandler.Shareble`，表示这个 handler 是支持多个 channel 共享的，否则会报错。
- 发布静态**final**的**不可变对象**来实现单例，编译器优化。  
- 最后还可以压缩Handler，把编码和解码过程放到一个Handler和公用的Handler放到一个Handller处理（比如请求指令分发解析处理）。

### 注意事项

主要的注意事项如下：
- 并不是所有的Handler都可以单例  
- **Spliter** 不是单例的，因为它需要对每个数据做拆包处理。

## 2. 缩短事件传播路径

### 问题分析

- 首先，指令的**decode**必须要在最前面前面，因为涉及后面的命令解析，所以这个Handler是无法“压缩”的。
- 但是如果把每个命令**decode**之后再传播到每个命令事件，但是对应的事件又不做任何处理，那么会浪费很多次多余的命令判断。
- 根本目的：**缩短事件传播链条**，事件传播链尽可能短。

### 优化手段

优化手段实际上也很简单，那就是 **使用统一Handler**。

通常的做法如下：

1. 该Handler只做判断，不做任何状态存储，使用单例优化。

```java
public static final IMHandler INSTANCE = new IMHandler();
```

2. 聊天系统中利用HashMap存储所有的命令处理Handler，这里个人顺带指定下初始化大小优化一下。

```java
private IMHandler() {  
    handlerMap = new HashMap<>(7);  
  
    handlerMap.put(MESSAGE_REQUEST, MessageRequestHandler.INSTANCE);  
    handlerMap.put(CREATE_GROUP_REQUEST, CreateGroupRequestHandler.INSTANCE);  
    handlerMap.put(JOIN_GROUP_REQUEST, JoinGroupRequestHandler.INSTANCE);  
    handlerMap.put(QUIT_GROUP_REQUEST, QuitGroupRequestHandler.INSTANCE);  
    handlerMap.put(LIST_GROUP_MEMBERS_REQUEST, ListGroupMembersRequestHandler.INSTANCE);  
    handlerMap.put(GROUP_MESSAGE_REQUEST, GroupMessageRequestHandler.INSTANCE);  
    handlerMap.put(LOGOUT_REQUEST, LogoutRequestHandler.INSTANCE);  
}
```

3. 回调`channelRead0` 实际上就是委托给map中的元素对应的指令处理器处理。

```java
@Override  
protected void channelRead0(ChannelHandlerContext ctx, Packet packet) throws Exception {  
    handlerMap.get(packet.getCommand()).channelRead(ctx, packet);  
}
```

通过一个统一的处理器包括多个静态单例处理器，有效减少JVM内存开销，单例也可以减少对象实例化的开销。

## 3. 事件传播源调整

### 关键点

如果你的 outBound 类型的 handler 较多，在写数据的时候能用 `ctx.writeAndFlush()` 就用这个方法， 不要用 `ctx.channel().writeAndFlush()`。

### 原因

究其原因是**ctx.writeAndFlush()** 会绕过所有不需要处理的其他Outbound类型。`ctx.writeAndFlush()` 是从 pipeline 链中的**当前节点开始往前找到第一个 outBound 类型向前传播**的，如果这个对象不需要其他outBound的handler处理就可以用这个方法。

![ctx.writeAndFlush()](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230710214033.png)

而**ctx.channel().writeAndFlush()** 表现则不同，它是从pipeline 链中的**最后一个** outBound 类型的 handler 开始，把对象往前进行传播，从图中就可以看到， outBound 的处理器越多，就会产生越多“无用”操作。

当然如果确定后面的 outBound 需要如此处理，那么就可以用这个方法。

![ctx.channel().writeAndFlush()](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230710214215.png)

### 相关问题

- writeAndFlush为什么可以缩短事件传播路径？  
- 它是如何实现OutBound类型的事件传播缩短的?

## 4. 减少阻塞主线程的操作【重要】

Netty中容易被忽视，却是非常重要的一个概念。那就是 **一个Channel的其中一个Handler阻塞，会导致所有其他绑定的Channel一起被拖慢**。

比如只要有一个 `channel `的一个 `handler` 中的 `channelRead0()` 方法阻塞了 NIO 线程，最终都会拖慢绑定在该 NIO 线程上的其他所有的 channel

```java
List<Channel> channelList = 已有数据可读的 channel
for (Channel channel in channelist) {
   for (ChannelHandler handler in channel.pipeline()) {
       handler.channelRead0();
   }   
}
```

上面的操作如果for循环某次出现卡顿，这不仅仅拖慢一个客户端，而是拖慢所有客户端。

所以Netty进行客户端处理的时候一般设计为非阻塞模式，或者会使用 **业务线程池** 去预防这种情况。业务线程池的实现方式更为常见，也就是Netty中一套线程池，实际处理过程中再委派给自定义的业务线程池单开线程处理。这样就实现了非阻塞异步执行任务的目的。

需要注意引入业务线程池会增加系统复杂度，也会增加线上调试难度，所以做好链路追踪十分重要。

## 5. 如何准确统计时长？

错误做法：在一个线程的头尾加入时间差计算得出执行时长结果。

正确做法：使用**writeAndFlush+addListener** 的方式判断 `futrue.isDone` 之后才计算  
        
原因：**writeAndFlush** 在非NIO线程中它是一个异步操作，其他操作由第一个任务队列异步执行。 

关键点：**writeAndFlush** 真正执行完成才算是完成处理，监听它完成处理的回调动作才能算出准确执行时长。

## 优化小结

- 如果Handler多例但是无状态，完全可以改为单例模式 。
- 尽可能减少Handler的臃肿，防止调用链路过长。
- Handler的耗时操作要交给线程池开启新线程处理，一个耗时操作不只影响单个Channel。建议业务线程池单独开新线程方式优化，但是需要注意和线程绑定的相关参数处理问题 。
- 耗时统计，writeAndFlush属于异步任务。


# 实现登录

## 处理流程图

![处理流程图](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230711112329.png)

## 实现思路

- 目标客户端和服务端分别启动Netty服务。
- 客户端发送登录请求指令，服务端解码之后根据传输结果校验，根据校验结果构建登录请求响应指令LoginResponsePacket。
- 通过ctx.writeAndFlush(loginResponsePacket); 回送响应结果给客户端。
	- 登录校验成功，通过SessionUtil添加session信息  
- 客户端登录成功之后，构建请求指令对象，设置参数，通过Netty发送到服务端 。
- 服务端收到请求进行验证，并且构建相对应的响应指令结果对象。

## 实现步骤

下面是大致的实现步骤：

1. 添加 **LoginRequestHandler** 登录逻辑处理器在Server端。

```java
ch.pipeline().addLast(LoginRequestHandler.INSTANCE);
```

```java
  
@ChannelHandler.Sharable  
public class LoginRequestHandler extends SimpleChannelInboundHandler<LoginRequestPacket> {  
  
    public static final LoginRequestHandler INSTANCE = new LoginRequestHandler();  
  
    protected LoginRequestHandler() {  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, LoginRequestPacket loginRequestPacket) {  
        LoginResponsePacket loginResponsePacket = new LoginResponsePacket();  
        loginResponsePacket.setVersion(loginRequestPacket.getVersion());  
        loginResponsePacket.setUserName(loginRequestPacket.getUserName());  
  
        if (valid(loginRequestPacket)) {  
            loginResponsePacket.setSuccess(true);  
            String userId = IDUtil.randomId();  
            loginResponsePacket.setUserId(userId);  
            System.out.println("[" + loginRequestPacket.getUserName() + "]登录成功");  
            SessionUtil.bindSession(new Session(userId, loginRequestPacket.getUserName()), ctx.channel());  
        } else {  
            loginResponsePacket.setReason("账号密码校验失败");  
            loginResponsePacket.setSuccess(false);  
            System.out.println(new Date() + ": 登录失败!");  
        }  
  
        // 登录响应  
        ctx.writeAndFlush(loginResponsePacket);  
    }  
  
    private boolean valid(LoginRequestPacket loginRequestPacket) {  
        return true;  
    }  
  
    @Override  
    public void channelInactive(ChannelHandlerContext ctx) {  
        SessionUtil.unBindSession(ctx.channel());  
    }  
}
```

2. 在客户端同样添加**Handler**，`LoginResponseHandler`，LoginResponseHandler的处理逻辑如下。

```java
ch.pipeline().addLast(LoginResponseHandler.INSTANCE);
```

```java
public class LoginResponseHandler extends SimpleChannelInboundHandler<LoginResponsePacket> {  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, LoginResponsePacket loginResponsePacket) {  
        String userId = loginResponsePacket.getUserId();  
        String userName = loginResponsePacket.getUserName();  
  
        if (loginResponsePacket.isSuccess()) {  
            System.out.println("[" + userName + "]登录成功，userId 为: " + loginResponsePacket.getUserId());  
            SessionUtil.bindSession(new Session(userId, userName), ctx.channel());  
        } else {  
            System.out.println("[" + userName + "]登录失败，原因：" + loginResponsePacket.getReason());  
        }  
    }  
  
    @Override  
    public void channelInactive(ChannelHandlerContext ctx) {  
        System.out.println("客户端连接被关闭!");  
    }  
}
```

客户端登录成功或者失败，如何把失败或者成功标识绑定在客户端连接？ 服务端如何高效判定客户端重新登录？

在聊天系统中实现比较简单粗暴。服务端高效判断的方法是在`ConcurrentHashMap`，Map当中存储用户的ID，如果登录成功则存储到此Map中，服务端也只需要判断Map元素即可高效判断是否登录。

```java
private static final Map<String, Channel> userIdChannelMap = new ConcurrentHashMap<>();
```


## 热插拔客户端是否登录验证

首先校验是否登录部分封装到工具类当中，实现比较简单。

**SessionUtil**

```java
public static boolean hasLogin(Channel channel) {  
  
    return getSession(channel) != null;  
}  
  
public static Session getSession(Channel channel) {  
  
    return channel.attr(Attributes.SESSION).get();  
}

// AttributeKey<Session> SESSION = AttributeKey.newInstance("session");
```

**AuthHandler**

实现热插拔的思路是**判断是否登录，统一通过该调用链条完成**，AuthHandler本身作为**单独处理器**封装判断登录校验逻辑。

```java
@Overridepublic void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
if (!SessionUtil.hasLogin(ctx.channel())) {
        ctx.channel().close();    
	} else {
        ctx.pipeline().remove(this);        
        super.channelRead(ctx, msg);    
	}
}
```

# 实现双端收发消息

## 客户端处理

客户端成功登录之后，下一步是实现客户端和服务端互相发送数据。客户端收消息处理器如下：

```java
// 收消息处理器  
ch.pipeline().addLast(new MessageResponseHandler());
```

**MessageResponseHandler**

```java
public class MessageResponseHandler extends SimpleChannelInboundHandler<MessageResponsePacket> {  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, MessageResponsePacket messageResponsePacket) {  
        String fromUserId = messageResponsePacket.getFromUserId();  
        String fromUserName = messageResponsePacket.getFromUserName();  
        System.out.println(fromUserId + ":" + fromUserName + " -> " + messageResponsePacket  
                .getMessage());  
    }  
}
```

## 服务端处理

因为是通用组件，服务端这里封装到 **IMHandler** 通用组件当中。

```java
handlerMap.put(MESSAGE_REQUEST, MessageRequestHandler.INSTANCE);
```

**MessageRequestHandler**

```java
@ChannelHandler.Sharable  
public class MessageRequestHandler extends SimpleChannelInboundHandler<MessageRequestPacket> {  
    public static final MessageRequestHandler INSTANCE = new MessageRequestHandler();  
  
    private MessageRequestHandler() {  
  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, MessageRequestPacket messageRequestPacket) {  
        long begin = System.currentTimeMillis();  
  
  
        // 1.拿到消息发送方的会话信息  
        Session session = SessionUtil.getSession(ctx.channel());  
  
        // 2.通过消息发送方的会话信息构造要发送的消息  
        MessageResponsePacket messageResponsePacket = new MessageResponsePacket();  
        messageResponsePacket.setFromUserId(session.getUserId());  
        messageResponsePacket.setFromUserName(session.getUserName());  
        messageResponsePacket.setMessage(messageRequestPacket.getMessage());  
  
        // 3.拿到消息接收方的 channel        Channel toUserChannel = SessionUtil.getChannel(messageRequestPacket.getToUserId());  
  
        // 4.将消息发送给消息接收方  
        if (toUserChannel != null && SessionUtil.hasLogin(toUserChannel)) {  
            toUserChannel.writeAndFlush(messageResponsePacket).addListener(future -> {  
                if (future.isDone()) {  
  
                }  
  
            });  
        } else {  
            System.err.println("[" + session.getUserId() + "] 不在线，发送失败!");  
  
        }  
    }  
}
```

## 小结

实现双端收发消息小结内容如下：

1. 定义收发消息Java对象，对于消息进行收发。
2. 学习 `Channel` 的 `attr` 的实际用法，可以给Channel绑定属性并且设置某些状态，内部实际也是通过Map维护的，所以不需要用户外部自己在自定义去维护。
3. 如何在控制台当中获取消息并且发送到服务端。
4. 服务端回传消息给客户端。

# ChannelPipleline 和 ChannelHandler 概念

本部分是补充部分。主要介绍 Pipeline 和ChannelHanlder构成和一些基础概念。理解这一点之前需要先理解Channel这个概念。

## ChannelPipleline 和 ChannelHandler 构成图

![ChannelPipleline 和 ChannelHandler 构成图](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230711121733.png)

## Channel 概念理解

**一个客户端连接对应一个Channel，这个Channel可以类比BIO当中的传统概念Socket套接字。**

> A nexus to a network socket or a component which is capable of I/O operations such as read, write, connect, and bind.

一个网络套接字的节点或一个能够进行（网络）I/O操作的组件，如读、写、连接和绑定。

## ChannelPipeline 

源码对于 ChannelPipeline 的定义如下：

> A list of ChannelHandlers which handles or intercepts inbound events and outbound operations of a Channel. ChannelPipeline implements an advanced form of the Intercepting Filter pattern to give a user full control over how an event is handled and how the ChannelHandlers in a pipeline interact with each other.

源码中还有一个直观的设计图。

下图描述了I/O事件在ChannelPipeline中是如何被ChannelHandlers处理的。一个I/O事件由 ChannelInboundHandler 或 ChannelOutboundHandler处理，并通过调用ChannelHandlerContext中定义的事件传播方法，如ChannelHandlerContext.fireChannelRead(Object)和ChannelHandlerContext.write(Object)，转发给其最接近的处理程序。

![ChannelInboundHandler](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230629165658.png)

`ChannelPipeline` 的核心如下：

- 处理或拦截一个Channel的入站事件和出站操作的链表。
- 通过责任链模式的设计，可以完全自定义处理逻辑和`ChannelHandler`之间互相通信的逻辑。

## ChannelContext

**ChannelHandler与Channel和ChannelPipeline之间的映射关系**，由`ChannelHandlerContext`进⾏维护，根据其名称Context也可以看到存储更为丰富的信息。

> Enables a ChannelHandler to interact with its ChannelPipeline and other handlers.

使得ChannelHandler能够与它的ChannelPipeline和其他处理程序互动。

- `ChannelContext`可以获取整个`Channel`的信息。
- 获取所有的上下文。
- 逻辑处理器ChannelHandler定义处理逻辑。

## ChannelHanlder

`ChannelHanlder` 包含两种理解。

第一种：可以理解为socket连接，客户端和服务端连接的时候会创建一个channel。 负责基本的IO操作，例如：`bind()`、`connect()`、`read()`、`write()`。 

第二种：Netty的Channel接口所提供的API，大大减少了Socket类复杂性。

因为Channel连接过程中存在双端 `input/output`，所以 `ChannelHandler` 也分类为 `ChannelInboundHandler` 和 `ChannelOutboundHandler`。

### ChannelInboundHandler

- 读取的逻辑抽象 。 
- `channelRead` 是最重要的方法 。
- 配合`ByteBuf`使用进行`buf.read`推进读指针移动 。

### ChannelOutboundHandler

- 对应写出的逻辑抽象 。 
- 核心方法是 `write`，`writeAndFlush` 。 

### 适配器

在使用过程中还存在对应的适配器。

- `ChannelOutboundHandlerAdapter`（注意处理顺序和添加addLast的顺序相反）
- `ChannelInboundHandlerAdapter`

# 客户端和服务端的 SimpleChannelInboundHandler/ChannelInboundHandlerAdapter 简化

整个聊天系统大部分的指令判断逻辑是重复的，下面介绍如何通过 SImpleChannelInboundHandler/ChannelInboundHandlerAdapter 简化指令的处理逻辑。

> [`ChannelInboundHandlerAdapter`](https://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html "class in io.netty.channel") which allows to explicit only handle a specific type of messages. For example here is an implementation which only handle `String` messages.

**ChannelInboundHandlerAdapter**  允许明确地只处理特定类型的消息。而`SimpleChannelInboundHandler`提供了一个模板，作用是把处理逻辑不变的内容写好在 `channelRead(ctx,msg)` 中，并且在里面调用` channelRead0` ，这样处理之后就可以通过抽象方法实现传递到子类中去了。

## 区别

`SimpleChannelInboundHandler`和`ChannelInboundHandlerAdapter`这两个类使用上不太好区分，下面再补充介绍一下如何正确对待使用两者。

`ChannelInboundHandlerAdapter` 需要覆盖的方法是**channelRead**，特点是**不会自动释放消息**，需要调用**ctx.fireChannelRead(msg)** 向后续链条处理器传递消息，也就是需要手动通过责任链的方式传递给下位处理器。
​
`SimpleChannelInboundHandler` 是 `ChannelInboundHandlerAdapter` 的子类，**做了额外的处理，会自动释放消息**，如果还需要继续传递消息，需调用一次 **ReferenceCountUtil.retain(msg)**。需注意`SimpleChannelInboundHandler`也需要调用`ctx.fireChannelRead(msg)`来触发链条中下一处理器处理。

`ChannelInboundHandlerAdapter`通常用于处于链条中间的某些环节处理，对数据进行某些处理，如数据验证，需要将消息继续传递。
​
`SimpleChannelInboundHandler`则比较适合链条最后一个环节，该环节处理完后，后续不再需要该消息，因此可以自动释放。

## 应用

在聊天系统中统一处理的Handler继承了SimpleChannelInboundHandler，重写`channelRead0`方法，主要对于解码之后的操作指令和通用Map进行匹配，如果匹配则分发到具体的逻辑处理器。

```java
@ChannelHandler.Sharable  
public class IMHandler extends SimpleChannelInboundHandler<Packet> {  
    public static final IMHandler INSTANCE = new IMHandler();  
  
    private Map<Byte, SimpleChannelInboundHandler<? extends Packet>> handlerMap;  
  
    private IMHandler() {  
        handlerMap = new HashMap<>(7);  
  
        handlerMap.put(MESSAGE_REQUEST, MessageRequestHandler.INSTANCE);  
        handlerMap.put(CREATE_GROUP_REQUEST, CreateGroupRequestHandler.INSTANCE);  
        handlerMap.put(JOIN_GROUP_REQUEST, JoinGroupRequestHandler.INSTANCE);  
        handlerMap.put(QUIT_GROUP_REQUEST, QuitGroupRequestHandler.INSTANCE);  
        handlerMap.put(LIST_GROUP_MEMBERS_REQUEST, ListGroupMembersRequestHandler.INSTANCE);  
        handlerMap.put(GROUP_MESSAGE_REQUEST, GroupMessageRequestHandler.INSTANCE);  
        handlerMap.put(LOGOUT_REQUEST, LogoutRequestHandler.INSTANCE);  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, Packet packet) throws Exception {  
        handlerMap.get(packet.getCommand()).channelRead(ctx, packet);  
    }  
}
```

# 客户端和服务端单聊

## 目标

- 输入用户名，服务端随机分配ID，这里省去通过账号和密码注册过程 。
- 多个客户端登录，用 userId 空格 消息的方式单聊。

## 实现过程

1. 使用工具类把UserId和Channel绑定为Session。

    -  Session的信息包含用户ID以及名称 ，后续可以扩展更多的字段。
    
2. 使用`SessionUtil`工具类操作Session，通过Session贮存当前会话信息。
   
    - 这里用的**ConcurrentHashMap**实现并发安全
    - ConcurrentHashMap为**userId -> Channel**的映射Map。
    - 用户登录的时候，需要把Session塞入Map。
    - 当用户断开`Channel`连接退出的时候，需要移除Session信息  
    
3. 服务端接受消息并且转发（这里Netty类似转发手机信号的基站）
   
    - 获取会话信息。
    - 构造发给客户端的对象`MessageResponse `。
    - 消息接收方标识获取对应`Channel`。
    - 如果目标用户登录则发送消息，如果对方不在线，则控制台打印警告信息。

具体的代码在前面的收发消息中有提到过，这里重复展示一遍。

**MessageResponseHandler**

```java
public class MessageResponseHandler extends SimpleChannelInboundHandler<MessageResponsePacket> {  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, MessageResponsePacket messageResponsePacket) {  
        String fromUserId = messageResponsePacket.getFromUserId();  
        String fromUserName = messageResponsePacket.getFromUserName();  
        System.out.println(fromUserId + ":" + fromUserName + " -> " + messageResponsePacket  
                .getMessage());  
    }  
}
```


**MessageRequestHandler**

```java
@ChannelHandler.Sharable  
public class MessageRequestHandler extends SimpleChannelInboundHandler<MessageRequestPacket> {  
    public static final MessageRequestHandler INSTANCE = new MessageRequestHandler();  
  
    private MessageRequestHandler() {  
  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, MessageRequestPacket messageRequestPacket) {  
        long begin = System.currentTimeMillis();  
  
  
        // 1.拿到消息发送方的会话信息  
        Session session = SessionUtil.getSession(ctx.channel());  
  
        // 2.通过消息发送方的会话信息构造要发送的消息  
        MessageResponsePacket messageResponsePacket = new MessageResponsePacket();  
        messageResponsePacket.setFromUserId(session.getUserId());  
        messageResponsePacket.setFromUserName(session.getUserName());  
        messageResponsePacket.setMessage(messageRequestPacket.getMessage());  
  
        // 3.拿到消息接收方的 channel        Channel toUserChannel = SessionUtil.getChannel(messageRequestPacket.getToUserId());  
  
        // 4.将消息发送给消息接收方  
        if (toUserChannel != null && SessionUtil.hasLogin(toUserChannel)) {  
            toUserChannel.writeAndFlush(messageResponsePacket).addListener(future -> {  
                if (future.isDone()) {  
  
                }  
  
            });  
        } else {  
            System.err.println("[" + session.getUserId() + "] 不在线，发送失败!");  
  
        }  
    }  
}
```

# 群聊发起和通知

下面两个小节围绕群聊实现介绍。整个群聊和单聊实现类似，都是通过标识获取Channel，为了方面多个成员管理，设计 `ChannelGroup` 完成`Channel `的批量操作。

## 预期效果

1. 三位用户依次登录。
2. 控制台输入 createGroup 指令，提示创建群聊需要 userId 列表，之后以英文逗号分隔userId。
3. 群聊创建成功之后，所有群聊成员收到加入成功消息。

## 创建群聊实现

主要逻辑如下：
1. 创建一个 channel 分组。
2. 筛选出待加入群聊的用户的 channel 和 userName。
3.  创建群聊创建结果的响应。
4. 给每个客户端发送拉群通知
5. 保存群组相关的信息。

其中存储群的相关信息利用了`ConcurrentHashMap`实现，和Session的会话信息存储方式类似，**ChannelGroup**对象负责封装多个Channel的信息，模拟群聊中的“群”。

```java
  
@ChannelHandler.Sharable  
public class CreateGroupRequestHandler extends SimpleChannelInboundHandler<CreateGroupRequestPacket> {  
    public static final CreateGroupRequestHandler INSTANCE = new CreateGroupRequestHandler();  
  
    private CreateGroupRequestHandler() {  
  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, CreateGroupRequestPacket createGroupRequestPacket) {  
        List<String> userIdList = createGroupRequestPacket.getUserIdList();  
  
        List<String> userNameList = new ArrayList<>();  
        // 1. 创建一个 channel 分组  
        ChannelGroup channelGroup = new DefaultChannelGroup(ctx.executor());  
  
        // 2. 筛选出待加入群聊的用户的 channel 和 userName        for (String userId : userIdList) {  
            Channel channel = SessionUtil.getChannel(userId);  
            if (channel != null) {  
                channelGroup.add(channel);  
                userNameList.add(SessionUtil.getSession(channel).getUserName());  
            }  
        }  
  
        // 3. 创建群聊创建结果的响应  
        String groupId = IDUtil.randomId();  
        CreateGroupResponsePacket createGroupResponsePacket = new CreateGroupResponsePacket();  
        createGroupResponsePacket.setSuccess(true);  
        createGroupResponsePacket.setGroupId(groupId);  
        createGroupResponsePacket.setUserNameList(userNameList);  
  
        // 4. 给每个客户端发送拉群通知  
        channelGroup.writeAndFlush(createGroupResponsePacket);  
  
        System.out.print("群创建成功，id 为 " + createGroupResponsePacket.getGroupId() + ", ");  
        System.out.println("群里面有：" + createGroupResponsePacket.getUserNameList());  
  
        // 5. 保存群组相关的信息  
        SessionUtil.bindChannelGroup(groupId, channelGroup);  
    }  
}
```

客户端处理部分则是简单的打印创建群聊成功的信息，实现比较简单这里不再贴出相关代码。

# 群聊成员管理实现

## 设计流程和实现思路

### 设计流程

1. 加入群聊，控制台输出创建成功消息。
2. 控制台输入joinGroup 之后输入群ID，加入群聊，控制台显示加入群成功。
3. 控制台输入 listGroupMembers 然后输入群ID，展示群成员。
4. quitGroup 输入群ID，进行退群
5. 控制台输入joinGroup 之后输入群ID显示对应成员不在，则退群成功。

### 实现思路

1. 在控制台中加入群加入的命令处理器。
2. 服务端处理群聊请求。
3. 客户端处理加群响应.
4. 群聊退出实现。

## 在控制台中加入群加入的命令处理器

**JoinGroupConsoleCommand**

```java
public class JoinGroupConsoleCommand implements ConsoleCommand {  
    @Override  
    public void exec(Scanner scanner, Channel channel) {  
        JoinGroupRequestPacket joinGroupRequestPacket = new JoinGroupRequestPacket();  
  
        System.out.print("输入 groupId，加入群聊：");  
        String groupId = scanner.next();  
  
        joinGroupRequestPacket.setGroupId(groupId);  
        channel.writeAndFlush(joinGroupRequestPacket);  
    }  
}
```


## 服务端处理群聊请求

服务端处理群聊请求：

1. 构建Channel分区，把处在同一个分组的Channel放到一个List当中存储  
2. 如果群聊构建成功，则构建创建成功响应结果 。

```java
@ChannelHandler.Sharable  
public class JoinGroupRequestHandler extends SimpleChannelInboundHandler<JoinGroupRequestPacket> {  
    public static final JoinGroupRequestHandler INSTANCE = new JoinGroupRequestHandler();  
  
    private JoinGroupRequestHandler() {  
  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, JoinGroupRequestPacket requestPacket) {  
        // 1. 获取群对应的 channelGroup，然后将当前用户的 channel 添加进去  
        String groupId = requestPacket.getGroupId();  
        ChannelGroup channelGroup = SessionUtil.getChannelGroup(groupId);  
        channelGroup.add(ctx.channel());  
  
        // 2. 构造加群响应发送给客户端  
        JoinGroupResponsePacket responsePacket = new JoinGroupResponsePacket();  
  
        responsePacket.setSuccess(true);  
        responsePacket.setGroupId(groupId);  
        ctx.writeAndFlush(responsePacket);  
    }  
}
```

## 客户端处理加群响应

简单打印加群的响应消息。

```java
public class JoinGroupResponseHandler extends SimpleChannelInboundHandler<JoinGroupResponsePacket> {  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, JoinGroupResponsePacket responsePacket) {  
        if (responsePacket.isSuccess()) {  
            System.out.println("加入群[" + responsePacket.getGroupId() + "]成功!");  
        } else {  
            System.err.println("加入群[" + responsePacket.getGroupId() + "]失败，原因为：" + responsePacket.getReason());  
        }  
    }  
}
```


## 群聊退出实现

群聊退出主要是获取群对应的 channelGroup，然后将当前用户的 channel 移除，之后构建退群的响应信息回传客户端即可。

**QuitGroupRequestHandler**

```java
@ChannelHandler.Sharable  
public class QuitGroupRequestHandler extends SimpleChannelInboundHandler<QuitGroupRequestPacket> {  
    public static final QuitGroupRequestHandler INSTANCE = new QuitGroupRequestHandler();  
  
    private QuitGroupRequestHandler() {  
  
    }  
  
    @Override  
    protected void channelRead0(ChannelHandlerContext ctx, QuitGroupRequestPacket requestPacket) {  
        // 1. 获取群对应的 channelGroup，然后将当前用户的 channel 移除  
        String groupId = requestPacket.getGroupId();  
        ChannelGroup channelGroup = SessionUtil.getChannelGroup(groupId);  
        channelGroup.remove(ctx.channel());  
  
        // 2. 构造退群响应发送给客户端  
        QuitGroupResponsePacket responsePacket = new QuitGroupResponsePacket();  
  
        responsePacket.setGroupId(requestPacket.getGroupId());  
        responsePacket.setSuccess(true);  
        ctx.writeAndFlush(responsePacket);  
  
    }  
}
```


# 心跳检测

## 网络问题

### 假死

TCP层面来看，服务端收到4次握手包或者RST包才算真正断开连接，如果中途应用程序并没有捕获到，此时是认为这条连接存在的。
        
### 假死引发问题  

- 客户端发送数据超时无响应，影响体验。
- 浪费CPU和内存资源，性能下滑。
       
### 假死原因  

- 公网丢包，网络抖动 。
- 应用程序阻塞无法读写 。
- 客户端或者服务端设别故障，网卡，机房故障。

为了解决上面的问题，通常会使用心跳检测机制定期检测每个`Channel`连接是否存活。

## 服务端心跳检测实现

1. 通过`IdleStateHandler`自带`Handler`实现
2. 继承类，然后开启定时任务
3. 触发假死该`Handler`回调`channelIdle` 方法

## 客户端预判和防御假死

1. 新建`Handler`。
2. 开启定时线程。
3. 组装心跳包。
4. 发送心跳。
5. 服务端简单开发接受和识别心跳包的Handler，之后回送收到心跳包消息即可。

## 注意事项

- 心跳检测**Handler**插入到整个**Pipeline**最前面，因为如果连接实际已经断开后续的所有处理均无意义。
- 假死不一定“死”，防止服务端误判，客户端也需要措施防止假死和预判假死，这就是客户端预判的含义。

## 思考

1. IdleHandler 可否单例？
2. 断开链接之后重新连接登录

### IdleHandler 可否单例？

答案是**不能**。因为它并不是无状态的，并且每个Channel都有各自的连接状态。

### 断开链接之后重新连接登录

通过额外的线程定时轮循所有的连接的活跃性，如果发现其中有死连接，则执行重连。

# 写在最后

熟悉聊天系统对于后续的源码分析十分有意义，项目的整体构建比较简单，个人在笔记中将重点部分做了一个梳理。

# 文章参考

[https://juejin.cn/book/m/6844733738119593991/section/6844733738291576840?suid=2040300414187416](https://juejin.cn/book/m/6844733738119593991/section/6844733738291576840?suid=2040300414187416)