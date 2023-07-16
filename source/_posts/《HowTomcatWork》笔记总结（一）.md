---
title: 《HowTomcatWork》笔记总结（一）
subtitle: howTomcatWork的书籍笔记内容
author: 阿东
url_suffix: howtomcat
tags:
  - Java
categories:
  - Java
keywords: howtomcatwork
description: 笔记总结
copyright: true
date: 2022-02-20 22:49:55
---

# 《HowTomcatWork》笔记总结（一）

# 前言

​	这一篇是howTomcatWork的书籍笔记内容。下面是根据书中的内容做的一部分个人笔记。

# 书籍地址：

> 链接：https://pan.baidu.com/s/1jazo55sA_hCP9_2Q_2NBrQ 
> 提取码：lp96 
> --来自百度网盘超级会员V6的分享



# 个人评价

​	没啥好说的，tomcat作者写的书，看了下海淘居然要500多确实吓到了。虽然代码使用的是tomcat5的版本，但是可以基本理解tomcat的内部工作机制。也可以看到作者是如何用一个十多行的肉鸡服务器代码不断升级成为现在的tomcat的模样。

​	本文是个人根据看书记录的一些笔记，中间逻辑不一定连贯，因为有些内容过于基础没有记录的价值，所以挑了一些个人关注的点。

<!-- more -->

# 一个最简单的Servlet是如何工作的

1. 创建Request 对象，并且解析HTTP的请求信息，通过Request对象封装了这些信息的具体细节

具体接口：

> `javax.servlet.ServletRequest` 或` javax.servlet.http.ServletRequest`

2. 创建Response对象,封装了客户需需要的真正数据，封装了响应体的相关信息

>  `javax.servlet.ServletResponse` 或`javax.servlet.http.ServletResponse`

3. servlet的`service` 方法，根据此方法对于请求头进行解析，同时创建response将数据回传给客户端



# tomcat的基本结构

​	Tomcat把服务器在大体上可以拆分为两部分，一部分叫做**容器**，另一部分叫做**连接器**

## 连接器

作用：接收到每一个 HTTP 请求构造一个 ==request== 和 ==response== 对象

## 容器

作用：接受连接器的请求根据`service`方法进行响应给对应的客户端

>  Tomcat 4 和 和  5的主要区别
>
> + Tomcat 5 支持 Servlet 2.4 和 JSP 2.0 规范，而 Tomcat 4 支持 Servlet 2.3 和 JSP 1.2。
>
> + 比起 Tomcat 4，Tomcat 5 有一些更有效率的默认连接器。
>
> + Tomcat 5 共享一个后台处理线程，而 Tomcat 4 的组件都有属于自己的后台处理线程。
>   因此，就这一点而言，Tomcat 5 消耗较少的资源。
>
> + Tomcat 5 并不需要一个映射组件(mapper component)用于查找子组件，因此简化了代码。



# 构建一个最简单的web程序

## 构建对象

​	下方的代码简单阅读即可，无需自己动手实验

### HttpServer

 用于构建一个服务器，同时建立serverSocket套接字等待链接

+ 调用`httprequest.parse()`方法

代码如下：

```java
public class HttpServer {

    /**
     * 关闭容器的请求路径
     */
    private static final String SHUTDOWN = "/SHUTDOWN";

    /**
     * 静态资源根路径
     */
    public static final String WEBROOT = System.getProperty("user.dir") + File.separator + "webroot";

    /**
     * 是否关闭标识
     */
    private boolean SHUTDOWN_FLAG = false;

    public static void main(String[] args) {
        new HttpServer().await();
    }

    /**
     * 具体的server 方法，等待socket请求
     */
    public void await() {
        // 默认为8080端口
        int port = 8080;
        String host = "127.0.0.1";
        ServerSocket serverSocket = null;
        try {
            // 创建套接字
            serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(-1);
        }

        while (!SHUTDOWN_FLAG) {
            try {
                // 等待套接字
                Socket accept = serverSocket.accept();
                HttpRequest httpRequest = new HttpRequest(accept.getInputStream());
                // 处理请求数据
                httpRequest.parse();
                // 创建响应对象，处理响应信息
                HttpResponse httpResponse = new HttpResponse(accept.getOutputStream());
                // 设置静态资源
                httpResponse.setRequest(httpRequest);
                httpResponse.setResource();
                // 关闭的套接字
                accept.close();
                // 判断请求Url是否为 /shutdown
                SHUTDOWN_FLAG = httpRequest.getUri().equalsIgnoreCase(SHUTDOWN);
            } catch (IOException e) {
                e.printStackTrace();
                continue;
            }
        }

    }
}

```

### HttpRequest 

​	以httpserver 的请求inputstream, 解析请求内容，分解请求uri

​	使用`parse()`方法解析请求信息，设置到stringbuffer里面

​	使用`parseUri(str)`截取请求信息的请求uri,设置到属性里面

```java
public class HttpRequest {

    /**
     * 缓冲区的大小为 1M
     */
    private static final int BUFFER_COUNT = 1024;

    /**
     * 请求路径
     */
    private String uri;

    /**
     * 请求流
     */
    private InputStream inputStream;

    public HttpRequest(InputStream inputStream) {
        this.inputStream = inputStream;
    }

    /**
     * 解析inputstream 对于内容进行解析
     */
    public void parse() {
        // 字符串缓冲池
        StringBuffer stringBuffer = new StringBuffer(BUFFER_COUNT);

        byte[] byteBuffer = new byte[BUFFER_COUNT];

        if (inputStream == null) {
            System.err.println("未找到套接字");
            return;
        }

        int read = 0;
        try {
            // 读取数据到byte数组
            read = inputStream.read(byteBuffer);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(-1);
        }
        //读取byte数组的数据进入到stringbuffer
        for (int i = 0; i < read; i++) {
            stringBuffer.append((char)byteBuffer[i]);
        }
        // 打印stringbuffer
        System.err.println(stringBuffer.toString());
        // 获取uri
        uri = parseUri(stringBuffer.toString());
    }

    /**
     * 解析请求，获取请求Uri
     * @param requestString 需要处理的uri
     */
    public String parseUri(String requestString){
        // 建立index1 和 2
        int index1, index2;
        // 获取到第一个空行
        index1 = requestString.indexOf(' ');
        if(index1 != -1){
            // 从index1 开始找
            index2 = requestString.indexOf(' ', index1 + 1);
            if(index2 > index1){
                // 获取请求路径
                return requestString.substring(index1 + 1, index2);
            }
        }
        return null;

    }


    public String getUri() {
        return uri;
    }
}
```



### HttpResonse 

​	以 httpserver 的请求outputstream ，获取输入流，将数据返回给客户端

​	关键方法为`setResouces`，获取请求Uri，同时使用file 读取文件

```java
public class HttpResponse {

    /**
     * 组合httprequest
     * 根据request返回对应到信息
     */
    private HttpRequest request;

    /**
     * 输出流
     */
    private OutputStream outputStream;

    /**
     * 缓冲区大小
     */
    private static final int BUFFER_COUNT = 1024;


    public HttpResponse(OutputStream outputStream) {
        this.outputStream = outputStream;
    }

    /**
     * 设置静态资源
     */
    public void setResource() throws IOException {
        String errMsg = "404 msg";
        // 字节缓存区
        byte[] bytes = new byte[BUFFER_COUNT];
        // 读取静态资源
        File file = new File(HttpServer.WEBROOT, request.getUri());
        if (file.exists()) {
            // 文件流
            try {
                FileInputStream fileInputStream = new FileInputStream(file);
                // 读取字节
                int ch = fileInputStream.read(bytes, 0, BUFFER_COUNT);
                // 输出
                while (ch != -1) {
                    // 写入流
                    outputStream.write(bytes, 0, ch);
                    // 重复读取数据到缓冲区
                    ch = fileInputStream.read(bytes, 0, BUFFER_COUNT);
                }

            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (outputStream != null) {
                    outputStream.close();
                }
            }
        } else {
            try {
                outputStream.write(errMsg.getBytes());
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (outputStream != null) {
                    outputStream.close();
                }
            }
        }
    }

    /**
     * 设置request
     *
     * @param httpRequest
     */
    public void setRequest(HttpRequest httpRequest) {
        this.request = httpRequest;
    }
}

```



## 基本步骤：

下面是代码的基本交互步骤：

1. 创建httpserver 对象
2. 绑定端口和主机，建立套接字连接
3. `accept()`方法等待请求，阻塞当前线程
4. 创建reuqest 请求对象
5. 获取`inputstream `解析请求信息
6. 获取请求到uri,设置到Request对象
7. 创建 response 响应对象
8. 设置响应对象的request，拿到uri, 同时使用Io，读取请求对应到文件
9. `outputstream` 解析文件流数据，使用write 返回到客户端
10. 无论成功还是失败，关闭流（重要）



## 补充内容：

​	这里重点关注关于套接字的一些知识

### ServerSocket

​	服务器套接字的另一个重要的属性是 **backlog**，这是服务器套接字开始==拒绝==传入的请求之前，传入的连接请求的==最大队列长度==。

### parseUri() 处理逻辑

`GET /index.html HTTP/1.1`。parse 方法从传递给 Requst 对象的套接字的 InputStream 中读取整个字节流并在一个缓冲区中存储字节数组。然后它使用缓冲区字节数据的字节来填入一个 StringBuffer 对象，并且把代表 StringBuffer 的字符串传递给 parseUri 方法。

## Http请求对于servlet的操作

​	当第一次调用 servlet 的时候，加载该 servlet 类并调用 servlet 的 init 方法(仅仅一次)。

+ 对每次请求，构造一个 javax.servlet.ServletRequest 实例和一个
  javax.servlet.ServletResponse 实例。
+  调用 servlet 的 service 方法，同时传递 ServletRequest 和 ServletResponse 对象。
+   当 servlet 类被关闭的时候，调用 servlet 的 destroy 方法并卸载 servlet 类。
  本章的第一个servlet容器不是全功能的。因此，她不能运行什么除了非常简单的servlet，
  而且也不调用 servlet 的 init 方法和 destroy 方法。相反它做了下面的事情：
+   等待 HTTP 请求。
+   构造一个 ServletRequest 对象和一个 ServletResponse 对象。
+   假如该请求需要一个静态资源的话，调用 StaticResourceProcessor 实例的 process 方
  法，同时传递 ServletRequest 和 ServletResponse 对象。
+   假如该请求需要一个 servlet 的话，加载 servlet 类并调用 servlet 的 service 方法，
  同时传递 ServletRequest 和 ServletResponse 对象

## StringManager

### 特点：

1. 使用单例模式
2. 每个实例会读取包对应的一个属性文件
3. StringManager 类被设计成一个 StringManager实例可以被包里边的所有类共享
4. getManager() 方法被 同步修饰，并且使用hashtable对于manager进行管理（tomcat4）

### 核心方法解释

**SocketInputStream**：套接字读取流，主要用于处理Http请求中的各种参数，为了提高效率，使用懒加载特性读取

1. 回收检查流数据
2. 检查空行，如果出现-1抛出结尾异常
3. 获取servlet方法名称
   1. 如果缓冲区已经满了，则进行扩展
      1. 我们在内部缓冲区的尽头
      2. 如果到了缓冲区结尾，将指针归位
   2. 这里有一个关键点：**System.arraycopy** 用来扩展缓冲区
4. 阅读协议



# 模块解释

​	关于部分模块的相关解释。

## 解析头部

+ 你可以通过使用类的无参数构造方法构造一个 HttpHeader 实例。
+   一旦你拥有一个HttpHeader实例，你可以把它传递给SocketInputStream的readHeader
  方法。假如这里有头部需要读取，readHeader 方法将会相应的填充 HttpHeader 对象。
  假如再也没有头部需要读取了，HttpHeader实例的nameEnd和valueEnd字段将会置零。
+   为了获取头部的名称和值，使用下面的方法：
+   String name = new String(header.name, 0, header.nameEnd);
+   String value = new String(header.value, 0, header.valueEnd);

## 启动器

+ 启动应用程序
+   连接器
+   创建一个 HttpRequest 对象
+   创建一个 HttpResponse 对象
+   静态资源处理器和 servlet 处理器

+ 运行应用程序

startup 模块只有一个类，Bootstrap，用来启动应用的。connector 模块的类可以分为五组：

+ 连接器和它的支撑类(HttpConnector 和 HttpProcessor)。

+ 指代 HTTP 请求的类(HttpRequest)和它的辅助类。
+   指代 HTTP 响应的类(HttpResponse)和它的辅助类。
+   Facade 类(HttpRequestFacade 和 HttpResponseFacade)。
+   Constant 类



# 问题以及解决

​	如何避免在servlet调用连接器的时候，不需要请求参数可以避免掉getParamMap,getAttribute等昂贵开销的操作？

​	Tomcat 的默认连接器(和本章应用程序的连接器)试图不解析参数直到 servlet 真正需要它的时候，通过这样来获得更高效率



# 小知识补充

1. System 在打印的时候 print 方法不会刷新输出。
2. 在一个 servlet 容器里边，一个类加载器可以找到 servlet 的地方被称为资源库(repository）。
3. 通过外观模式将Request对象的细节隐藏，setvlet调用内部无法知道，但是在解析的时候依然可以相互通信，只需要使用faced将接口进行一层包裹， 即可保证getUri()方法安全性



# 总结

​	书中第一个章节内容比较简单，后续章节代码的难度会逐渐上升，同时使用了不少的设计模式也是需要多加阅读理解和消化的



# 写在最后

​	这篇笔记目的是让更多人了解这本书，这本书算是一本神书，毕竟开发的原作者自己写的东西毫无疑问是一手知识了。