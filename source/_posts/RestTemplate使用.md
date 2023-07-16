---
title: RestTemplate使用
subtitle: RestTemplate日常的使用和常见API
author: lazytime
url_suffix: note15
tags:
  - SpringBoot
categories:
  - SpringBoot
keywords: RestTemplate日常使用
description: RestTemplate
copyright: true
date: 2020-08-22 19:34:40
---

# `RestTemplate`介绍

## 借鉴

https://juejin.im/post/6844903842065154061 掘金

<!-- more -->

## 认识 `RestTemplate`

简要改过：HTTP请求工具，封装了HttpClient请求的一系列方法

所属：[spring.web](http://spring.web) 包 4.1.3 之后

**Spring Framework 3.0之后才开始引入**

## 为什么会出现`RestTemplate`

1. 最大的作用就是简化Http请求。
2. 无需自己编写底层
3. 开箱即用

## Spring 5.0之后的改变

5.0之后，官方更加推荐应对非阻塞响应式的 HTTP 请求处理类 `org.springframework.web.reactive.client.WebClient`来处理

## 一些常用API

其中多数是单个方法重载实现，这里我主要参考官方文档 [rest-client-access](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/integration.html#rest-client-access) 进行如下分类：

| 方法名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `getForObject`    | 通过 GET 请求获得响应结果                                    |
| `getForEntity`    | 通过 GET 请求获取 `ResponseEntity` 对象，包容有状态码，响应头和响应数据 |
| `headForHeaders`  | 以 HEAD 请求资源返回所有响应头信息                           |
| `postForLocation` | 用 POST 请求创建资源，并返回响应数据中响应头的字段 `Location`的数据 |
| `postForObject`   | 通过 POST 请求创建资源，获得响应结果                         |
| `put`             | 通过 PUT 方式请求来创建或者更新资源                          |
| `patchForObject`  | 通过 PATH 方式请求来更新资源，并获得响应结果。(JDK `HttpURLConnection` 不支持 PATH 方式请求，其他 HTTP 客户端库支持) |
| `delete`          | 通过 DELETE 方式删除资源                                     |
| `optionsForAllow` | 通过 ALLOW 方式请求来获得资源所允许访问的所有 HTTP 方法，可用看某个请求支持哪些请求方式 |
| `exchange`        | 更通用版本的请求处理方法，接受一个 `RequestEntity` 对象，可以设置路径，请求头，请求信息等，最后返回一个 `ResponseEntity` 实体 |
| `execute`         | 最通用的执行 HTTP 请求的方法，上面所有方法都是基于 `execute`的封装，全面控制请求信息，并通过回调接口获得响应数据 |

## 常见用法：

### GET 请求：

#### 使用GET请求获取JSON字符串:

我们只需要调用：`getForObject`即可

```
String forObject = build.getForObject("http://www.baidu.com", String.class);
.....
```

#### 使用GET请求获取JSON映射对象：

只需要将后面的参数改为 对应实体对象即可

```
Product forObject = build.getForObject("http://www.baidu.com", Product.class);
```

#### 使用GET请求获取更加详细的Entity对象

使用`getForEntity`即可获取更为详细的对象

```
ResponseEntity<String> forEntity = build.getForEntity("http://www.baidu.com", String.class);
```

#### 构建自定义的请求头：

```
 MultiValueMap header = new LinkedMultiValueMap();
        header.add(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
        HttpEntity<Object> requestEntity = new HttpEntity<>(header);
```

#### 构建自定义请求header

一般使用`exchange`的相关方法

```
ResponseEntity<String> exchange = build.exchange("http://www.baidu.com", HttpMethod.GET, requestEntity, String.class);
        System.out.println("get_product1返回结果：" + exchange);
```

#### 手动处理请求头以及返回信息

```
build.execute("http://www.baidu.com", HttpMethod.GET, request -> {
            request.getHeaders().add(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
        }, response -> {
            HttpStatus statusCode = response.getStatusCode();
            int value = statusCode.value();
            System.err.println("响应code: = " + value);
            return value;
        });
```

这里有两个比较关键的接口

**RequestCallback requestCallback,**

**ResponseExtractor<T> responseExtractor**

可以使用lambada 表达式简化，或者使用匿名内部类

#### 使用带参数的Get请求

1. 首先我们需要定义占位符

```
String url ="htttp://localhost:8080?id={id}"
```

注意{id} 的写法,有点像`@PathValiable` 的写法

##### 使用动态参数：

```
// 带参数的GET请求
String url = "http://localhost:7474?id={id}";
build.getForObject(url, String.class, 11);
```

##### 使用map设置参数

```
// 使用map 设置参数
Map<String, Object> uriVariables = new HashMap<>();
uriVariables.put("id", 101);
build.getForObject(url, String.class, uriVariables);
```

### POST 请求：

Post请求相对GET请求有更多的变化，这里也是列出日常开放比较常用的点

#### 发送 `Content-Type` 为 `application/x-www-form-urlencoded`的 POST 请求：

```
// 设置请求头
MultiValueMap multiValueMap = new LinkedMultiValueMap();
multiValueMap.add(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_FORM_URLENCODED_VALUE);

User user = new User();
//方式二： 将请求参数值以 K=V 方式用 & 拼接，发送请求使用
String productStr = "id=" +  user.getId() + "&name=" + user.getName() + "&price=" + user.getSalt();
// 构建form参数的url
HttpEntity<String> request = new HttpEntity<>(productStr, header);
ResponseEntity<String> exchange1 = build.exchange(url, HttpMethod.POST, request, String.class);

// 方式3 ，使用map
// 构建form参数的url
HttpEntity<String> request = new HttpEntity<>(productStr, header);
ResponseEntity<String> exchange1 = build.exchange(url, HttpMethod.POST, request, String.class);

Map<String, Object> stringObjectMap = new HashMap<>();
stringObjectMap.put("test", "123");
stringObjectMap.put("test1", "123");
stringObjectMap.put("test2", "123");
HttpEntity httpEntity = new HttpEntity<>(stringObjectMap, header);
ResponseEntity exchange2 = build.exchange(url, HttpMethod.POST, httpEntity, String.class);
```

#### 发送 `Content-Type` 为 `application/json` 的 POST 请求：

只需要将上面的案例当中的:

`MediaType.APPLICATION_FORM_URLENCODED_VALUE`改为：`MediaType.APPLICATION_JSON_VALUE`即可

### DELETE 请求以及PUT请求

因为都是`RestFul`的请求，所以简单描述

```
String url = "http://localhost:8080/product/update";
Map<String, ?> variables = new HashMap<>();
MultiValueMap<String, String> header = new LinkedMultiValueMap();
header.put(HttpHeaders.CONTENT_TYPE, Arrays.asList(MediaType.APPLICATION_FORM_URLENCODED_VALUE));
Product product = new Product(101, "iWatch", BigDecimal.valueOf(2333));
String productStr = "id=" + product.getId() + "&name=" + product.getName() + "&price=" + product.getPrice();
HttpEntity<String> request = new HttpEntity<>(productStr, header);
restTemplate.put(url, request);
```

### 使用restTemplate实现文件上传

1. 需要设置multitype/file 文件请求头
2. 需要设置headaer 以及 body
3. 只能使用post请求

```
String ui = "";
MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
FileSystemResource fileSystemResource = new FileSystemResource();
body.add("file", fileSystemResource);

MultiValueMap<String, String> header2 = new LinkedMultiValueMap();
header2.put(HttpHeaders.CONTENT_TYPE, Arrays.asList(MediaType.MULTIPART_FORM_DATA_VALUE));
HttpEntity httpquest = new HttpEntity<>(body, header2);
ResponseEntity<String> xxx = build.postForEntity("xxx", httpquest, String.class);
```

## 进阶 RestTemplate

### 如何设置RestTempalate的默认底层引擎

RestTemplate 默认使用的引擎：

- Apache HttpComponents
- Netty
- OkHttp

一句话概括：

```
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

### 如何修改RestTemplate 的超时时间

参考代码：

```
RestTemplate customRestTemplate = new RestTemplate(getClientHttpRequestFactory());

private SimpleClientHttpRequestFactory getClientHttpRequestFactory() {
    SimpleClientHttpRequestFactory clientHttpRequestFactory
            = new SimpleClientHttpRequestFactory();
    // 连接超时设置 10s
    clientHttpRequestFactory.setConnectTimeout(10_000);

    // 读取超时设置 10s
    clientHttpRequestFactory.setReadTimeout(10_000);
    return clientHttpRequestFactory;
}
```

如果要调整 `HttpComponentsClient` 的超时设置，可以参考文章[resttemplate-timeout-example](https://howtodoinjava.com/spring-boot2/resttemplate-timeout-example/) 。当然除了设置超时时间之外，还有更多参数进行定制，这里就不一一列举，可以参考文章 [resttemplate-httpclient-java-config](https://howtodoinjava.com/spring-restful/resttemplate-httpclient-java-config/) 进一步学习。

## 参考资料

[www.baeldung.com/rest-templa…](https://www.baeldung.com/rest-template)

[blog.didispace.com/spring-boot…](http://blog.didispace.com/spring-boot-learning-21-1-1)

[www.baeldung.com/spring-rest…](https://www.baeldung.com/spring-rest-template-multipart-upload)

[www.zhihu.com/question/28…](https://www.zhihu.com/question/28557115)

[howtodoinjava.com/spring-boot…](https://howtodoinjava.com/spring-boot2/resttemplate-timeout-example)

[docs.spring.io/spring/docs…](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/integration.html#rest-client-access)

[zh.wikipedia.org/wiki/表现层状态转…](https://zh.wikipedia.org/wiki/表现层状态转换)

[docs.spring.io/spring-fram…](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestOperations.html)

作者：闻人的技术博客 链接：https://juejin.im/post/6844903842065154061 来源：掘金 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。