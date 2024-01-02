---
title: 【Tomcat】《How Tomcat Works》英文版GPT翻译（第一章）
subtitle: 这个人很懒，不想写副标题
url_suffix: note
author: lazytime
tags:
  - 无
categories:
  - 未分类
keywords: 请输入关键字（英文逗号分隔多个关键字）
description: 请输入描述信息
copyright: true
date: 2024-01-02 16:40:22
---


# 回顾

[[【Tomcat】《How Tomcat Works》英文版GPT翻译（序章）]]

# Chapter 1: A Simple Web Server 

This chapter explains how Java web servers work. A web server is also called a Hypertext Transfer Protocol (HTTP) server because it uses HTTP to communicate with its clients, which are usually web browsers. A Java-based web server uses two important classes: java.net.Socket and java.net.ServerSocket, and communications are done through HTTP messages. It is therefore natural to start this chapter with a  discussion of HTTP and the two classes. Afterwards, it goes on to explain the simple web server application that accompanies this chapter.

本章介绍了Java Web服务器的工作原理。Web服务器也被称为超文本传输协议（HTTP）服务器，因为它使用HTTP与其客户端进行通信，通常是Web浏览器。基于Java的Web服务器使用两个重要的类：java.net.Socket和java.net.ServerSocket，并通过HTTP消息进行通信。因此，在本章开始时自然而然地讨论了HTTP和这两个类。之后，它继续解释了附带本章的简单Web服务器应用程序。

# The Hypertext Transfer Protocol (HTTP) 

HTTP is the protocol that allows web servers and browsers to send and receive data over the Internet. It is a request and response protocol. The client requests a file and the server responds to the request. HTTP uses reliable TCP connections—by default on TCP port 80. The first version of HTTP was HTTP/0.9, which was then overridden by HTTP/1.0. Replacing HTTP/1.0 is the current version of HTTP/1.1, 
which is defined in Request for Comments (RFC) 2616 and downloadable from http://www.w3.org/Protocols/HTTP/1.1/rfc2616.pdf.

HTTP是一种协议，它允许Web服务器和浏览器在互联网上发送和接收数据。它是一种请求和响应的协议。客户端请求一个文件，服务器对请求进行响应。HTTP使用可靠的TCP连接，默认情况下在TCP端口80上。HTTP的第一个版本是HTTP/0.9，然后被HTTP/1.0取代。取代HTTP/1.0的是当前版本的HTTP/1.1，该版本在《请求评论》（RFC）2616中定义，并可从 http://www.w3.org/Protocols/HTTP/1.1/rfc2616.pdf 下载。

> Note This section covers HTTP 1.1 only briefly and is intended to help you 
> understand the messages sent by web server applications. If you are interested 
> in more details, read RFC 2616.
> 
> 注意 本节仅简要介绍 HTTP 1.1，旨在帮助您 
> 了解网络服务器应用程序发送的信息。如果您对 
> 更多详细信息，请阅读 RFC 2616。

In HTTP, it is always the client who initiates a transaction by establishing a connection and sending an HTTP request. The web server is in no position to contact a client or make a callback connection to the client. Either the client or the server can prematurely terminate a connection. For example, when using a web browser you can click the Stop button on your browser to stop the download process of a file, effectively closing the HTTP connection with the web server

在HTTP中，始终是客户端通过建立连接并发送HTTP请求来发起一次事务。Web服务器无法联系客户端或向客户端进行回调连接。无论是客户端还是服务器都可以提前终止连接。例如，当使用Web浏览器时，您可以点击浏览器上的停止按钮来停止文件的下载过程，从而有效地关闭与Web服务器的HTTP连接。

<!-- more -->

# HTTP Requests 

An HTTP request consists of three components:

一个HTTP请求由三个组成部分组成：

- Method—Uniform Resource Identifier (URI)—Protocol/Version
-  Request headers
-  Entity body

- 方法 - 统一资源标识符（URI） - 协议/版本
- 请求头
- 实体主体

An example of an HTTP request is the following:

一个HTTP请求的示例如下：

```
OST /examples/default.jsp HTTP/1.1
Accept: text/plain; text/html
Accept-Language: en-gb
Connection: Keep-Alive
Host: localhost
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98)
Content-Length: 33
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
lastName=Franks&firstName=Michael
```

The method—URI—protocol version appears as the first line of the request

请求的方法-URI-协议版本出现在请求的第一行

```
POST /examples/default.jsp HTTP/1.1
```

where POST is the request method, /examples/default.jsp represents the URI and 
HTTP/1.1 the Protocol/Version section.

其中POST是请求方法，/examples/default.jsp表示URI，而HTTP/1.1表示协议/版本部分。

Each HTTP request can use one of the many request methods as specified in the HTTP standards. The HTTP 1.1 supports seven types of request: GET, POST, HEAD, OPTIONS, PUT, DELETE, and TRACE. GET and POST are the most commonly used in Internet applications.

每个HTTP请求可以使用HTTP标准中指定的多种请求方法之一。HTTP 1.1支持七种类型的请求：GET、POST、HEAD、OPTIONS、PUT、DELETE和TRACE。在互联网应用中，GET和POST是最常用的。

The URI specifies an Internet resource completely. A URI is usually interpreted as being relative to the server's root directory. Thus, it should always begin with a forward slash /. A Uniform Resource Locator (URL) is actually a type of URI (see http://www.ietf.org/rfc/rfc2396.txt). The protocol version represents the version of the HTTP protocol being used.

URI完全指定了一个Internet资源。URI通常被解释为相对于服务器的根目录。因此，它应该始终以斜杠/开头。统一资源定位符（URL）实际上是URI的一种类型（参见http://www.ietf.org/rfc/rfc2396.txt）。协议版本表示正在使用的HTTP协议的版本。

The request header contains useful information about the client environment and the entity body of the request. For example, it could contain the language the browser is set for, the length of the entity body, and so on. Each header is separated by a carriage return/linefeed (CRLF) sequence.

请求头包含有关客户端环境和请求的实体主体的有用信息。例如，它可以包含浏览器设置的语言、实体主体的长度等。每个头部都由回车/换行符（CRLF）序列分隔。

Between the headers and the entity body, there is a blank line (CRLF) that is important to the HTTP request format. The CRLF tells the HTTP server where the entity body begins. In some Internet programming books, this CRLF is considered the fourth component of an HTTP request. In the previous HTTP request, the entity body is simply the following line:

在头部和实体主体之间，有一个空行（CRLF），这对于HTTP请求格式很重要。CRLF告诉HTTP服务器实体主体从哪里开始。在一些互联网编程书籍中，这个CRLF被认为是HTTP请求的第四个组成部分。在前面的HTTP请求中，实体主体只是以下一行：

```
lastName=Franks&firstName=Michael
```

The entity body can easily become much longer in a typical HTTP request.HTTP Responses

实体主体在典型的HTTP请求中很容易变得更长。

Similar to an HTTP request, an HTTP response also consists of three parts:

与HTTP请求类似，HTTP响应也由三个部分组成：

- Protocol—Status code—Description

- Response headers

- Entity body

- 协议 - 状态码 - 描述

- 响应头

- 实体主体

The following is an example of an HTTP response:

下面是一个 HTTP 响应示例：

```html
HTTP/1.1 200 OK

Server: Microsoft-IIS/4.0

Date: Mon, 5 Jan 2004 13:13:33 GMT

Content-Type: text/html

Last-Modified: Mon, 5 Jan 2004 13:13:12 GMT

Content-Length: 112

<html>

<head>

<title>HTTP Response Example</title>

</head>

<body>

Welcome to Brainy Software

</body>

</html>
```

The first line of the response header is similar to the first line of the request header. The first line tells you that the protocol used is HTTP version 1.1, the request succeeded (200 = success), and that everything went okay. The response headers contain useful information similar to the headers in the request. The entity body of the response is the HTML content of the response itself. The headers and the entity body are separated by a sequence of CRLFs.

响应头的第一行与请求头的第一行类似。第一行告诉您所使用的协议是HTTP版本1.1，请求成功（200 = 成功），一切都很顺利。响应头包含了与请求中头部类似的有用信息。响应的实体主体是响应本身的HTML内容。头部和实体主体之间由一系列CRLF（回车换行符）分隔。

# The Socket Class

A socket is an endpoint of a network connection. A socket enables an application to read from and write to the network. Two software applications residing on two different computers can communicate with each other by sending and receiving byte streams over a connection. To send a message from your application to another application, you need to know the IP address as well as the port number of the socket of the other application. In Java, a socket is represented by the java.net.Socket class.

套接字是网络连接的端点。套接字使应用程序能够从网络中读取和写入数据。两个位于不同计算机上的软件应用程序可以通过在连接上发送和接收字节流来相互通信。要将消息从您的应用程序发送到另一个应用程序，您需要知道另一个应用程序的套接字的IP地址和端口号。在Java中，套接字由java.net.Socket类表示。

To create a socket, you can use one of the many constructors of the Socket class. One of these constructors accepts the host name and the port number:

要创建一个套接字，您可以使用Socket类的许多构造函数之一。其中一个构造函数接受主机名和端口号作为参数：

```java
public Socket (java.lang.String host, int port)
```

where host is the remote machine name or IP address and port is the port number of the remote application. For example, to connect to yahoo.com at port 80, you would construct the following Socket object:

其中 host 是远程计算机名称或 IP 地址，port 是远程应用程序的端口号。例如，要连接端口为 80 的 yahoo.com，需要构建以下 Socket 对象：

```java
new Socket ("yahoo.com", 80);
```

Once you create an instance of the Socket class successfully, you can use it to send and receive streams of bytes. To send byte streams, you must first call the Socket class's getOutputStream method to obtain a java.io.OutputStream object. To send text to a remote application, you often want to construct a java.io.PrintWriter object from the OutputStream object returned. To receive byte streams from the other end of the connection, you call the Socket class's getInputStream method that returns a java.io.InputStream.

套接字是网络连接的端点。套接字使应用程序能够从网络中读取和写入数据。两个位于不同计算机上的软件应用程序可以通过在连接上发送和接收字节流来相互通信。要将消息从您的应用程序发送到另一个应用程序，您需要知道另一个应用程序的套接字的IP地址和端口号。在Java中，套接字由java.net.Socket类表示。

The following code snippet creates a socket that can communicate with a local HTTP server (127.0.0.1 denotes a local host), sends an HTTP request, and receives the response from the server. It creates a StringBuffer object to hold the response and prints it on the console.

下面的代码片段创建了一个可以与本地 HTTP 服务器（127.0.0.1 表示本地主机）通信的套接字，发送 HTTP 请求并接收服务器的响应。它创建了一个 StringBuffer 对象来保存响应，并将其打印在控制台上。

```java
  
    Socket socket = new Socket("127.0.0.1", "8080");  
    OutputStream os = socket.getOutputStream();  
    boolean autoflush = true;  
    PrintWriter out = new PrintWriter(  
            socket.getOutputStream(), autoflush);  
    BufferedReader in = new BufferedReader(  
            new InputStreamReader( socket.getInputstream() ));  
// send an HTTP request to the web server  
 out.println("GET /index.jsp HTTP/1.1");  
         out.println("Host: localhost:8080");  
         out.println("Connection: Close");  
         out.println();  
         // read the response  
         boolean loop = true;  
         StringBuffer sb = new StringBuffer(8096);  
         while (loop) {  
         if ( in.ready() ) {  
         int i=0;  
         while (i!=-1) {  
         i = in.read();  
         sb.append((char) i);  
         }  
         loop = false;  
         }  
         Thread.currentThread().sleep(50);  
         }  
         // display the response to the out console  
         System.out.println(sb.toString());  
         socket.close();
```

Note that to get a proper response from the web server, you need to send an HTTP request that complies with the HTTP protocol. If you have read the previous section, The Hypertext Transfer Protocol (HTTP), you should be able to understand the HTTP request in the code above.

请注意，为了从网络服务器获得正确的响应，您需要发送符合HTTP协议的HTTP请求。如果您已经阅读了前面的章节《超文本传输协议（HTTP）》，您应该能够理解上述代码中的HTTP请求。

> Note You can use the com.brainysoftware.pyrmont.util.HttpSniffer class included with this book to send an HTTP request and display the response. To use this Java program, you must be connected to the Internet. Be warned, though, that it may not work if you are behind a firewall.
> 
> 注意：您可以使用本书附带的com.brainysoftware.pyrmont.util.HttpSniffer类发送HTTP请求并显示响应。要使用这个Java程序，您必须连接到互联网。但请注意，如果您启动防火墙，它可能无法正常工作。
# The ServerSocket Class

The Socket class represents a "client" socket, i.e. a socket that you construct whenever you want to connect to a remote server application. Now, if you want to implement a server application, such as an HTTP server or an FTP server, you need a different approach. This is because your server must stand by all the time as it does not know when a client application will try to connect to it. In order for your application to be able to stand by all the time, you need to use the java.net.ServerSocket class. This is an implementation of a server socket. 

Socket类表示一个“客户端”套接字，即当您想要连接到远程服务器应用程序时构建的套接字。现在，如果您想要实现一个服务器应用程序，比如一个HTTP服务器或FTP服务器，您需要采用不同的方法。这是因为您的服务器必须始终保持待命状态，因为它不知道何时会有客户端应用程序尝试连接到它。为了使您的应用程序能够始终保持待命状态，您需要使用java.net.ServerSocket类。这是一个服务器套接字的实现。

ServerSocket is different from Socket. The role of a server socket is to wait for connection requests from clients. Once the server socket gets a connection request, it creates a Socket instance to handle the communication with the client.

ServerSocket与Socket不同。服务器套接字的作用是等待来自客户端的连接请求。一旦服务器套接字收到连接请求，它就会创建一个Socket实例来处理与客户端的通信。

To create a server socket, you need to use one of the four constructors the ServerSocket class provides. You need to specify the IP address and port number the server socket will be listening on. Typically, the IP address will be 127.0.0.1, meaning that the server socket will be listening on the local machine. The IP address the server socket is listening on is referred to as the binding address. Another important property of a server socket is its backlog, which is the maximum queue length of incoming connection requests before the server socket starts to refuse the incoming requests. One of the constructors of the ServerSocket class has the following signature:

要创建一个服务器套接字，您需要使用ServerSocket类提供的四个构造函数之一。您需要指定服务器套接字将监听的IP地址和端口号。通常，IP地址将为127.0.0.1，表示服务器套接字将在本地机器上监听。服务器套接字正在监听的IP地址被称为绑定地址。服务器套接字的另一个重要属性是其backlog，即在服务器套接字开始拒绝传入请求之前，传入连接请求的最大队列长度。ServerSocket类的一个构造函数具有以下签名：

```java
public ServerSocket(int port, int backLog, InetAddress bindingAddress);
```

Notice that for this constructor, the binding address must be an instance of java.net.InetAddress. An easy way to construct an InetAddress object is by calling its static method getByName, passing a String containing the host name, such as in the following code.

请注意，对于这个构造函数，绑定地址必须是java.net.InetAddress的实例。构造一个InetAddress对象的简单方法是调用它的静态方法getByName，传递一个包含主机名的字符串，例如下面的代码。

```java
InetAddress.getByName("127.0.0.1");
```

The following line of code constructs a ServerSocket that listens on port 8080 of the local machine. The ServerSocket has a backlog of 1.

下面的代码行构造了一个在本地机器上监听8080端口的ServerSocket。ServerSocket的等待队列长度为1。

```java
new ServerSocket(8080, 1, InetAddress.getByName("127.0.0.1"));
```

Once you have a ServerSocket instance, you can tell it to wait for an incoming connection request to the binding address at the port the server socket is listening on. You do this by calling the ServerSocket class's accept method. This method will only return when there is a connection request and its return value is an instance of the Socket class. This Socket object can then be used to send and receive byte streams from the client application, as explained in the previous section, "The Socket Class". Practically, the accept method is the only method used in the application accompanying this chapter.

一旦你获得了一个`ServerSocke`t实例，你可以告诉它在绑定地址和端口上等待一个传入的连接请求。你可以通过调用ServerSocket类的accept方法来实现这一点。当有连接请求时，该方法将返回一个Socket类的实例。这个Socket对象可以用来与客户端应用程序进行字节流的发送和接收，如前一节“Socket类”中所述。实际上，在本章附带的应用程序中，`accept`方法是唯一使用的方法。
# The Application

Our web server application is part of the ex01.pyrmont package and consists of three classes:

我们的网页服务器应用程序是ex01.pyrmont包的一部分，由三个类组成：

- HttpServer

- Request

- Response

The entry point of this application (the static main method) can be found in the HttpServer class. The main method creates an instance of HttpServer and calls its await method. The await method, as the name implies, waits for HTTP requests on a designated port, processes them, and sends responses back to the clients. It keeps waiting until a shutdown command is received.The application cannot do more than sending static resources, such as HTML files and image files, residing in a certain directory. It also displays the incoming HTTP request byte streams on the console. However, it does not send any header, such as dates or cookies, to the browser.

该应用程序的入口点（静态main方法）可以在HttpServer类中找到。main方法创建了一个HttpServer实例并调用其await方法。如其名称所示，await方法等待在指定端口上接收HTTP请求，处理它们并向客户端发送响应。它会一直等待，直到接收到关闭命令。该应用程序只能发送静态资源，例如HTML文件和图像文件，这些文件位于特定目录中。它还会在控制台上显示传入的HTTP请求字节流。但是，它不会向浏览器发送任何头部信息，例如日期或cookie。

We will now take a look at the three classes in the following subsections.

接下来，我们将在以下小节中详细介绍这三个类。

## The HttpServer Class

The HttpServer class represents a web server and is presented in Listing 1.1. Note that the await method is given in Listing 1.2 and is not repeated in Listing 1.1 to save space.

HttpServer类表示一个Web服务器，如图1.1所示。请注意，await方法在图1.2中给出，并且在图1.1中没有重复以节省空间。

Listing 1.1: The HttpServer class

清单 1.1： HttpServer 类

```java

package ex01.pyrmont;
import java.net.Socket;
import java.net.ServerSocket;
import java.net.InetAddress;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.IOException;
import java.io.File;
public class HttpServer {
	 /** WEB_ROOT is the directory where our HTML and other files reside.
	 * For this package, WEB_ROOT is the "webroot" directory under the
	 * working directory.
	 * The working directory is the location in the file system
	 * from where the java command was invoked.
	 */
	 public static final String WEB_ROOT =
		 System.getProperty("user.dir") + File.separator + "webroot";
		 // shutdown command
		 private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
		 // the shutdown command received
		 private boolean shutdown = false;
		 public static void main(String[] args) {
			 HttpServer server = new HttpServer();
			 server.await();
		 }
		 public void await() {
		 ...
		 } 
		 }
		
	}
}
```

Listing 1.2: The HttpServer class's await method

清单 1.2：HttpServer 类的 await 方法

```java
public void await() {
	 ServerSocket serverSocket = null;
	 int port = 8080;
	 try {
		 serverSocket = new ServerSocket(port, 1,
		 InetAddress.getByName("127.0.0.1"));
	 }
	 catch (IOException e) {
		 e.printStackTrace();
		 System.exit(1);
	 }
	 // Loop waiting for a request
	 while (!shutdown) {
		 Socket socket = null;
		 InputStream input = null;
		 OutputStream output = null;
		 try {
		 socket = serverSocket.accept();
		 input = socket.getInputStream();
		 output = socket.getOutputStream();
		 // create Request object and parse
		 Request request = new Request(input);
		 request.parse();
		 // create Response object
		 Response response = new Response(output);
		 response.setRequest(request);
		 response.sendStaticResource();
		 // Close the socket
		 socket.close();
		 //check if the previous URI is a shutdown command
		 shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
	 }
	 catch (Exception e) {
	 e.printStackTrace ();
	 continue;
	 }
 } 
```

This web server can serve static resources found in the directory indicated by the public static final WEB_ROOT and all subdirectories under it. WEB_ROOT is initialized as follows:

这个网络服务器可以提供位于公共静态常量WEB_ROOT所指示的目录中以及其所有子目录中找到的静态资源。WEB_ROOT的初始化如下：

```java
public static final String WEB_ROOT = System.getProperty("user.dir") + File.separator + "webroot";
```

The code listings include a directory called webroot that contains some static resources that you can use for testing this application. You can also find several servlets in the same directory for testing applications in the next chapters.

代码清单中包含一个名为webroot的目录，其中包含一些静态资源，您可以用于测试此应用程序。您还可以在同一目录中找到几个Servlet，用于测试下一章节中的应用程序。

To request for a static resource, you type the following URL in your browser's Address or URL box:

要请求静态资源，您需要在浏览器的地址栏或URL框中输入以下URL：

http://machineName:port/staticResource

If you are sending a request from a different machine from the one running your application, machineName is the name or IP address of the computer running this application. If your browser is on the same machine, you can use localhost for the machineName.port is 8080 and staticResource is the name of the file requested and must reside in WEB_ROOT.

如果您正在从与运行应用程序的计算机不同的计算机发送请求，则machineName是运行此应用程序的计算机的名称或IP地址。如果您的浏览器位于同一台计算机上，您可以使用localhost作为machineName。port是8080，staticResource是所请求的文件的名称，必须位于WEB_ROOT中。

For instance, if you are using the same computer to test the application and you want to ask the HttpServer object to send the index.html file, you use the following URL:

例如，如果您正在使用同一台计算机测试应用程序，并且希望请求HttpServer对象发送index.html文件，则可以使用以下URL：

http://localhost:8080/index.html

To stop the server, you send a shutdown command from a web browser by typing the pre-defined string in the browser's Address or URL box, after the host:port section of the URL. The shutdown command is defined by the SHUTDOWN static final variable in the HttpServer class:

停止服务器，您可以通过在浏览器的地址或URL框中输入预定义的字符串来发送关闭命令，该命令位于URL的主机：端口部分之后。关闭命令在HttpServer类中由SHUTDOWN静态常量变量定义：

```
private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
```

Therefore, to stop the server, you use the following URL:

因此，要停止服务器，请使用以下 URL：

http://localhost:8080/SHUTDOWN

Now, let's look at the await method printed in Listing 1.2. The method name await is used instead of wait because wait is an important method in the java.lang.Object class for working with threads. The await method starts by creating an instance of ServerSocket and then going into a while loop.

现在，让我们看看清单 1.2 中打印的 await 方法。之所以使用 await 而不是 wait 这个方法名，是因为 wait 是 java.lang.Object 类中处理线程的一个重要方法。await 方法首先创建一个 ServerSocket 实例，然后进入一个 while 循环。

```java
serverSocket = new ServerSocket(port, 1,

InetAddress.getByName("127.0.0.1"));

...

// Loop waiting for a request

while (!shutdown) {

...

}
```

The code inside the while loop stops at the accept method of ServerSocket, which returns only when an HTTP request is received on port 8080:

while 循环内的代码停止在 ServerSocket 的 accept 方法上，该方法只有在 8080 端口收到 HTTP 请求时才返回：

```java
socket = serverSocket.accept();
```

Upon receiving a request, the await method obtains java.io.InputStream and java.io.OutputStream objects from the Socket instance returned by the accept method.

收到请求后，await 方法会从 accept 方法返回的 Socket 实例中获取 java.io.InputStream 和 java.io.OutputStream 对象。

```java
input = socket.getInputStream();

output = socket.getOutputStream();
```

The await method then creates an ex01.pyrmont.Request object and calls its parse method to parse the HTTP request raw data.

然后，await 方法会创建一个 `ex01.pyrmont.Request` 对象，并调用其解析方法来解析 HTTP 请求的原始数据。

```java
// create Request object and parse

Request request = new Request(input);

request.parse ();
```

Afterwards, the await method creates a Response object, sets the Request object to it, and calls its sendStaticResource method.

之后，await 方法会创建一个 Response 对象，将 Request 对象设置为 Response 对象，并调用其 sendStaticResource 方法。

```java
// create Response object
Response response = new Response(output);

response.setRequest(request);

response.sendStaticResource();
```

Finally, the await method closes the Socket and calls the getUri method of Request to check if the URI of the HTTP request is a shutdown command. If it is, the shutdown variable is set to true and the program exits the while loop.

最后，await 方法关闭 Socket，并调用 Request 的 getUri 方法检查 HTTP 请求的 URI 是否为关闭命令。如果是，则将 shutdown 变量设为 true，然后程序退出 while 循环。

```java
// Close the socket

socket.close ();

//check if the previous URI is a shutdown command

shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
```
## The Request Class

The ex01.pyrmont.Request class represents an HTTP request. An instance of this class is constructed by passing the InputStream object obtained from a Socket that handles the communication with the client. You call one of the read methods of the InputStream object to obtain the HTTP request raw data.

ex01.pyrmont.Request类表示一个HTTP请求。通过从处理与客户端通信的Socket获取的InputStream对象构造此类的实例。您可以调用InputStream对象的其中一个读取方法来获取HTTP请求的原始数据。

The Request class is offered in Listing 1.3. The Request class has two public methods, parse and getUri, which are given in Listings 1.4 and 1.5, respectively.

Request类在代码清单1.3中提供。Request类有两个公共方法parse和getUri，分别在代码清单1.4和1.5中给出。

**Listing 1.3: The Request class**

**代码清单 1.3：请求类**

```java


package ex01.pyrmont;

import java.io.InputStream;

import java.io.IOException;

public class Request {

	private InputStream input;
	
	private String uri;
	
	public Request(InputStream input) {
	
	this.input = input;
	
	}
	
	public void parse() {
	
	...
	
	}
	
	private String parseUri(String requestString) { ...
	
	}
	
	public String getUri() {
	
	return uri;
	
	}
	
}
	
	


```

Listing 1.4: The Request class's parse method

清单1.4：Request类的解析方法

```java

public void parse() {

	// Read a set of characters from the socket
	
	StringBuffer request = new StringBuffer(2048);
	
	int i;
	
	byte[] buffer = new byte[2048];
	
	try {
	
		i = input.read(buffer);
	
	}
	
	catch (IOException e) {
	
		e.printStackTrace();
		
		i = -1;
	
	}
	
	for (int j=0; j<i; j++) {
	
		request.append((char) buffer[j]);
	
	}
	
	System.out.print(request.toString());
	
	uri = parseUri(request.toString());

}
```

Listing 1.5: the Request class's parseUri method

第1.5节：Request类的parseUri方法

```java

private String parseUri(String requestString) {

int index1, index2;

index1 = requestString.indexOf(' ');

if (index1 != -1) {

index2 = requestString.indexOf(' ', index1 + 1);

if (index2 > index1)

return requestString.substring(index1 + 1, index2);

}

return null;

}
```

The parse method parses the raw data in the HTTP request. Not much is done by this method. The only information it makes available is the URI of the HTTP request that it obtains by calling the private method parseUri. The parseUri method stores the URI in the uri variable. The public getUri method is invoked to return the URI of the HTTP request.

解析方法（parse method）用于解析HTTP请求中的原始数据。该方法并没有做太多的工作。它所提供的唯一信息是通过调用私有方法parseUri来获取HTTP请求的URI。parseUri方法将URI存储在uri变量中。公共的getUri方法被调用以返回HTTP请求的URI。

> Note More processing of the HTTP request raw data will be done in the applications accompanying Chapter 3 and the subsequent chapters.
> 
> 请注意，在第三章及其后续章节的应用程序中，将对HTTP请求原始数据进行更多的处理。

To understand how the parse and parseUri methods work, you need to know the structure of an HTTP request, discussed in the previous section, "The Hypertext Transfer Protocol (HTTP)". In this chapter, we are only interested in the first part of the HTTP request, the request line. A request line begins with a method token, followed by the request URI and the protocol version, and ends with carriage-return linefeed (CRLF) characters. Elements in a request line are separated by a space character. For instance, the request line for a request for the index.html file using the GET method is as follows.

要理解parse和parseUri方法的工作原理，您需要了解HTTP请求的结构，该结构在前一节“超文本传输协议（HTTP）”中已讨论。在本章中，我们只关注HTTP请求的第一部分，即请求行。请求行以方法标记开头，后跟请求URI和协议版本，并以回车换行（CRLF）字符结尾。请求行中的元素由空格字符分隔。例如，使用GET方法请求index.html文件的请求行如下。

```
GET /index.html HTTP/1.1
```

The parse method reads the whole byte stream from the socket's InputStream that is  passed to the Request object and stores the byte array in a buffer. It then populates a StringBuffer object called request using the bytes in the buffer byte array, and passes the String representation of the StringBuffer to the parseUri method

parse方法从传递给Request对象的socket的InputStream中读取整个字节流，并将字节数组存储在缓冲区中。然后，它使用缓冲区字节数组中的字节填充一个名为request的StringBuffer对象，并将StringBuffer的字符串表示形式传递给parseUri方法。

The parse method is given in Listing 1.4. The parseUri method then obtains the URI from the request line. Listing 1.5 presents the parseUri method. The parseUri method searches for the first and the second spaces in the request and obtains the URI from it.

parse方法在代码清单1.4中给出。然后，parseUri方法从请求行中获取URI。代码清单1.5展示了parseUri方法。parseUri方法在请求中搜索第一个和第二个空格，并从中获取URI。
## The Response Class（响应类）

The ex01.pyrmont.Response class represents an HTTP response and is given in Listing 1.6.

ex01.pyrmont.Response 类表示 HTTP 响应，见清单 1.6。

Listing 1.6: The Response class

清单 1.6：响应类

```java
package ex01.pyrmont;
import java.io.OutputStream;
import java.io.IOException;
import java.io.FileInputStream;
import java.io.File;
/*
 HTTP Response = Status-Line
 *(( general-header | response-header | entity-header ) CRLF)
 CRLF
 [ message-body ]
 Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
*/
public class Response {
	 private static final int BUFFER_SIZE = 1024;
	 Request request;
	 OutputStream output;
	 public Response(OutputStream output) {
		 this.output = output;
	 }
	 public void setRequest(Request request) {
		 this.request = request;
	 }
	 public void sendStaticResource() throws IOException {
		 byte[] bytes = new byte[BUFFER_SIZE];
		 FileInputStream fis = null;
		 try {
			 File file = new File(HttpServer.WEB_ROOT, request.getUri());
			 if (file.exists()) {
				 fis = new FileInputStream(file);
				 int ch = fis.read(bytes, 0, BUFFER_SIZE);
				 while (ch!=-1) {
				 output.write(bytes, 0, ch);
				 ch = fis.read(bytes, 0, BUFFER_SIZE);
			 }
		 }
		 else {
			 // file not found
			 String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
				 "Content-Type: text/html\r\n" +
				 "Content-Length: 23\r\n" +
				 "\r\n" +
				 "<h1>File Not Found</h1>";
				 output.write(errorMessage.getBytes());
		 }
		 }
		 catch (Exception e) {
			 // thrown if cannot instantiate a File object
			 System.out.println(e.toString() );
		 }
		 finally {
			 if (fis!=null)
				 fis.close();
			 }
		 } 
 }
```

First note that its constructor accepts a java.io.OutputStream object, such as the following.

首先注意，它的构造函数接受一个java.io.OutputStream对象，例如以下内容。

```java
public Response(OutputStream output) {

	this.output = output;

}
```

A Response object is constructed by the HttpServer class's await method by passing the OutputStream object obtained from the socket.

一个响应对象是通过HttpServer类的await方法构造的，该方法通过从套接字获取的OutputStream对象进行传递。

The Response class has two public methods: setRequest and sendStaticResource method. The setRequest method is used to pass a Request object to the Response object.

Response类有两个公共方法：setRequest和sendStaticResource方法。setRequest方法用于将Request对象传递给Response对象。

The sendStaticResource method is used to send a static resource, such as an HTML file. It first instantiates the java.io.File class by passing the parent path and child path to the File class's constructor.

sendStaticResource方法用于发送静态资源，例如HTML文件。它首先通过将父路径和子路径传递给File类的构造函数来实例化java.io.File类。

```java
File file = new File(HttpServer.WEB_ROOT, request.getUri());
```

It then checks if the file exists. If it does, sendStaticResource constructs a java.io.FileInputStream object by passing the File object. Then, it invokes the read method of the FileInputStream and writes the byte array to the OutputStream output. Note that in this case the content of the static resource is sent to the browser as raw data.

然后它检查文件是否存在。如果存在，sendStaticResource通过传递File对象构造了一个java.io.FileInputStream对象。然后，它调用FileInputStream的read方法，并将字节数组写入OutputStream output。请注意，在这种情况下，静态资源的内容以原始数据的形式发送到浏览器。

```java
if (file.exists()) {

	fis = new FileInputstream(file);
	
	int ch = fis.read(bytes, 0, BUFFER_SIZE);
	
	while (ch!=-1) {
	
		output.write(bytes, 0, ch);
		
		ch = fis.read(bytes, 0, BUFFER_SIZE);
	
	}

}
```

If the file does not exist, the sendStaticResource method sends an error message to the browser.

如果文件不存在，sendStaticResource 方法会向浏览器发送错误信息。

```java
String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +

"Content-Type: text/html\r\n" +

"Content-Length: 23\r\n" +

"\r\n" +

"<h1>File Not Found</h1>";

output.write(errorMessage.getBytes());
```

# Running the Application（运行应用程序）

To run the application, from the working directory, type the following:

要运行应用程序，请在工作目录中键入以下内容：

```java
java ex01.pyrmont.HttpServer
```

To test the application, open your browser and type the following in the URL or Address box:

要测试应用程序，请打开浏览器并在 URL 或地址框中键入以下内容：

http://localhost:8080/index.html

You will see the index.html page displayed in your browser, as in Figure 1.1.

您将在浏览器中看到 index.html 页面，如图 1.1 所示。

Figure 1.1: The output from the web server

图 1.1： 网络服务器的输出

![[Pasted image 20231206214720.png]]

On the console, you can see the HTTP request similar to the following:

在控制台中，您可以看到类似下面的 HTTP 请求：

```html
GET /index.html HTTP/1.1

Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg,

application/vnd.ms-excel, application/msword, application/vnd.ms

powerpoint, application/x-shockwave-flash, application/pdf, */*

Accept-Language: en-us

Accept-Encoding: gzip, deflate

User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR

1.1.4322)

Host: localhost:8080

Connection: Keep-Alive

GET /images/logo.gif HTTP/1.1

Accept: */*

Referer: http://localhost:8080/index.html

Accept-Language: en-us

Accept-Encoding: gzip, deflate

User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR

1.1.4322)

Host: localhost:8080

Connection: Keep-Alive
```

# Summary（摘要）

In this chapter you have seen how a simple web server works. The application accompanying this chapter consists of only three classes and is not fully functional. Nevertheless, it serves as a good learning tool. The next chapter will discuss the processing of dynamic contents.

在本章中，你将看到一个简单的网络服务器是如何工作的。本章附带的应用程序只有三个类，功能并不完整。不过，它是一个很好的学习工具。下一章将讨论动态内容的处理。