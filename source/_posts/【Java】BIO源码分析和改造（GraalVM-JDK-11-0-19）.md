---
title: 【Java】BIO源码分析和改造（GraalVM JDK 11.0.19）
subtitle: Java 的传统BIO Socket编程源码分析
url_suffix: socketjavabio
author: lazytime
tags:
  - Java
categories:
  - Java
keywords: ['socket','accept']
description: 网络IO编程的入门部分Java 的传统BIO Socket编程源码分析
copyright: true
date: 2023-07-16 14:28:56
---

# 引言

本文介绍网络IO编程的入门部分，Java 的传统BIO Socket编程源码分析，之后了解如何将BIO阻塞`accept()` 和 `read()` 改造为非阻塞行为，最后将会介绍Linux文档，其中描述了如何处理`Socket`的`accept`，对比Java的Socket实现代码，基本可以发现和Linux行为基本一致。

废话不多说，我们直接开始。

# 什么是Socket？

Socket起源于Unix的一种通信机制，中文通常叫他“套接字”，通常代表了网络IP和端口，可以看作是通信过程的一个“句柄”。

Socket 也可以理解为网络编程当中的API，编程语言提供了对应的API实现方式，电脑上的网络应用程序也是通过“套接字”完成网络请求接受与应答。

总而言之：**Socket是应用层与TCP/IP协议族通信的中间软件抽象层**。

![Socket是应用层与TCP/IP协议族通信的中间软件抽象层](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230705110036.png)

> 图片来源：[socket图解 · Go语言中文文档 (topgoer.com)](https://www.topgoer.com/%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/socket%E7%BC%96%E7%A8%8B/socket%E5%9B%BE%E8%A7%A3.html)

<!-- more -->

# 阻塞式IO模型

在 **《UNIX Network Programming》** 一书当中，用UDP传输的案例模拟了阻塞式的IO模型，这个模型的概念和Java BIO的阻塞模型类似。

下面函数中应用进程在调用 `recvfrom` 之后就开始系统调用并且进行阻塞，等待内核把数据准备并且复制完成之后才得到结果，或者等待过程中发生错误返回。

![阻塞式IO模型](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230411212436.png)

从图片可以看到，在内核工作的整个过程中应用进程无法做其他任何操作。

# BIO 通信模型

我们把上面的阻塞IO模型转为IO通信模型，结果如下：

![BIO 通信模型](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230705103727.png)

BIO对于每一个客户端进行阻塞等待接收连接，同一个时间只能处理一个Socket请求，并且在构建完成之后通常会分配一个Thread线程为其进行服务。

# BIO 阻塞案例代码

## BioClientSocket

```java
/**  
 *  BioClientSocket 客户端 Socket实现  
 * @author Xander  
 * @version v1.0.0  
 * @Package : com.zxd.interview.niosource.bio  
 * @Description : BioClientSocket 客户端 Socket实现  
 * @Create on : 2023/7/5 09:52  
 **/@Slf4j  
public class BioClientSocket {  
  
  
    public void initBIOClient(String host, int port) {  
        BufferedReader reader = null;  
        BufferedWriter writer = null;  
        Socket socket = null;  
        String inputContent;  
        int count = 0;  
        try {  
            reader = new BufferedReader(new InputStreamReader(System.in));  
            socket = new Socket(host, port);  
            writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));  
            log.info("clientSocket started: " + stringNowTime());  
            while (((inputContent = reader.readLine()) != null) && count < 2) {  
                inputContent = stringNowTime() + ": 第" + count + "条消息: " + inputContent + "\n";  
                writer.write(inputContent);//将消息发送给服务端  
                writer.flush();  
                count++;  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally {  
            try {  
                socket.close();  
                reader.close();  
                writer.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
  
    }  
  
    public String stringNowTime() {  
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        return format.format(new Date());  
    }  
  
    public static void main(String[] args) {  
        BioClientSocket client = new BioClientSocket();  
        client.initBIOClient("127.0.0.1", 8888);  
    }/**  
     运行结果：  
     clientSocket started: 2023-07-05 10:26:05  
     7978987     797898     tyuytu     */}
```

## BioServerSocket

```java
  
/**  
 * ServerSocket 实现  
 * @author Xander  
 * @version v1.0.0  
 * @Package : com.zxd.interview.niosource.bio  
 * @Description : ServerSocket 实现  
 * @Create on : 2023/7/5 09:48  
 **/@Slf4j  
public class BioServerSocket {  
  
    public void initBIOServer(int port) {  
        ServerSocket serverSocket = null;//服务端Socket  
        Socket socket = null;//客户端socket  
        BufferedReader reader = null;  
        String inputContent;  
        int count = 0;  
        try {  
            serverSocket = new ServerSocket(port);  
           log.info(stringNowTime() + ": serverSocket started");  
            while (true) {  
                socket = serverSocket.accept();  
               log.info(stringNowTime() + ": id为" + socket.hashCode() + "的Clientsocket connected");  
                reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));  
                while ((inputContent = reader.readLine()) != null) {  
                   log.info("收到id为" + socket.hashCode() + "  " + inputContent);  
                    count++;  
                }  
               log.info("id为" + socket.hashCode() + "的Clientsocket " + stringNowTime() + "读取结束");  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        } finally {  
            try {  
                if(Objects.nonNull(reader)){  
  
                    reader.close();  
                }  
                if(Objects.nonNull(socket)){  
  
                    socket.close();  
                }  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
  
    }/**  
     运行结果：  
     10:25:57.731 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - 2023-07-05 10:25:57: serverSocket started  
     10:26:08.442 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - 2023-07-05 10:26:08: id为161960012的Clientsocket connected  
     10:26:29.356 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - 收到id为161960012  2023-07-05 10:26:26: 第0条消息: 7978987  
     10:26:34.409 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - 收到id为161960012  2023-07-05 10:26:34: 第1条消息: 797898  
     10:26:38.298 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - id为161960012的Clientsocket 2023-07-05 10:26:38读取结束  
     */  
  
    public String stringNowTime() {  
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        return format.format(new Date());  
    }  
  
    public static void main(String[] args) {  
        BioServerSocket server = new BioServerSocket();  
        server.initBIOServer(8888);  
  
    }  
}
```

BIO 阻塞模型中，需要关注的代码主要是这几个：

- `serverSocket = new ServerSocket(port);`
- `socket = serverSocket.accept();`
- `socket = new Socket(host, port);`

从代码中可以看出，客户端在获取Socket建立连接后，通过系统输入输出流完成读写IO操作，服务端则通过系统缓冲流Buffer来提高读写效率。 

# ServerSocket 中 bind 解读

在具体的解读之前，先看下整个调用的大致流程图。

![ServerSocket 中 bind 解读](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230706174016.png)

由于是`ServerSocket`服务端先启动，这里先对`bind`操作进行解读，`bind`操作是在本机的某个端口和IP地址上进行listen监听，在bind成功之后，服务端通常进入`accept`阻塞等待，此时客户端Socket请求此地址将会进行Socket连接绑定。

我们从`ServerSocket`的初始化代码作为入口进行介绍。

```java
//java.net.ServerSocket#ServerSocket(int)
public ServerSocket(int port) throws IOException {
    this(port, 50, null);
}


public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException {
    setImpl();
    // 检查端口是否越界
    // 0xFFFF = 15 * 16^3 + 15 * 16^2 + 15 * 16^1 + 15 * 16^0 = **65535**
    if (port < 0 || port > 0xFFFF)
        throw new IllegalArgumentException(
                    "Port value out of range: " + port);
    if (backlog < 1)
        backlog = 50;
    try {
	    // 核心部分
        bind(new InetSocketAddress(bindAddr, port), backlog);
    } catch(SecurityException e) {
        close();
        throw e;
    } catch(IOException e) {
        close();
        throw e;
    }
}

```

`setImpl();`这个方法我们先暂时放到一边，我们简单扫一下其他代码。

在上面的模型案例代码当中，我们传入的`ip`和`port`都处在合法的范围内，Socket规定的端口范围是**0 - 65525**，超过这个范围不允许进行`bind`。

上面代码的核心逻辑是`bind(xxx)`这一段操作。

```java
bind(new InetSocketAddress(bindAddr, port), backlog);
```

## new InetSocketAddress(bindAddr, port)

在`bind`方法调用之前，`ServerSocket`会先构建 **InetSocketAddress** 对象。**InetSocketAddress**对象构建实际为**InetSocketAddressHolder**包装类。包装类的作用是可以保护`IP`和`Port`等敏感字段的外部篡改。

此外凑够代码可以看到，构建对象会对于`IP`和`Port`进行二次检查，如果IP地址不存在，会给一个默认值（通常是` 0.0.0.0` ）。

> Creates a socket address from an IP address and a port number.
> A valid port value is between 0 and 65535. A port number of zero will let the system pick up an ephemeral port in a bind operation.|
> 根据IP地址和端口号创建Socket地址。有效的端口值介于0和65535之间。端口号为0时，系统将在绑定操作中使用短暂端口。

**InetSocketAddressHolder** 对象构建完成之后，接着就进入到核心的`bind(SocketAddress endpoint, int backlog)`内部代码。

## bind(SocketAddress endpoint, int backlog)

**java.net.ServerSocket#bind(java.net.SocketAddress, int)**

```java
/**
Binds the ServerSocket to a specific address (IP address and port number).
将ServerSocket绑定到一个特定的地址（IP地址和端口号）。

If the address is null, then the system will pick up an ephemeral port and a valid local address to bind the socket.
如果地址为空，那么系统会选取一个短暂的端口和一个有效的本地地址来绑定套接字。
*/
public void bind(SocketAddress endpoint, int backlog) throws IOException {
		// Socket是否已经被关闭
        if (isClosed())
            throw new SocketException("Socket is closed");
        // 判断是否已经绑定
        if (!oldImpl && isBound())
            throw new SocketException("Already bound");
        if (endpoint == null)
	        // 如果地址为空，给一个默认地址
            endpoint = new InetSocketAddress(0);
        if (!(endpoint instanceof InetSocketAddress))
            throw new IllegalArgumentException("Unsupported address type");
        // 类型强转为 InetSocketAddress
        InetSocketAddress epoint = (InetSocketAddress) endpoint;
        // 如果地址已经被占用了
        if (epoint.isUnresolved())
            throw new SocketException("Unresolved address");
        if (backlog < 1)
          backlog = 50;
        try {
            SecurityManager security = System.getSecurityManager();
            if (security != null)
	            // 端口进行安全检查
                security.checkListen(epoint.getPort());
            getImpl().bind(epoint.getAddress(), epoint.getPort());
            getImpl().listen(backlog);
            bound = true;
        } catch(SecurityException e) {
            bound = false;
            throw e;
        } catch(IOException e) {
            bound = false;
            throw e;
        }
    }
```

bind 方法是将 **ServerSocket** 绑定到一个特定的地址（IP地址和端口号）。如果地址为空，那么系统会选取一个临时端口和有效的本地地址来绑定 ServerSocket。

跳过不需要关注的校验代码，在·`try` 中最后有三行比较重要的代码。

```java
getImpl().bind(epoint.getAddress(), epoint.getPort());  
getImpl().listen(backlog);  
bound = true;
```

这里的代码按照初步理解是获取一个impl对象，然后绑定地址和端口，调用listen方法会传递backlog。

backlog这个值的作用可以看下面的地址，这里整理文章内容大致理解：

- [Linux Network Programming, Part 1 (linuxjournal.com)](https://www.linuxjournal.com/files/linuxjournal.com/linuxjournal/articles/023/2333/2333s2.html)

- [详解socket中的backlog 参数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/104874605)

`backlog`主要是和`Socket`有关。在Socket编程中**listen函数的第二个参数为backlog**，用于服务器编程。

```c
listen(sock, backlog);
```

![TCP 握手](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230706114407.png)

在TCP 三次握手当中，LISTEN 状态的服务端 Socket 收到 SYN，会建立一个 **SYN_REVD** 的连接，**SYN_REVD** 是一个半连接状态，只有在收到客户端的ACK之后才会进入**ESTABLISHED**，也就是说三次握手的过程必然会经历**SYN_REVD**和**ESTABLISHED**两个状态。

针对这两个状态，不同的操作系统有不同实现，**在 FressBSD 中 backlog 就是描述状态为 SYN_REVD 和 ESTABLISHED 的所有连接最大数量**。

在 Linux 系统当中，使用两个队列 **syn queue**和 **accept queue**，这两个队列分别存储状态为SYN_REVD和状态为ESTABLISHED的连接，**Llinux2.2及以后，backlog表示accept queue的大小**，而syn queue大小由 `/proc/sys/net/ipv4/tcp_max_syn_backlog`配置。

所以可以看到backlog的值直接影响了建立连接的效率。上面代码中`backlog=50`，意味着 accept queue 的容量为50。

`listen`方法执行完成之后，此时会设置`bound = true`，代码执行到此处说明`Socket`绑定成功了。

现在我们回过头看`getImpl().bind(epoint.getAddress(), epoint.getPort()); `这块代码工作。

## setImpl()

介绍`getImpl()`的前提是我们要知道如何`set`的，具体代码位于构造方法中一行不起眼的`setImpl()`操作。

**java.net.ServerSocket#setImpl**。

```java
private void setImpl() {
        if (factory != null) {
            impl = factory.createSocketImpl();
            checkOldImpl();
        } else {
            // No need to do a checkOldImpl() here, we know it's an up to date
            // SocketImpl!
            impl = new SocksSocketImpl();
        }
        if (impl != null)
            impl.setServerSocket(this);
    }
```

注意，在第一次初始化的时候，`SocketImplFactory`是没有被初始化过的，所以走的是else分支，为内部的成员变量 `SocketImpl`进行初始化。

```java
  
/**  
 * The implementation of this Socket. */
private SocketImpl impl;
```

`SocksSocketImpl` 被初始化，会将 `SocketImpl` 的 `ServerSocket` 设置为当前正在实例化的 ServerSocket（也就是this 引用）。

这里的处理工作很简单，分别是初始化 **SocksSocketImpl** 和 把当前实例引用设置给这个初始化的 **SocksSocketImpl** 的成员变量，这时候自身的引用逸出了。

下面这里我们再看看 `getImpl()` 干了啥。

## getImpl()

**java.net.ServerSocket#getImpl**

代码内容也比较简单，首先检查`SocketImpl`是否创建，第一次连接这里为false，会进入`createImpl()`方法。

```java
/**
Get the SocketImpl attached to this socket, creating it if necessary.
获取连接到此套接字的SocketImpl，如果有必要，可以创建它。
*/
SocketImpl getImpl() throws SocketException {  
    if (!created)  
        createImpl();  
    return impl;  
}
```

在`createImpl()`当中，通常**SocketImpl**已经在构造器初始化完成，这里直接更新`created`状态即可。

```java
void createImpl() throws SocketException {  
    if (impl == null)  
        setImpl();  
    try {  
        impl.create(true);  
        created = true;  
    } catch (IOException e) {  
        throw new SocketException(e.getMessage());  
    }  
}
```

`setImpl()`和`getImpl()`方法配合，可以确定 **SocketImpl** 在使用的时候一定是被初始化完成的。

下面我们再来看看他是如何进行下面两项关键操作的：

```java
getImpl().bind(epoint.getAddress(), epoint.getPort());  
getImpl().listen(backlog);
```

### SocketImpl.bind(epoint.getAddress(), epoint.getPort())

在之前的初始化代码中给InetAddress对象初始化设置了IP和端口等参数，现在委托 **SocketImpl**

**java.net.AbstractPlainSocketImpl#bind**

`bind`方法是同步的，一开始需要先获取到`fdLock`锁，然后判断是否满足Socket绑定条件，如果满足则利用钩子(NetHooks) 对象进行前置TCP绑定。

注意，此处个人进入代码之后，发现此方法发现在**JDK11**之中是一个**空方法**，而**JDK8**当中会有一段`provider.implBeforeTcpBind(fdObj, address, port);`的调用。

```java
protected synchronized void bind(InetAddress address, int lport)  
    throws IOException  
{  
	// 获取 fdLock 锁
   synchronized (fdLock) {  
        if (!closePending && (socket == null || !socket.isBound())) {  
            NetHooks.beforeTcpBind(fd, address, lport);  
        }  
    }  
    // 是否链接本地地址
    if (address.isLinkLocalAddress()) {  
        address = IPAddressUtil.toScopedAddress(address);  
    }  
    // 关键
    socketBind(address, lport);  
    // 服务端和客户端的Socket走不同的 if 判断
    if (socket != null)  
        socket.setBound();  
    if (serverSocket != null)  
        serverSocket.setBound();  
}
```

在JDK11版本中，**fdLock**加锁之后的一些操作实际上一个“无用”操作。

不过这些内容和核心逻辑无太多干系，我们跳过细枝末节，看`socketBind(address, lport);  `这部分代码。

> fdLock 锁作用介绍，注释说明它用于在增加/减少**fdUseCount**时锁定。

```java
/* lock when increment/decrementing fdUseCount */  
// 在增加/减少fdUseCount时锁定
protected final Object fdLock = new Object();
```

### PlainSocketImpl#socketBind(InetAddress address, int port)

```java
@Override  
void socketBind(InetAddress address, int port) throws IOException {  
    int nativefd = checkAndReturnNativeFD();  
  
    if (address == null)  
        throw new NullPointerException("inet address argument is null.");  

	// 目前IPv4地址已经分配完毕，所以优先用 IPV6的，并且不支持 IPV4 
    if (preferIPv4Stack && !(address instanceof Inet4Address))  
        throw new SocketException("Protocol family not supported");  

	// 核心操作
    bind0(nativefd, address, port, useExclusiveBind);  
    // 如果是之前 InetAddress 为空默认初始化的端口为0，则重新随机分配一个端口
    if (port == 0) {  
        localport = localPort0(nativefd);  
    } else {  
        localport = port;  
    }  
  
    this.address = address;  
}
```

`socketBind(address, lport); `方法调用，最后绑定操作为JVM的底层C++操作`bind0`。

`bind0`属于比较底层的代码，这里我们就不继续考究了，如果读者好奇可以阅读 HotSpot 的开源实现代码。

```java
static native void bind0(int fd, InetAddress localAddress, int localport,  
                         boolean exclBind)  
    throws IOException;
```

从整体上看，上面这一整个`bind`操作都是同步完成的，其中主要是做一系列检查和异常抛出等操作，最后调用底层的JVM方法完成Socket绑定。

## 画图小结

最后笔者通过个人理解画了一幅图，主要描述了 `bind` 操作大致的逻辑，可以看到很多地方都和JVM的底层C++代码打交道，调试起来也非常麻烦。

> <s>毕竟是 Java1.0 出来的玩意，</s>看源码我们要抓大放小，不过后续的JDK提案中，有人提出要收拾这个老古董？

![ServerSocket的bind](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707151128.png)

> 根据上面画图图可以看到，整个IO文件描述符也是要走native方法从操作系统底层才可以拿到。

# ServerSocket中accept解读

`ServerSocket`的`accpet`是如何阻塞获取连接的？`accept`方法的作用是询问操作系统是否有收到新的`Socket`套接字信息，注意这个操作过程在操作系统底层调用实现上都是 **同步**的。

那么如果操作系统从Socket中对应的绑定端口没有连接进来怎么办？根据Linux的`accept`文档描述以及Java注释都有介绍他会在底层操作系统**阻塞** 。

我们解答为什么 `accept` 方法会阻塞这个问题很简单，因为底层操作系统使用的是**同步IO**，Linux底层就是这么干的，Java Doc上的注释也是这样明明白白写的。

## java.net.ServerSocket#accept

我们从代码层面看看 `accept` 方法干了啥。

```java
/**
Listens for a connection to be made to this socket and accepts it. The method blocks until a connection is made.
监听并接受与此套接字的连接。该方法会阻塞，直到有一个连接被建立。

A new Socket s is created and, if there is a security manager, the security manager's checkAccept method is called with s.getInetAddress().getHostAddress() and s.getPort() as its arguments to ensure the operation is allowed. This could result in a SecurityException.
一个新的Socket s被创建，如果有一个安全管理器，安全管理器的checkAccept方法被调用，参数是s.getInetAddress().getHostAddress()和s.getPort()，以确保该操作被允许。这可能会导致一个SecurityException。
*/
public Socket accept() throws IOException {  
    if (isClosed())  
        throw new SocketException("Socket is closed");  
    if (!isBound())  
        throw new SocketException("Socket is not bound yet");  
    Socket s = new Socket((SocketImpl) null);  
    implAccept(s);  
    return s;  
}
```

Java Doc 说明了`accept()`会进行阻塞，这里疑问比较大的点可能是`Socket s = new Socket((SocketImpl) null); `，这行代码为什么又要新建一个Socket？**带着疑问**我们继续看`implAccept(s);  `方法。

## java.net.ServerSocket#implAccept

```java
/**
Subclasses of ServerSocket use this method to override accept() to return their own subclass of socket. So a FooServerSocket will typically hand this method an empty FooSocket. On return from implAccept the FooSocket will be connected to a client.

ServerSocket的子类使用这个方法来覆盖accept()（的行为），以返回他们自己的socket子类。比如一个FooServerSocket通常会将一个空的FooSocket交给这个方法。从 implAccept 返回时，FooSocket 将被连接到一个客户端。
*/
protected final void implAccept(Socket s) throws IOException {  
    SocketImpl si = null;  
    try {  
	    // 判断新对象 Socketimpl 是否设置
        if (s.impl == null)  
          s.setImpl();  
        else {  
            s.impl.reset();  
        }  

		// si 指向 Socket 对象的 impl 
        si = s.impl; 
        // Socket 对象的 impl 引用 暂时置空
        s.impl = null;  
        // impl 地址和文件描述符初始化
        si.address = new InetAddress();  
        si.fd = new FileDescriptor();  
        // getImpl() 获取的是 ServerSocket 的 impl，注意不是 Socket的
// 4. 调用 ServerSocket 持有的 SocksSocketImpl 对象完成底层操作系统的 accept 操作
        getImpl().accept(si);  

		// raw fd has been set 
		// 原始fd已被设置  
        SocketCleanable.register(si.fd);    
		// 安全检查
        SecurityManager security = System.getSecurityManager();  
        if (security != null) {  
            security.checkAccept(si.getInetAddress().getHostAddress(),  
                                 si.getPort());  
        }  
    } catch (IOException e) {  
	    // 如果出现底层IO异常，则s.impl = si;把之前临时置空的引用给重置回来
        if (si != null)  
            si.reset();  
        s.impl = si;  
        throw e;  
    } catch (SecurityException e) {  
        if (si != null)  
            si.reset();  
        s.impl = si;  
        throw e;  
    }  
    // 把之前临时置空的引用给重置回来
    s.impl = si;  
    s.postAccept();  
}
```

首先代码进入一个`if`判断 `new Socket` 新对象的**Socketimpl**是否设置，如果为空则就设置，如果不为空则`reset()` 重置。毫无疑问，这里是刚刚初始化的Socket，此时**Socketimpl** 肯定是没有设置的。

```java
if (s.impl == null)  
  s.setImpl();  
else {  
	s.impl.reset();  
}  
```

显然首次进入代码通常就是走`if`分支。`setImpl` 这个方法和ServerSocket的`setImpl`非常像，`new Socket` 新对象会为自己的 **SocketImpl** 成员对象进行初始化。

![SocketImpl](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707160516.png)

至此，我们简单画图理解上面的操作干了啥：

![accept 操作分析](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707161437.png)

接下来是一些有点”绕“的操作，建议读者边调试边跟着画图理解：

```java
// 1. si 指向 Socket 对象的 impl 
si = s.impl; 
// 2. Socket 对象的 impl引用 暂时置空
s.impl = null;  
// 3. impl 地址和文件描述符初始化
si.address = new InetAddress();  
si.fd = new FileDescriptor();
// getImpl() 获取的是 ServerSocket 的 impl，注意不是 Socket的
// 4. 调用 ServerSocket 持有的 SocksSocketImpl 对象完成底层操作系统的 accept 操作
getImpl().accept(si);  


// ....
// 假设此时 accept 获取到连接
s.impl = si; 

```

> 这里吐槽下老外这种变量命名给规则，啥`si`呀`s`，a，b，c，d 的，不画图很容易绕进去。

![ServerSocket 中 accepet 解读](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707163648.png)

格外强调下， `getImpl()` 的 **impl对象**和 `si.impl` 对象并不是同一个，这些代码内容非常像但是属于不同的类，切记不要混淆。

![实例对象对比](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707163430.png)

![实例对象对比](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707163500.png)

代码最后有一个必定会执行的 `s.impl = si; `操作（因为之前暂时把引用“脱钩”了），如果是异常的`si`还会进行额外的`reset`重置操作。

```java
s.impl = si; 
```

这里回答一下之前遗留的问题，**`Socket s = new Socket((SocketImpl) null); `这行代码为什么又要新建一个Socket？**

我们观察上面绘制的操作图，`s.impl = null; `的执行，此时Socket对象和这个`SocketImpl`暂时”失去关联“，这个时候确保哪怕`new Socket`对象绑定失败，此时对于`SocketImpl`来说根本是无感知的，如果失败了`Socket`会完全重置，好像什么都没有发送过，而过成功了，此时把引用“接回去”，必然得到的可用的Socket。

> 这里给一个不恰当的比喻，当年诸葛亮草船借箭，如果有碰到没有借箭的船，是不是就可以直接”烧了“不要了，而如果接到箭自然需要回港“卸货‘”，对于吴国来说，它们只看到“成功”借到箭的船只。

执行了`getImpl().accept(si);`方法之后，我们在**AbstractPlainSocketImpl**找到**accept**方法。我们进入方法继续下探。

### java.net.AbstractPlainSocketImpl#accept

```java
/**
Accepts connections.
接受连接
*/
protected void accept(SocketImpl s) throws IOException {  
    acquireFD();  
    try {  
        socketAccept(s);  
    } finally {  
        releaseFD();  
    }  
}
```

首先会获取并且植入文件描述符号，加锁获取之后会把**fdUseCount** 的计数器值+1，表示有一个新增的Socket连接。

> 加锁保证 fdUseCount  计数是线程安全的

```java
// "Acquires" and returns the FileDescriptor for this impl
// - "获取 "并返回该植入物的文件描述符。
FileDescriptor acquireFD() {  
    synchronized (fdLock) {  
        fdUseCount++;  
        return fd;  
    }  
}
```

### java.net.PlainSocketImpl#socketAccept

因为不同的操作系统实现不同，这里仅以个人看到的**JDK11**版本源码为例。

```java
@Override
void socketAccept(SocketImpl s) throws IOException {
	int nativefd = checkAndReturnNativeFD();

	if (s == null)
		throw new NullPointerException("socket is null");

	int newfd = -1;
	InetSocketAddress[] isaa = new InetSocketAddress[1];
	if (timeout <= 0) {
		newfd = accept0(nativefd, isaa);
	} else {
		configureBlocking(nativefd, false);
		try {
			waitForNewConnection(nativefd, timeout);
			newfd = accept0(nativefd, isaa);
			if (newfd != -1) {
				configureBlocking(newfd, true);
			}
		} finally {
			configureBlocking(nativefd, true);
		}
	}
	/* Update (SocketImpl)s' fd */
	fdAccess.set(s.fd, newfd);
	/* Update socketImpls remote port, address and localport */
	InetSocketAddress isa = isaa[0];
	s.port = isa.getPort();
	s.address = isa.getAddress();
	s.localport = localport;
	if (preferIPv4Stack && !(s.address instanceof Inet4Address))
		throw new SocketException("Protocol family not supported");
}
```

这里我们只关心下面这部分代码，方法中首先判断 **timeout** 是否小于等于0（如果没有设置，那么默认就是 0），如果是则走`accept0(nativefd, isaa)`方法。

前面反复提到的`accept`操作关键实现这是个 native 方法，具体操作是**在操作系统层面检查`bind`的端口上是否有客户端数据接入，如果没有则一直阻塞等待**。

```java
if (timeout <= 0) {
	newfd = accept0(nativefd, isaa);
} else {
	configureBlocking(nativefd, false);
	try {
		waitForNewConnection(nativefd, timeout);
		newfd = accept0(nativefd, isaa);  
		if (newfd != -1) {
			configureBlocking(newfd, true);
		}
	} finally {
		configureBlocking(nativefd, true);
	}
} // <4>

```

```java
static native int accept0(int fd, InetSocketAddress[] isaa) throws IOException;
```

因为操作系统层面的阻塞需要影响到应用程序级别阻塞？、

显然`accept0(nativefd, isaa)`的操作系统层面阻塞这个是无法避免的，但是上面的代码提供了另外一种选择，那就是 **timeout** 的值设置大于0的值，此时**程序会在等到我们设置的时间后返回**，并且只会阻塞设置的这个时间量的值（单位毫秒）。

注意这里的 **newfd** 如果是 -1表示底层没有任何数据返回，在Linux的文档中同样有对应的介绍。

### java.net.ServerSocket#setSoTimeout

既然不阻塞的关键参数是**timeout** ， 接下来我们看下 timeout 值要如何设置，先来看下源码部分。

```java
/**
Enable/disable SO_TIMEOUT with the specified timeout, in milliseconds. With this option set to a non-zero timeout, a call to accept() for this ServerSocket will block for only this amount of time. If the timeout expires, a java.net.SocketTimeoutException is raised, though the ServerSocket is still valid. The option must be enabled prior to entering the blocking operation to have effect. The timeout must be > 0. A timeout of zero is interpreted as an infinite timeout.

启用/禁用SO_TIMEOUT，指定超时时间，单位为毫秒。在这个选项被设置为非零超时的情况下，对这个ServerSocket的accept()的调用将只阻塞这个时间量。如果超时过后，会引发java.net.SocketTimeoutException，尽管ServerSocket仍然有效。该选项必须在进入阻塞操作之前启用才能生效。超时必须大于0。超时为0会被解释为无限期超时。

*/
public synchronized void setSoTimeout(int timeout) throws SocketException {  
    if (isClosed())  
        throw new SocketException("Socket is closed");  
    getImpl().setOption(SocketOptions.SO_TIMEOUT, timeout);  
}
```

**java.net.SocketOptions#setOption** 方法最终调用的是`java.net.AbstractPlainSocketImpl#setOption`。

### java.net.AbstractPlainSocketImpl#setOption

```java
public void setOption(int opt, Object val) throws SocketException {  
    if (isClosedOrPending()) {  
        throw new SocketException("Socket Closed");  
    }  
    boolean on = true;  
    switch (opt) {  
	case SO_LINGER:  
        //..
    case SO_TIMEOUT:  
        if (val == null || (!(val instanceof Integer)))
                throw new SocketException("Bad parameter for SO_TIMEOUT");
            int tmp = ((Integer) val).intValue();
            if (tmp < 0)
                throw new IllegalArgumentException("timeout < 0");
            timeout = tmp;
            break;
    case TCP_NODELAY:  
        //....
    case SO_RCVBUF:  
        //....
    case SO_KEEPALIVE:  
        //....
    }  
    socketSetOption(opt, on, val);  
}
```

为了方便阅读，这里把其他的代码都删除了，只保留传参部分。

可以看到这里仅仅是将我们`setOption`里面传入的`timeout`值设置到了`AbstractPlainSocketImpl`的全局变量`timeout`里而已。


## 画图小结

个人认为整个`accept() `操作比较”恶心“（个人观点）的是几个引用的赋值变化上面，暂时”解绑“的目的是在进行底层Socket连接的时候如果出现异常也没有影响，此时Socket持有的引用也是null。

换句话说，**整个Socket要么对接成功，要么就是重置回没对接之前的状态可以进行下一次尝试，保证ServerSocket会收到一个没有任何异常的Socket连接**。

最后再看一眼图：

![accept 操作总结](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230707175534.png)


# 改造并实现accept的非阻塞实现

在进行案例程序的改造之前，必须要先理解**同步、异步、阻塞、非阻塞**这几个概念。

这个概念在之前的笔记中 [[《跟闪电侠学Netty》阅读笔记 - 开篇入门Netty]] 【洗衣机案例理解阻塞非阻塞，同步异步概念】这一部分提到过，[[【Java】《2小时搞定多线程》个人笔记]] 中又一次对于这几个概念做了个区分。

区分**同步**和**异步**的关键点是**被调用方的行为**，如果没有得到结果之前，服务端不返回任何结果，那么是同步的。如果没有得到结果之前，服务器可以返回结果，比如给一个**句柄**，通过这个句柄可以在未来某个时间点之后获得结果，这个句柄可以对应Java 并发编程的 **Future** 的概念。

再举个例子，比如说前面的`accept0`是**应用程序调用操作系统**，在Linux中就是访问系统内核，此时这一整块逻辑处理是选择”**永久等待一个客户端连接**“，符合 **如果没有得到结果之前，服务端不返回任何结果** 这种情况，所以它是**同步**的。

区分阻塞和非阻塞的关键点则是 **对于调用者而言的服务端状态***，如果我们站在线程状态的角度，阻塞对应 **Blocking**，非阻塞则通常对应**Running**（通常情况）。站在**线程发出请求**之后请求方的角度，区别则是**waiting**和**No waiting**。

理解同步异步阻塞和非阻塞之后，下面来尝试改造相关代码`accept`的阻塞问题，实现方式很简单，那就是设置 ** **timeout** ** ， 然后在异常处理上`continue`重试。

```java
/**  
 * accept 超时时间设置  
 */  
private static final int SO_TIMEOUT = 2000;

/***
 * NIO 改写
 * @description NIO 改写
 * @param port
 * @return void
 * @author xander
 * @date 2023/7/12 10:35
 */
public void initNioServer(int port) {
	ServerSocket serverSocket = null;//服务端Socket
	Socket socket = null;//客户端socket
	BufferedReader reader = null;
	String inputContent;
	int count = 0;
	try {
		serverSocket = new ServerSocket(port);
		// 1. 需要设置超时时间，会等待设置的时间之后再进行返回
		serverSocket.setSoTimeout(SO_TIMEOUT);
		log.info(stringNowTime() + ": serverSocket started");
		while (true) {
			// 2. 如果超时没有获取，这里会抛出异常，这里的处理策略是不处理异常
			try {
				socket = serverSocket.accept();
			} catch (SocketTimeoutException e) {
				//运行到这里表示本次accept是没有收到任何数据的，服务端的主线程在这里可以做一些其他事情
				log.info("now time is: " + stringNowTime());
				continue;
			}
			log.info(stringNowTime() + ": id为" + socket.hashCode() + "的Clientsocket connected");
			reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			while ((inputContent = reader.readLine()) != null) {
				log.info("收到id为" + socket.hashCode() + "  " + inputContent);
				count++;
			}
			log.info("id为" + socket.hashCode() + "的Clientsocket " + stringNowTime() + "读取结束");
		}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		try {
			if(Objects.nonNull(reader)){

				reader.close();
			}
			if(Objects.nonNull(socket)){

				socket.close();
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}/**运行结果：
     10:40:49.272 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - 2023-07-12 10:40:49: serverSocket started
     10:40:52.826 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:40:52
     10:40:54.830 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:40:54
     10:40:56.837 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:40:56
     10:40:58.840 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:40:58
     10:41:00.849 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:41:00
     10:41:02.852 [main] INFO com.zxd.interview.niosource.bio.BioServerSocket - now time is: 2023-07-12 10:41:02
     */
```

设置了**timeout**之后，`accept` 方法每次都会在间隔指定时间之后被唤醒一次，如果没有收到连接就会抛出异常，我们的处理方式是吞掉异常并且重新`accept`，这样就实现了类似非阻塞的效果。

## 小结

 Socket 当中 `getInputStream()` 的方法解析以及后续的`read`操作结构图如下。

![Socket.getInputStream()](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712151156.png)

# Socket 中的 getInputStream() 方法解析

我们实现了非阻塞的`accept`之后，再来看下另一个会产生阻塞的方法，那就是`Socket.getInputStream`，这个方法在Socket连接，服务端在read() 读取数据的时候会进行调用。

**java.net.Socket#getInputStream**

```java
/**
返回该socket的输入流。
如果该套接字有一个关联的通道，那么生成的输入流会将其所有操作委托给该通道。如果通道处于非阻塞模式，那么输入流的读操作将抛出java.nio.channel.IllegalBlockingModeException。
在异常情况下，底层连接可能会被远程主机或网络软件中断（例如在TCP连接中的连接重置）。当网络软件检测到连接断开时，返回的输入流会出现以下情况：

*/
  public InputStream getInputStream() throws IOException {
        if (isClosed())
            throw new SocketException("Socket is closed");
        if (!isConnected())
            throw new SocketException("Socket is not connected");
        if (isInputShutdown())
            throw new SocketException("Socket input is shutdown");
        InputStream is = null;
        try {
            is = AccessController.doPrivileged(
                new PrivilegedExceptionAction<>() {
                    public InputStream run() throws IOException {
                        return impl.getInputStream();
                    }
                });
        } catch (java.security.PrivilegedActionException e) {
            throw (IOException) e.getException();
        }
        return is;
    }
```

这里通过`AccessController`进行授权，`run`方法中调用**java.net.AbstractPlainSocketImpl#getInputStream**方法。

```java
protected synchronized InputStream getInputStream() throws IOException {  
    synchronized (fdLock) {  
        if (isClosedOrPending())  
            throw new IOException("Socket Closed");  
        if (shut_rd)  
            throw new IOException("Socket input is shutdown");  
        if (socketInputStream == null)  
            socketInputStream = new SocketInputStream(this);  
    }  
    return socketInputStream;  
}
```

可以看到，代码中创建了 **SocketInputStream** 对象，并且会将当前`AbstractPlainSocketImpl`对象传进去（这个对象实际就是 **SocksSocketImpl** ）。

在读数据的时候会调用如下方法：

```java
public int read(byte b[], int off, int length) throws IOException {  
    return read(b, off, length, impl.getTimeout());  
}
```

```java
int read(byte b[], int off, int length, int timeout) throws IOException {
	int n;

	// EOF already encountered
	if (eof) {
		return -1;
	}

	// connection reset
	if (impl.isConnectionReset()) {
		throw new SocketException("Connection reset");
	}

	// bounds check
	if (length <= 0 || off < 0 || length > b.length - off) {
		if (length == 0) {
			return 0;
		}
		throw new ArrayIndexOutOfBoundsException("length == " + length
				+ " off == " + off + " buffer length == " + b.length);
	}

	// acquire file descriptor and do the read
	// 获取文件描述符并进行读取
	FileDescriptor fd = impl.acquireFD();
	try {
		n = socketRead(fd, b, off, length, timeout);
		if (n > 0) {
			return n;
		}
	} catch (ConnectionResetException rstExc) {
		impl.setConnectionReset();
	} finally {
		impl.releaseFD();
	}

	/*
	 * If we get here we are at EOF, the socket has been closed,
	 * or the connection has been reset.
	 */
	if (impl.isClosedOrPending()) {
		throw new SocketException("Socket closed");
	}
	if (impl.isConnectionReset()) {
		throw new SocketException("Connection reset");
	}
	eof = true;
	return -1;
}
```

重点关注下面这一行代码，可以看到这里在读取的时候同样传递了 **timeout** 参数：

```java
n = socketRead(fd, b, off, length, timeout);
```

**socketRead** 方法会调用 **native** 的 `socketRead0` 方法。**timeout** 代表了读取的超时时间。

```java
private native int socketRead0(FileDescriptor fd,  
                               byte b[], int off, int len,  
                               int timeout)  
    throws IOException;
```

**timeout** 参数源于前面的`new SocketInputStream(this)`（也就是 **AbstractPlainSocketImpl** 对象）中的**this**引用`impl.getTimeout()`，这个参数的作用是指定`read`的超时时间，超时之后没有结果抛出异常。

```java
serverSocket.setSoTimeout(SO_TIMEOUT);
```

了解`read`方法中`timeout`的作用之后，我们便可以着手改造代码了，具体的改造部分个人放到后文单独的 `titile` 进行说明，方便后续回顾。

此外，这里经过仔细考虑，判断这部分代码读者很有可能会存在理解误区，误以为此处的 **AbstractPlainSocketImpl** 属于 **ServerSocket**，实际上它属于 **Socket**，也就是说我们设置的 `timeout` 是设置到 **Socket** 的 **AbstractPlainSocketImpl** 。

最为简单的证明方法是先在  **java.net.Socket#setImpl** 中打上断点，在启动BIO的服务端之后立即启动客户端。具体的Debug断点如下：

![Socket 的 setImpl](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712160311.png)

通过单步调试，我们在**BioServerSocket** 中看到两个对象是不一样的。

![BioServerSocket](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712160613.png)

![对象对比](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712160628.png)

为什么不一样呢？这里需要回顾前面的【ServerSocket中accept 解读】这一部分的操作。这里把一些重要操作标记了一下：

![ServerSocket中accept解读可能的理解误区1](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712161314.png)

这里复习之前提到的内容，在**accept();** 中为了确保Socket连接是正确并且可用的，每次都会`new Socket()`，而这里的`SocksSocketImpl` 是属于 **Socket** 的成员变量。

在进行Socket套接字连接之前会先判断是否初始化，如果没有就先进行初始化（具体可以看红框框的位置）。

如果还是理解不了，那么只能再次寄出另一张杀手锏图了：

![ServerSocket中accept解读可能的理解误2](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230712163105.png)

# 实现 Socket 中的 read 方法非阻塞

前面介绍了服务端的**AbstractPlainSocketImpl**如何实现`socketRead`方法非阻塞的 ，具体做法其实就是使用 **AbstractPlainSocketImpl** 传入了 **timeout** 参数，实现 **SocketInputStream** 非阻塞`read`。

这样表面上看上去 **read** 方法是非阻塞的，实际上这里存在一个明显的 **误区**，那就是在`socket = serverSocket.accept();`这一段代码中，服务端构建出 Socket 连接之后，客户端和服务端交互是通过独立的`Socket`对象完成IO读写的。

然而在第一次改造过后，中实际上还有两点不易察觉的问题：

（1）服务端`read`的非阻塞轮询效率非常低，基本上是“一核繁忙、多核围观”的情况。

（2）第一次改造设置的是设定的是**ServerSocket级别**的**SocksSocketImpl**的timeout。而每个新的客户端进来都是新的Socket连接，每个Socket又有各自的 **SocksSocketImpl**，这里**客户端连接所产生新的Socket**的**timeout**是没有做设置的，换句话说，服务端针对每个Socket的read依然是完全阻塞。

前文提到，在BIO非阻塞同步模型中，我们虽然没法解决 "同步" 这一点，但是我们可以让“非阻塞”这一块更为优化合理和更为高效一些。

第一个问题的解决策略是启动**多线程**以非阻塞`read()`方式轮询，这样做的另一点好处是，某个Socket读写压力大并不会影响CPU 切到其他线程的正常工作。

第二点的解决方式在前面已经反复提到了，我们需要为**每个新的Socket**设置 **timeout**，非阻塞 `read`的条件才算是真正成立。

经过上面的层层铺垫，下面我们来看下第二版优化代码实现：

```java
 /**
     * 1. NIO 改写，accept 非阻塞
     * 2. 实现 read() 同样非阻塞
     *
     * @param port
     * @return void
     * @description
     * @author xander
     * @date 2023/7/12 16:38
     */
    public void initNioAndNioReadServer(int port) {
        ServerSocket serverSocket = null;//服务端Socket
        Socket socket = null;//客户端socket
        BufferedReader reader = null;
        ExecutorService threadPool = Executors.newCachedThreadPool();
        String inputContent;
        int count = 0;
        try {
            serverSocket = new ServerSocket(port);
            // 1. 需要设置超时时间，会等待设置的时间之后再进行返回
            serverSocket.setSoTimeout(SO_TIMEOUT);
            log.info(stringNowTime() + ": serverSocket started");
            while (true) {
                // 2. 如果超时没有获取，这里会抛出异常，这里的处理策略是不处理异常
                try {
                    socket = serverSocket.accept();
                } catch (SocketTimeoutException e) {
                    //运行到这里表示本次accept是没有收到任何数据的，服务端的主线程在这里可以做一些其他事情
                    log.info("now time is: " + stringNowTime());
                    continue;
                }
                // 3. 拿到Socket 之后，应该使用线程池新开线程方式处理客户端连接，提高CPU利用率。
                Thread thread = new Thread(new ClientSocketThread(socket));
                threadPool.execute(thread);
//                log.info(stringNowTime() + ": id为" + socket.hashCode() + "的Clientsocket connected");
//                reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
//
//                while ((inputContent = reader.readLine()) != null) {
//                    log.info("收到 id为" + socket.hashCode() + "  " + inputContent);
//                    count++;
//                }
//                log.info("id为" + socket.hashCode() + "的Clientsocket " + stringNowTime() + "读取结束");
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (Objects.nonNull(reader)) {
                    reader.close();
                }
                if (Objects.nonNull(socket)) {

                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }

    /**
     * 改写 客户端 Socket 连接为单独线程处理
     */
    class ClientSocketThread implements Runnable {

        private static final int SO_TIMEOUT = 2000;
        
        private static final int SLEEP_TIME = 1000;

        public final Socket socket;

        public ClientSocketThread(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            BufferedReader reader = null;
            String inputContent;
            int count = 0;
            try {
                socket.setSoTimeout(SO_TIMEOUT);
            } catch (SocketException e1) {
                e1.printStackTrace();
            }
            try {
                reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                while (true) {
                    try {
                        while ((inputContent = reader.readLine()) != null) {
                            log.info("收到id为" + socket.hashCode() + "  " + inputContent);
                            count++;
                        }
                    } catch (Exception e) {
                        //执行到这里表示read方法没有获取到任何数据，线程可以执行一些其他的操作
                        log.info("Not read data: " + stringNowTime());
                        continue;
                    }
                    //执行到这里表示读取到了数据，我们可以在这里进行回复客户端的工作
                    log.info("id为" + socket.hashCode() + "的Clientsocket " + stringNowTime() + "读取结束");
                    Thread.sleep(SLEEP_TIME);
                }
            } catch (IOException | InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                try {
                    if (Objects.nonNull(reader)) {
                        reader.close();
                    }
                    if (Objects.nonNull(socket)) {
                        socket.close();
                    }
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }

    }
```

经过上面的改造，我们基本把 BIO 同步阻塞的工作方式更新为 **同步非阻塞**的工作方式，核心是对于 `read()`以及服务端接收新连接的`accept()`设置`timeout`参数，在外部处理上则通过`while(true)` 加上“吞异常”结合`sleep()`的老套路实现“非阻塞”。

当然我们也可以看到，通过线程池每次都构建新线程的方式，在连接比较少的时候是比较高效的，但是一旦连接暴增，理论上JVM虽然可以构建非常多线程，实际上CPU肯定是吃不消，多线程“空轮询”判断的方式也十分浪费CPU资源，多线程切换起来更是雪上加霜。

基于BIO的种种弊端，Java 官方在JDK1.4 提供了 NIO 解决上面的几点问题。

# native `accept`方法在Linux运作解读

[accept(2): accept connection on socket - Linux man page (die.net)](https://linux.die.net/man/2/accept)

原始文档相关解读：[[【Linux】accept(2) - Linux man page]]，下面的内容基本为文档的翻译和理解介绍。

`accept()`本地方法，我们可以来试着看一看Linux这块的相关解读：

```java
#include <sys/types.h>

#include <sys/socket.h>

int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);

```

`accept()`系统调用主要用在基于连接的套接字类型，比如**SOCK_STREAM**和**SOCK_SEQPACKET**。它提取出所监听套接字的等待连接队列中第一个连接请求，**创建一个新的套接字**，并返回指向该套接字的文件描述符。新建立的套接字不在监听状态，原来所监听的套接字也不受该系统调用的影响。

**备注：新建立的套接字准备发送`send()`和接收数据`recv()`。**

**sockfd**，作用是 利用系统调用`socket()`建立的套接字描述符，通过`bind()`绑定到一个本地地址(一般为服务器的套接字)，并且通过`listen()`一直在监听连接；

**addr**, 指向`struct sockaddr`的指针，该结构用通讯层服务器对等套接字的地址(一般为客户端地址)填写，返回地址`addr`的确切格式由套接字的地址类别(比如TCP或UDP)决定；

若`addr`为NULL，没有有效地址填写，这种情况下，addrlen也不使用，应该置为NULL；

**备注：addr是个指向局部数据结构sockaddr_in的指针，这就是要求接入的信息本地的套接字(地址和指针)。**

`addrlen`， 代表一个值结果参数，调用函数必须初始化为包含addr所指向结构大小的数值，函数返回时包含对等地址(一般为服务器地址)的实际数值；

**备注：addrlen是个局部整形变量，设置为`sizeof(struct sockaddr_in)`。**

如果队列中没有等待的连接，套接字也没有被标记为Non-blocking，accept()会阻塞调用函数直到连接出现；如果套接字被标记为Non-blocking，队列中也没有等待的连接，accept()返回错误**EAGAIN**或**EWOULDBLOCK**。

**备注：一般来说accept()为阻塞函数，当监听socket调用accept()时，它先到自己的receive_buf中查看是否有连接数据包；若有，把数据拷贝出来，删掉接收到的数据包，创建新的socket与客户发来的地址建立连接；若没有，就阻塞等待；**

为了在套接字中有到来的连接时得到通知，可以使用**select()** 或**poll()**。当尝试建立新连接时，系统发送一个可读事件，然后调用`accept()`为该连接获取套接字。另一种方法是，当套接字中有连接到来时设定套接字发送SIGIO信号。

返回值成功时，返回非负整数，该整数是接收到套接字的描述符；**出错时会返回－1**，相应地设定全局变量error。

所以，在Java部分的源码里（**java.net.ServerSocket#accept**）会new 一个Socket出来，方便连接后拿到的新Socket的文件描述符的信息给设定到我们new出来的这个Socket 上来，这点在`java.net.PlainSocketImpl#socketAccept`中看到的尤为明显，读者可以回顾相关源码。

# 总结

本文一开始介绍了Bio Socket的基本代码，接着从`ServerSocket`的`bind`方法解读，通过图文结合的方式介绍了源码如何处理，整个`bind`操作过程中有许多`native`层调用，所以Socket的代码调试是非常麻烦的。

介绍完`bind`之后，我们接着介绍了`ServerSocket`中`accept`方法，并且介绍了`accept` 方法的阻塞问题，实际上和底层的操作系统行为有关，并且通过画图的方式理解`accept`中Socket连接比较“绕”的操作。

最后，文章的后半部分介绍了如何改造`accept`以及客户端的`Socket`连接解决非阻塞问题IO，最后我们介绍了  `native accept`方法在Linux运作，主要内容为Linux的相关文档理解。

# 写在最后

理解Socket的非阻塞操作有助于理解 NIO的Channel和Buffer的概念，实际上从我们的Demo代码可以看到Channel和非阻塞的BIO思路比较类似，而BufferReader缓冲流则贴合了 Buffer 的概念。

# 参考资料


[Linux Network Programming, Part 1 (linuxjournal.com)](https://www.linuxjournal.com/files/linuxjournal.com/linuxjournal/articles/023/2333/2333s2.html)

[详解socket中的backlog 参数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/104874605)

[BIO到NIO源码的一些事儿之BIO - 掘金 (juejin.cn)](https://juejin.cn/post/6844903751178780686)

## CachedThreadPool的工作原理

源码：

```java
public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS, //60s 
                                  new SynchronousQueue<Runnable>());  
}
```

（1）corePoolSize = 0，maximumPoolSize = 最大值（无限大），keepAliveTime = 60s，workQueue = **SynchronousQueue**

（2）**SynchronousQueue**（实际上没有存储数据的空闲，是用来做多线程通信之间的协调作用的）。一开始提交一个任务过来，要求线程池里**必须有一个线程对应可以处理这个任务**，但是此时一个线程都没有，poolSize >= corePoolSize , workQueue已经满了，poolSize < maximumPoolSize（最大值），直接就会**创建一个新的线程来处理这个任务**。

> 这样的效果也就是来一个任务就开一个线程，无界，无限开新线程，线程过多容易导致JVM的压力过大甚至直接崩溃。这也是为什么阿里巴巴规范禁掉这个方法的直接原因，容易误用。

（3）如果短期内有大量的任务都涌进来，实际上是走一个直接提交的思路，对每个任务，如果没法找到一个空闲的线程来处理它，那么就会立即创建一个新的线程出来，来处理这个新提交的任务

（4）短时间内，如果大量的任务涌入，可能会导致瞬间创建出来几百个线程，几千个线程，是不固定的。

（5）但是当这些线程工作完一段时间之后，就会处于空闲状态，就会看超过60s的空闲，就会直接将空闲的线程给释放掉。