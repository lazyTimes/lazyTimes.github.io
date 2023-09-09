---
title: 【Java】Exploring the New HTTP Client in Java
subtitle: 新HTTPClient API介绍
author: lazytime
url_suffix: httpclientjdk11
tags:
  - Java
categories:
  -  Java
keywords: httpclient,java
description: 新HTTPClient API介绍
copyright: true
date: 2023-08-10 16:49:34
---

# 探索 Java 中的新 HTTP 客户端
# 原文

[https://www.baeldung.com/java-9-http-client](https://www.baeldung.com/java-9-http-client)

## [**1. Overview**](https://www.baeldung.com/java-9-http-client#introduction)

In this tutorial, we'll explore Java 11's standardization of **HTTP client API that implements HTTP/2 and Web Socket.**

本文讲讨论Java 11 的新HTTP客户端API是如何实现 HTTP/2 和 WebSocket的。

It aims to replace the legacy _[HttpUrlConnection](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/net/HttpURLConnection.html)_ class that has been present in the JDK since the very early years of Java.

它旨在取代自 Java 诞生之初就存在于 JDK 中的传统_[HttpUrlConnection](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/net/HttpURLConnection.html)_ 类。

<!-- more -->

> It aims to .... 旨在

Until very recently, Java provided only the _HttpURLConnection_ API, which is low-level and isn't known for being feature-rich and user-friendly.

在旧版本中，Java 提供 _HttpURLConnection_ API，该 API 是低级的，并不以功能丰富和用户友好而著称。

Therefore, some widely used third-party libraries were commonly used, such as [Apache HttpClient](https://hc.apache.org/httpcomponents-client-ga/), [Jetty](https://eclipse.dev/jetty/documentation/jetty-9/index.html#http-client-api) and Spring's [RestTemplate](https://www.baeldung.com/rest-template).

所以，我们通常都会使用 类似   [Apache HttpClient](https://hc.apache.org/httpcomponents-client-ga/), [Jetty](https://eclipse.dev/jetty/documentation/jetty-9/index.html#http-client-api) 或者 Spring's [RestTemplate](https://www.baeldung.com/rest-template) 这样的第三方库作为替代。

## Further reading:（相关阅读）

## [Posting with Java HttpClient](https://www.baeldung.com/java-httpclient-post)

From Java 9 onwards, the new <em>HttpClient</em> API provides both a synchronous and asynchronous modern web client. We look at how it can be used to make requests.

从 Java 9 开始，新的 <em>HttpClient</em> API 提供了**同步**和**异步**的现代 Web 客户端。我们来看看如何使用它来发出请求。

[Read more](https://www.baeldung.com/java-httpclient-post) →

## [Java HttpClient With SSL](https://www.baeldung.com/java-httpclient-ssl)

Learn how to use the Java HttpClient to connect to HTTPS URLs and also find out how to bypass certificate verification in non-production environments.

了解如何使用 Java HttpClient 连接 HTTPS URL，以及如何在非生产环境中绕过证书验证。

[Read more](https://www.baeldung.com/java-httpclient-ssl) →

## [Adding Parameters to Java HttpClient Requests](https://www.baeldung.com/java-httpclient-request-parameters)

Different examples of HTTPClient core Java.

HTTPClient core Java 的不同示例。

[Read more](https://www.baeldung.com/java-httpclient-request-parameters) →

## [**2. Background**](https://www.baeldung.com/java-9-http-client#introduction-1)

The change was implemented as a part of JEP 321.

所有的改变均实现自**JEP 321**。 
### [2.1. Major Changes as Part of JEP 321](https://www.baeldung.com/java-9-http-client#1-major-changes-as-part-of-jep-321)

1. The incubated HTTP API from Java 9 is now officially incorporated into the Java SE API. The new [HTTP APIs](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/package-summary.html) can be found in **java.net.HTTP.***

Java 9 中孵化的 HTTP API 现已正式纳入 Java SE API。新的 [HTTP APIs](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/package-summary.html) 可在 **java.net.HTTP.** 中找到。

2. The newer version of the HTTP protocol is designed to improve the overall performance of sending requests by a client and receiving responses from the server. This is achieved by introducing a number of changes such as stream multiplexing, header compression and push promises.

较新版本的 HTTP 协议旨在提高客户端发送请求和服务器接收响应的整体性能。这是通过引入流**多路复用**、报头压缩和推送承诺来实现的。

3. As of Java 11, **the API is now fully asynchronous (the previous HTTP/1.1 implementation was blocking).** Asynchronous calls are implemented using _CompletableFuture_.The _CompletableFuture_ implementation takes care of applying each stage once the previous one has finished, so this whole flow is asynchronous.

从 Java 11 开始，**应用程序接口现在是完全异步的（以前的 HTTP/1.1 实现是阻塞的）。** 异步调用是使用 _CompletableFuture_ 实现的。

4. The new HTTP client API provides a standard way to perform HTTP network operations with support for modern Web features such as HTTP/2, without the need to add third-party dependencies.

新的 HTTP 客户端 API 提供了执行 HTTP 网络操作的标准方法，支持 HTTP/2 等现代网络功能，无需添加第三方依赖性。

5. The new APIs provide native support for HTTP 1.1/2 WebSocket. The core classes and interface providing the core functionality include:

新的应用程序接口为 `HTTP 1.1/2` WebSocket 提供本地支持。提供核心功能的核心类和接口包括

- The _HttpClient_ class, _java.net.http.HttpClient_
- The _HttpRequest_ class, _java.net.http.HttpRequest_
- The _HttpResponse_< T > interface, _java.net.http.HttpResponse_
- The _WebSocket_ interface, _java.net.http.WebSocket_

- _HttpClient_ 类， _java.net.http.HttpClient_ 。
- _HttpRequest_ 类，_java.net.http.HttpRequest_
- 接口_HttpResponse_< T >,  _java.net.http.HttpResponse_
- _WebSocket_ 接口，_java.net.http.WebSocket_ < T >。
### [**2.2. Problems With the Pre-Java 11 HTTP Client**](https://www.baeldung.com/java-9-http-client#2-problems-with-the-pre-java-11-http-client)

The existing _HttpURLConnection_ API and its implementation had numerous problems:

现有的 _HttpURLConnection_ API 及其实现存在许多问题：

- URLConnection API was designed with multiple protocols that are now no longer functioning (FTP, gopher, etc.).
- The API predates HTTP/1.1 and is too abstract.
- It works in blocking mode only (i.e., one thread per request/response).
- It is very hard to maintain.

- URLConnection API 在设计时使用了多个现已失效的协议（FTP、gopher 等）。
- 该 API 早于 HTTP/1.1，过于抽象。
- 只能在阻塞模式下工作（即每个请求/响应只有一个线程）。
- 很难维护。
## [**3. HTTP Client API Overview**](https://www.baeldung.com/java-9-http-client#api)

Unlike _HttpURLConnection_, HTTP Client provides synchronous and asynchronous request mechanisms.

与 _HttpURLConnection_ 不同，HTTP 客户端提供同步和异步请求机制。

The API consists of three core classes:

API 由三个核心类组成：

- _HttpRequest_ represents the request to be sent via the _HttpClient_.
- _HttpClient_ behaves as a container for configuration information common to multiple requests.
- _HttpResponse_ represents the result of an _HttpRequest_ call.

- _HttpRequest_ 表示要通过 _HttpClient_ 发送的请求。
- _HttpClient_ 是多个请求所共有的配置信息的容器。
- _HttpResponse_ 表示 _HttpRequest_ 调用的结果。

We'll examine each of them in more details in the following sections. First, let's focus on a request.

我们将在下面的章节中对它们逐一进行详细介绍。首先，我们来关注一个请求。
## [**4. _HttpRequest_**](https://www.baeldung.com/java-9-http-client#requests)

_HttpRequest_ is an object that represents the request we want to send. New instances can be created using _HttpRequest.Builder._

_HttpRequest_ 是一个对象，代表我们要发送的请求。可以使用 _HttpRequest.Builder._ 创建新实例。

We can get it by calling _HttpRequest.newBuilder()_. _Builder_ class provides a bunch of methods that we can use to configure our request.

我们可以通过调用 _HttpRequest.newBuilder()_ 来获取它。 _Builder_ 类提供了许多方法，我们可以用它们来配置我们的请求。

We'll cover the most important ones.

我们将介绍最重要的几项。

Note: In JDK 16, there is a new _HttpRequest.newBuilder(HttpRequest request, BiPredicate<String,​String> filter)_ method, which creates a _Builder_ whose initial state is copied from an existing _HttpRequest_.

> 注意：在JDK16， 有一个新的 _HttpRequest.newBuilder(HttpRequest request, BiPredicate<String,String> filter)_  方法，用于创建一个_Builder_，其初始状态是从现有的_HttpRequest_复制而来。

This builder can be used to build an _HttpRequest_, equivalent to the original, while allowing amendment of the request state prior to construction, for example, removing headers:

该构建器可用于构建一个 _HttpRequest_，等同于原始请求，同时允许在构建之前修改请求状态，例如删除头信息：

```coffeescript
HttpRequest.newBuilder(request, (name, value) -> !name.equalsIgnoreCase("Foo-Bar"))
```
### [**4.1. Setting _URI_**](https://www.baeldung.com/java-9-http-client#1-setting-uri)

The first thing we have to do when creating a request is to provide the URL.

创建请求时，我们要做的第一件事就是提供 URL。

We can do that in two ways — using the constructor for _Builder_ with _URI_ parameter or calling method _uri(URI)_ on the _Builder_ instance:

我们可以通过两种方法实现这一目的：使用 _URI_ 参数的 _Builder_ 构造函数，或者调用 _Builder_ 实例上的 _uri(URI)_ 方法：

```java
HttpRequest.newBuilder(new URI("https://postman-echo.com/get"))
 
HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
```

The last thing we have to configure to create a basic request is an HTTP method.

创建基本请求的最后一项配置是 HTTP 方法。
### [**4.2. Specifying the HTTP Method**](https://www.baeldung.com/java-9-http-client#2-specifying-the-http-method)

We can define the HTTP method that our request will use by calling one of the methods from _Builder_:

我们可以通过调用 _Builder_ 中的一个方法来定义请求将使用的 HTTP 方法：

- _GET()_
- _POST(BodyPublisher body)_
- _PUT(BodyPublisher body)_
- _DELETE()_

We'll cover _BodyPublisher_ in detail, later.

稍后我们将详细介绍 _BodyPublisher_ 的内容。

Now let's just create **a very simple GET request example**:

现在，让我们创建**个非常简单的 GET 请求示例**：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .GET()
  .build();
```

This request has all parameters required by _HttpClient_.

该请求包含 _HttpClient_ 要求的所有参数。

However, we sometimes need to add additional parameters to our request. Here are some important ones:

不过，有时我们需要在请求中添加其他参数。下面是一些重要的参数：

- The version of the HTTP protocol
- Headers
- A timeout

- HTTP 协议的版本
- 标题
- 超时
### [**4.3. Setting HTTP Protocol Version**](https://www.baeldung.com/java-9-http-client#3-setting-http-protocol-version)

The API fully leverages the HTTP/2 protocol and uses it by default, but we can define which version of the protocol we want to use:

该应用程序接口完全利用 HTTP/2 协议，并默认使用该协议，但我们可以定义要使用的协议版本：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .version(HttpClient.Version.HTTP_2)
  .GET()
  .build();
```

**Important to mention here is that the client will fall back to, e.g., HTTP/1.1 if HTTP/2 isn't supported.**

**这里需要指出的是，如果不支持 HTTP/2，客户端将退回到 HTTP/1.1 等协议。**
### [**4.4. Setting Headers**](https://www.baeldung.com/java-9-http-client#4-setting-headers)

In case we want to add additional headers to our request, we can use the provided builder methods.

如果我们想在请求中添加其他标头，可以使用提供的构建器方法。

We can do that by either passing all headers as key-value pairs to the _headers()_ method or by using _header()_ method for the single key-value header:

为此，我们可以将所有标头作为键值对传递给 _headers()_ 方法，或者使用 _header()_ 方法来处理单个键值标头：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .headers("key1", "value1", "key2", "value2")
  .GET()
  .build();

HttpRequest request2 = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .header("key1", "value1")
  .header("key2", "value2")
  .GET()
  .build();
```

The last useful method we can use to customize our request is a _timeout()_.

我们可以用来定制请求的最后一个有用方法是 _timeout()_ 。

### [**4.5. Setting a Timeout**](https://www.baeldung.com/java-9-http-client#5-setting-a-timeout)

Let's now define the amount of time we want to wait for a response.

现在我们来定义等待响应的时间。

If the set time expires, a _HttpTimeoutException_ will be thrown. The default timeout is set to infinity.

如果设定的时间已过，就会抛出一个 _HttpTimeoutException_ 异常。默认超时设置为无穷大。

The timeout can be set with the _Duration_ object by calling method _timeout()_ on the builder instance:

可以通过调用构建器实例上的 _timeout()_ 方法，使用 _Duration_ 对象设置超时时间：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .timeout(Duration.of(10, SECONDS))
  .GET()
  .build();
```

## [**5. Setting a Request Body**](https://www.baeldung.com/java-9-http-client#setting-a-request-body)

We can add a body to a request by using the request builder methods: _POST(BodyPublisher body)_, _PUT(BodyPublisher body)_ and _DELETE()_.

我们可以使用请求生成器方法为请求添加正文： _POST(BodyPublisher body)_、_PUT(BodyPublisher body)_  和 _DELETE()_。

The new API provides a number of _BodyPublisher_ implementations out-of-the-box that simplify passing the request body:

新的 API 提供了许多开箱即用的 _BodyPublisher_ 实现，简化了请求正文的传递：

- _StringProcessor_ – reads body from a _String_, created with _HttpRequest.BodyPublishers.ofString_
- _InputStreamProcessor_ – reads body from an _InputStream_, created with _HttpRequest.BodyPublishers.ofInputStream_
- _ByteArrayProcessor_ – reads body from a byte array, created with _HttpRequest.BodyPublishers.ofByteArray_
- _FileProcessor_ – reads body from a file at the given path, created with _HttpRequest.BodyPublishers.ofFile_

- _StringProcessor_ - 从 _String_ 中读取正文，使用 _HttpRequest.BodyPublishers.ofString_ 创建。
- _InputStreamProcessor_ - 从 _InputStream_ 中读取正文，使用 _HttpRequest.BodyPublishers.ofInputStream_ 创建。
- _ByteArrayProcessor_ - 从字节数组中读取正文，使用 _HttpRequest.BodyPublishers.ofByteArray_ 创建。
- _FileProcessor_ - 从指定路径的文件中读取正文，使用 _HttpRequest.BodyPublishers.ofFile_ 创建。

In case we don't need a body, we can simply pass in an _HttpRequest.BodyPublishers.__noBody()_:

如果不需要正文，我们只需传入 _HttpRequest.BodyPublishers. \_\_noBody()_  即可：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .POST(HttpRequest.BodyPublishers.noBody())
  .build();
```

Note: In JDK 16, there's a new _HttpRequest.BodyPublishers.concat(BodyPublisher…)_ method that helps us building a request body from the concatenation of the request bodies published by a sequence of publishers. The request body published by a _concatenation publisher_ is logically equivalent to the request body that would have been published by concatenating all the bytes of each publisher in sequence.

> 注：在 JDK 16 中，有一个新的 _HttpRequest.BodyPublishers.concat(BodyPublisher...)_ 方法，可以帮助我们通过串联一系列发布者发布的请求体来构建请求体。由 _concatenation 发布者_ 发布的请求正文在逻辑上等同于按顺序连接每个发布者的所有字节后发布的请求正文。
### [**5.1. _StringBodyPublisher_**](https://www.baeldung.com/java-9-http-client#1-stringbodypublisher)

Setting a request body with any _BodyPublishers_ implementation is very simple and intuitive.

使用任何 _BodyPublishers_ 实现来设置请求正文都非常简单直观。

For example, if we want to pass a simple _String_ as a body, we can use _StringBodyPublishers_.

例如，如果我们想传递一个简单的 _String_ 作为正文，我们可以使用 _StringBodyPublishers_。

As we already mentioned, this object can be created with a factory method _ofString()_ — it takes just a _String_ object as an argument and creates a body from it:

正如我们已经提到的，可以使用工厂方法 _ofString()_ 创建该对象 -- 该方法只接受一个 _String_ 对象作为参数，并从中创建一个正文：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;charset=UTF-8")
  .POST(HttpRequest.BodyPublishers.ofString("Sample request body"))
  .build();
```
### [**5.2. _InputStreamBodyPublisher_**](https://www.baeldung.com/java-9-http-client#2-inputstreambodypublisher)

To do that, the _InputStream_ has to be passed as a _Supplier_ (to make its creation lazy), so it's a little bit different than _StringBodyPublishers_.

要做到这一点，必须将 _InputStream_ 作为 _Supplier_ 传递（变为懒加载），因此它与 _StringBodyPublishers_ 有点不同。

However, this is also quite straightforward:

不过，这也很简单明了：

```java
byte[] sampleData = "Sample request body".getBytes();
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;charset=UTF-8")
  .POST(HttpRequest.BodyPublishers
   .ofInputStream(() -> new ByteArrayInputStream(sampleData)))
  .build();
```

Notice how we used a simple _ByteArrayInputStream_ here. Of course, that can be any _InputStream_ implementation.

请注意我们在这里使用了一个简单的 _ByteArrayInputStream_ 。当然，这可以是任何 _InputStream_ 的实现。
### [**5.3. _ByteArrayProcessor_**](https://www.baeldung.com/java-9-http-client#3-bytearrayprocessor)

We can also use _ByteArrayProcessor_ and pass an array of bytes as the parameter:

我们还可以使用 _ByteArrayProcessor_ 并将字节数组作为参数传递：

```java
byte[] sampleData = "Sample request body".getBytes();
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;charset=UTF-8")
  .POST(HttpRequest.BodyPublishers.ofByteArray(sampleData))
  .build();
```

### [**5.4. _FileProcessor_**](https://www.baeldung.com/java-9-http-client#4-fileprocessor)

To work with a File, we can make use of the provided _FileProcessor_.、

要处理文件，我们可以使用所提供的 _FileProcessor_。

Its factory method takes a path to the file as a parameter and creates a body from the content:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;charset=UTF-8")
  .POST(HttpRequest.BodyPublishers.fromFile(
    Paths.get("src/test/resources/sample.txt")))
  .build();
```

We've covered how to create _HttpRequest_ and how to set additional parameters in it.

我们已经介绍了如何创建 _HttpRequest_ 以及如何在其中设置附加参数。

Now it's time to take a deeper look at _HttpClient_ class, which is responsible for sending requests and receiving responses.

现在是深入了解 _HttpClient_ 类的时候了，它负责发送请求和接收响应。

## [**6. _HttpClient_**](https://www.baeldung.com/java-9-http-client#requests-1)

All requests are sent using _HttpClient_, which can be instantiated using the _HttpClient.newBuilder()_ method or by calling _HttpClient.newHttpClient()_.

所有请求都是通过 _HttpClient_ 发送的，可以使用 _HttpClient.newBuilder()_ 方法或调用 _HttpClient.newHttpClient()_ 来实例化 _HttpClient。

It provides a lot of useful and self-describing methods we can use to handle our request/response.

它提供了许多有用的自描述方法，我们可以用它们来处理请求/响应。

Let's cover some of these here.

下面我们就来介绍其中的一些方法。
### [**6.1. Handling Response Body**](https://www.baeldung.com/java-9-http-client#1-handling-response-body)

Similar to the fluent methods for creating publishers, there are methods dedicated to creating handlers for common body types:

与创建发布器的流畅方法类似，也有一些方法专门用于为常见的主体类型创建处理程序：

```java
BodyHandlers.ofByteArray
BodyHandlers.ofString
BodyHandlers.ofFile
BodyHandlers.discarding
BodyHandlers.replacing
BodyHandlers.ofLines
BodyHandlers.fromLineSubscriber
```

Pay attention to the usage of the new _BodyHandlers_ factory class.

请注意新的 _BodyHandlers_ 工厂类的用法。

Before Java 11, we had to do something like this:

在 Java 11 之前，我们不得不这样做：

```java
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandler.asString());
```

And we can now simplify it:

现在我们可以将其简化：

```java
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
```

### [**6.2. Setting a Proxy**](https://www.baeldung.com/java-9-http-client#2-setting-a-proxy)

We can define a proxy for the connection by just calling _proxy()_ method on a _Builder_ instance:

我们只需在 _Builder_ 实例上调用 _proxy()_ 方法，就能为连接定义一个代理：

```java
HttpResponse<String> response = HttpClient
  .newBuilder()
  .proxy(ProxySelector.getDefault())
  .build()
  .send(request, BodyHandlers.ofString());
```

In our example, we used the default system proxy.

在我们的示例中，我们使用了默认的系统代理。
### [**6.3. Setting the Redirect Policy**](https://www.baeldung.com/java-9-http-client#3-setting-the-redirect-policy)

Sometimes the page we want to access has moved to a different address.

有时，我们想要访问的页面已经转移到了不同的地址。

In that case, we'll receive HTTP status code 3xx, usually with the information about new URI. **_HttpClient_ can redirect the request to the new URI automatically if we set the appropriate redirect policy.**

在这种情况下，我们会收到 HTTP 状态代码 **3xx**，其中通常包含有关新 URI 的信息。 **如果我们设置了适当的重定向策略，**_HttpClient_ 就能自动将请求重定向到新的 URI。

We can do it with the _followRedirects()_ method on _Builder_:

我们可以通过 _Builder_ 上的 _followRedirects()_ 方法来实现：

```java
HttpResponse<String> response = HttpClient.newBuilder()
  .followRedirects(HttpClient.Redirect.ALWAYS)
  .build()
  .send(request, BodyHandlers.ofString());
```

All policies are defined and described in enum _HttpClient.Redirect_.

所有策略都在枚举 _HttpClient.Redirect_ 中定义和描述。
### [**6.4. Setting _Authenticator_ for a Connection**](https://www.baeldung.com/java-9-http-client#4-setting-authenticator-for-a-connection)

An _Authenticator_ is an object that negotiates credentials (HTTP authentication) for a connection.

验证器是一个为连接协商凭证（HTTP 验证）的对象。

It provides different authentication schemes (such as basic or digest authentication).

它提供不同的验证方案（如基本验证或摘要验证）。

In most cases, authentication requires username and password to connect to a server.

在大多数情况下，身份验证需要用户名和密码才能连接服务器。

We can use _PasswordAuthentication_ class, which is just a holder of these values:

我们可以使用 _PasswordAuthentication_ 类，它只是这些值的持有者：

```java
HttpResponse<String> response = HttpClient.newBuilder()
  .authenticator(new Authenticator() {
    @Override
    protected PasswordAuthentication getPasswordAuthentication() {
      return new PasswordAuthentication(
        "username", 
        "password".toCharArray());
    }
}).build()
  .send(request, BodyHandlers.ofString());
```

Here we passed the username and password values as a plaintext. Of course, this would have to be different in a production scenario.

在这里，我们以明文形式传递用户名和密码值。当然，这在生产场景中必须有所不同。

Note that not every request should use the same username and password. The _Authenticator_ class provides a number of _getXXX_ (e.g., _getRequestingSite()_) methods that can be used to find out what values should be provided.

> 请注意，并非每个请求都应使用相同的用户名和密码。_Authenticator_ 类提供了许多 _getXXX_（例如 _getRequestingSite()_）方法，可用于查找应提供哪些值。

Now we're going to explore one of the most useful features of new _HttpClient_ — asynchronous calls to the server.

现在，我们将探索新_HttpClient_最有用的功能之一--**对服务器的异步调用**。

### [**6.5. Send Requests – Sync vs Async**](https://www.baeldung.com/java-9-http-client#5-send-requests---sync-vs-async)

New _HttpClient_ provides two possibilities for sending a request to a server:

新的 _HttpClient_ 提供了两种向服务器发送请求的可能性：

- **_send(…)_ – synchronously** (blocks until the response comes)
- **_sendAsync(…)_ – asynchronously** (doesn't wait for the response, non-blocking)

- **_send(...)_-同步**（阻塞直到响应到来）
- **_sendAsync(...)_-异步**（不等待响应，非阻塞）。

Up until now, the _send(._..) method naturally waits for a response:

到目前为止，_send(._..) 方法一直在等待响应：

```java
HttpResponse<String> response = HttpClient.newBuilder()
  .build()
  .send(request, BodyHandlers.ofString());
```

This call returns an _HttpResponse_ object, and we're sure that the next instruction from our application flow will be run only when the response is already here.

该调用会返回一个 _HttpResponse_ 对象，我们可以确信，只有当响应已经存在时，应用流程的下一条指令才会运行。

However, it has a lot of drawbacks especially when we are processing large amounts of data.

不过，这种方法有很多缺点，尤其是在处理大量数据时。

So, now we can use _sendAsync(._..) method — which returns  _CompletableFeature< HttpResponse>_  — **to process a request asynchronously**:

因此，现在我们可以使用_sendAsync(._..)方法（该方法返回_CompletableFeature< HttpResponse>_）**异步处理请求**：

```java
CompletableFuture<HttpResponse<String>> response = HttpClient.newBuilder()
  .build()
  .sendAsync(request, HttpResponse.BodyHandlers.ofString());
```

The new API can also deal with multiple responses, and stream the request and response bodies:

新的应用程序接口还可以处理多个响应，并对请求和响应体进行流式处理：

```java
List<URI> targets = Arrays.asList(
  new URI("https://postman-echo.com/get?foo1=bar1"),
  new URI("https://postman-echo.com/get?foo2=bar2"));
HttpClient client = HttpClient.newHttpClient();
List<CompletableFuture<String>> futures = targets.stream()
  .map(target -> client
    .sendAsync(
      HttpRequest.newBuilder(target).GET().build(),
      HttpResponse.BodyHandlers.ofString())
    .thenApply(response -> response.body()))
  .collect(Collectors.toList());
```

### [**6.6. Setting _Executor_ for Asynchronous Calls**](https://www.baeldung.com/java-9-http-client#6-setting-executor-for-asynchronous-calls)

We can also define an _Executor_ that provides threads to be used by asynchronous calls.

我们还可以定义一个 _Executor_ 来提供线程供异步调用使用。

This way we can, for example, limit the number of threads used for processing requests:

例如，这样我们就可以限制用于处理请求的线程数量：

```java
ExecutorService executorService = Executors.newFixedThreadPool(2);

CompletableFuture<HttpResponse<String>> response1 = HttpClient.newBuilder()
  .executor(executorService)
  .build()
  .sendAsync(request, HttpResponse.BodyHandlers.ofString());

CompletableFuture<HttpResponse<String>> response2 = HttpClient.newBuilder()
  .executor(executorService)
  .build()
  .sendAsync(request, HttpResponse.BodyHandlers.ofString());
```

By default, the _HttpClient_ uses executor _java.util.concurrent.Executors.newCachedThreadPool()_.

默认情况下，_HttpClient_ 使用执行器 _java.util.concurrent.Executors.newCachedThreadPool()_。
### [**6.7. Defining a _CookieHandler_**](https://www.baeldung.com/java-9-http-client#7-defining-a-cookiehandler)

With new API and builder, it's straightforward to set a _CookieHandler_ for our connection. We can use builder method _cookieHandler(CookieHandler cookieHandler)_ to define client-specific _CookieHandler_.

有了新的 API 和构建器，为连接设置 _CookieHandler_ 就变得简单易行了。

我们可以使用构建器方法 _cookieHandler(CookieHandler cookieHandler)_ 来定义客户端特定的 _CookieHandler_。

Let's define _CookieManager (_a concrete implementation of _CookieHandler_ that separates the storage of cookies from the policy surrounding accepting and rejecting cookies) that doesn't allow to accept cookies at all:

让我们定义  _CookieManager_（ _CookieHandler_ 的具体实现，它将 Cookie 的存储与接受和拒绝 Cookie 的策略分离开来），它完全不接受 Cookie：

```java
HttpClient.newBuilder()
  .cookieHandler(new CookieManager(null, CookiePolicy.ACCEPT_NONE))
  .build();
```

In case our _CookieManager_ allows cookies to be stored, we can access them by checking _CookieHandler_ from our _HttpClient_:

如果 _CookieManager_ 允许存储 cookie，我们就可以通过检查 _HttpClient_ 中的 _CookieHandler_ 来访问它们：

```java
((CookieManager) httpClient.cookieHandler().get()).getCookieStore()
```

Now let's focus on the last class from Http API — the _HttpResponse_.

现在，让我们来关注 Http API 的最后一个类--_HttpResponse_。
## [**7. _HttpResponse_ Object**](https://www.baeldung.com/java-9-http-client#requests-2)

The _HttpResponse_ class represents the response from the server. It provides a number of useful methods, but these are the two most important:

_HttpResponse_ 类表示来自服务器的响应。它提供了许多有用的方法，但其中最重要的有两个：

- _statusCode()_ returns status code (type _int_) for a response (_HttpURLConnection_ class contains possible values).
- _body()_ returns a body for a response (return type depends on the response _BodyHandler_ parameter passed to the _send()_ method).

- _statusCode()_ 返回响应的状态代码（注意类型 _int_）（_HttpURLConnection_ 类包含可能的值）。
- _body()_ 返回响应的正文（返回类型取决于传递给 _send()_ 方法的响应 _BodyHandler_ 参数）。

The response object has other useful methods that we'll cover such as _uri()_, _headers()_, _trailers()_ and _version()_.

响应对象还有其他有用的方法，如 _uri()_、_headers()_、_trailers()_ 和 _version()_。
### [**7.1. _URI_ of Response Object**](https://www.baeldung.com/java-9-http-client#1-uri-of-response-object)

The method _uri()_ on the response object returns the _URI_ from which we received the response.

响应对象上的 _uri()_ 方法会返回我们收到响应的 _URI_ 地址。

Sometimes it can be different than _URI_ in the request object because a redirection may occur:

有时它可能与请求对象中的 _URI_ 不同，因为可能会发生重定向：

```java
assertThat(request.uri()
  .toString(), equalTo("http://stackoverflow.com"));
assertThat(response.uri()
  .toString(), equalTo("https://stackoverflow.com/"));
```

### [**7.2. Headers from Response**](https://www.baeldung.com/java-9-http-client#2-headers-from-response)

We can obtain headers from the response by calling method _headers()_ on a response object:

我们可以通过调用响应对象上的 _headers()_ 方法来获取响应的标题：

```java
HttpResponse<String> response = HttpClient.newHttpClient()
  .send(request, HttpResponse.BodyHandlers.ofString());
HttpHeaders responseHeaders = response.headers();
```

It returns _HttpHeaders_ object, which represents a read-only view of HTTP Headers.

它返回 _HttpHeaders_ 对象，该对象表示 HTTP 头信息的只读视图。

It has some useful methods that simplify searching for headers value.

它有一些有用的方法，可以简化头信息值的搜索。

### [**7.3. Version of the Response**](https://www.baeldung.com/java-9-http-client#3-version-of-the-response)

The method _version()_ defines which version of HTTP protocol was used to talk with a server.

方法 _version()_ 定义了与服务器通信时使用的 HTTP 协议版本。

**Remember that even if we define that we want to use HTTP/2, the server can answer via HTTP/1.1.**

**请记住，即使我们定义要使用 HTTP/2，服务器也可以通过 HTTP/1.1 进行应答**。

The version in which the server answered is specified in the response:

响应中指定了服务器应答的版本：

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .version(HttpClient.Version.HTTP_2)
  .GET()
  .build();
HttpResponse<String> response = HttpClient.newHttpClient()
  .send(request, HttpResponse.BodyHandlers.ofString());
assertThat(response.version(), equalTo(HttpClient.Version.HTTP_1_1));
```

## [8. Handling Push Promises in HTTP/2](https://www.baeldung.com/java-9-http-client#handling-push-promises-in-http2)

New _HttpClient_ supports push promises through _PushPromiseHandler_ interface_._ 

新的 _HttpClient_ 通过 _PushPromiseHandler_ 接口支持推送承诺。

**It allows the server to “push” content to the client additional resources while requesting the primary resource, saving more roundtrip and as a result improves performance in page rendering.**

**它允许服务器在请求主要资源的同时向客户端 "推送 "附加资源内容，从而节省了更多的往返时间，并因此提高了页面渲染的性能。**

It is really the multiplexing feature of HTTP/2 that allows us to forget about resource bundling. For each resource, the server sends a special request, known as a push promise to the client.

实际上，正是 HTTP/2 的多路复用功能让我们忘记了资源捆绑。对于每个资源，服务器都会向客户端发送一个特殊请求，即推送承诺。

Push promises received, if any, are handled by the given _PushPromiseHandler_. A null valued _PushPromiseHandler_ rejects any push promises.

收到的推送承诺（如果有）将由给定的 _PushPromiseHandler_ 处理。空值_PushPromiseHandler_将拒绝任何推送承诺。

The _HttpClient_ has an overloaded **sendAsync** method that allows us to handle such promises, as shown below.

如下所示，_HttpClient_ 有一个重载的 **sendAsync** 方法，允许我们处理此类承诺。

Let's first create a _PushPromiseHandler_:

让我们先创建一个 _PushPromiseHandler_ ：

```java
private static PushPromiseHandler<String> pushPromiseHandler() {
    return (HttpRequest initiatingRequest, 
        HttpRequest pushPromiseRequest, 
        Function<HttpResponse.BodyHandler<String>, 
        CompletableFuture<HttpResponse<String>>> acceptor) -> {
        acceptor.apply(BodyHandlers.ofString())
            .thenAccept(resp -> {
                System.out.println(" Pushed response: " + resp.uri() + ", headers: " + resp.headers());
            });
        System.out.println("Promise request: " + pushPromiseRequest.uri());
        System.out.println("Promise request: " + pushPromiseRequest.headers());
    };
}
```

Next, let's use _sendAsync_ method to handle this push promise:

接下来，让我们使用 _sendAsync_ 方法来处理这个推送承诺：

```java
httpClient.sendAsync(pageRequest, BodyHandlers.ofString(), pushPromiseHandler())
    .thenAccept(pageResponse -> {
        System.out.println("Page response status code: " + pageResponse.statusCode());
        System.out.println("Page response headers: " + pageResponse.headers());
        String responseBody = pageResponse.body();
        System.out.println(responseBody);
    })
    .join();
```

## [**9. Conclusion**](https://www.baeldung.com/java-9-http-client#conclusion)

In this article, we explored the Java 11 _HttpClient_ API that standardized the incubating HttpClient introduced in Java 9 with more powerful changes.

在本文中，我们探讨了 Java 11 _HttpClient_ API，它对 Java 9 中引入的孵化 HttpClient 进行了标准化，并做出了更强大的更改。

这篇文章讨论了JDK11全新的HTTP API，

The complete code used can be found [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11).

使用的完整代码可在 [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11) 上找到。

In the examples, we've used sample REST endpoints provided by _https://postman-echo.com_.

在示例中，我们使用了 _https://postman-echo.com_ 提供的 REST 端点示例。