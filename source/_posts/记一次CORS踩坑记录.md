---
title: 记一次CORS踩坑记录
subtitle: 记一次CORS踩坑记录
author: lazytime
tags:
  - cors
categories:
  - 工作
keywords: 记一次CORS踩坑记录）
description: 记一次CORS踩坑记录
copyright: true
date: 2020-07-26 17:43:48
---

# 记一次CORS踩坑记录

今天远程工作的内容是实现CORS的跨域访问，本来特别简单的问题，却因为Spring的封装“坑”的有点惨。下面记录下整个过程



<!-- more -->

## 踩坑原因

1. 如果存在`web.xml` 配置，访问的时候会忽略掉其他的任何i形式的配置，只生效`xml`配置
2. 项目中配置允许post跨域无效，只能get跨域访问，原因未知

### 解决过程

1. 在请求当中加入`@crossxxx`注解，发现无效
2. 将请求改为具体的`RequestMethod.GET` 或者`POST`

## 参考博客

网上的博客有许多的参考解决方案 http://www.thomasyoung.cn/fontend/2017/04/01/ajax-session/ 个人博客，多了参数的介绍

https://segmentfault.com/a/1190000012469713 segament一个博客，写得非常棒

其他资料：

https://www.imooc.com/qadetail/302333 https://www.jianshu.com/p/f3840c8c02ba 简书介绍CORS

https://blog.csdn.net/doujinlong1/article/details/80357819 【业务知识——Logger日志打印规范】https://www.cnblogs.com/doit8791/p/8947024.html 【聊聊日志打印规范问题】 研发的同时抽时间看下

## 解决办法

使用原生的配置代替spring 的整合配置 最佳答案：http://www.thomasyoung.cn/fontend/2017/04/01/ajax-session

## 更改之后的配置

1. 使用`pom.xml`加入新的依赖，使用Cors的原生支持
2. 在`web.xml`当中增加如下配置
3. 配置的更详细介绍在上面提到的博客当中，这里不多赘述

```
<!-- 跨域配置:参考 http://www.thomasyoung.cn/fontend/2017/04/01/ajax-session/-->
    <filter>
        <!-- 原生Cors过滤器配置 -->
        <filter-name>CorsFilter</filter-name>
        <filter-class>com.thetransactioncompany.cors.CORSFilter</filter-class>

        <!--
		非跨域请求是否可以通过此过滤器，默认为true；如果设置为false，则只有跨域的请求被允许
          -->
        <init-param>
            <param-name>cors.allowGenericHttpRequests</param-name>
            <param-value>true</param-value>
        </init-param>

        <init-param>
            <param-name>cors.allowOrigin</param-name>
            <param-value>http://192.168.92.190:8080,http://192.168.92.145:8080,</param-value>
        </init-param>
        <!--
        是否允许来自allowOrigin的子域名的请求，默认不允许；子域名的概念大家应该清楚，www.example.com是example.com的子域名
        -->
        <init-param>
            <param-name>cors.allowSubdomains</param-name>
            <param-value>false</param-value>
        </init-param>
        <!-- cors.allowed.methods -->
        <init-param>
            <param-name>cors.supportedMethods</param-name>
            <param-value>GET,POST,HEAD,OPTIONS,PUT</param-value>
        </init-param>
        <!-- cors.allowed.headers -->
        <init-param>
            <param-name>cors.supportedHeaders</param-name>
            <param-value>Content-Type,X-Requested-With,accept,Origin,Access-Control-Request-Method,Access-Control-Request-Headers</param-value>
        </init-param>

        <init-param>
            <param-name>cors.exposedHeaders</param-name>
            <!--这里可以添加一些自己的暴露Headers   -->
            <param-value>Access-Control-Allow-Origin,Access-Control-Allow-Credentials</param-value>
        </init-param>
        <!-- 对应 cors.support.credentials -->
        <init-param>
            <param-name>cors.supportsCredentials</param-name>
            <param-value>true</param-value>
        </init-param>
        <!-- cors.preflight.maxage 最大缓存时间 -->
        <init-param>
            <param-name>cors.maxAge</param-name>
            <param-value>10</param-value>
        </init-param>
    </filter>
  
    <filter-mapping>
        <filter-name>CorsFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

## IE8/IE9 解决拒绝访问的问题

### 第一种：引入js文件

```
<script src="http://cdnjs.cloudflare.com/ajax/libs/jquery-ajaxtransport-xdomainrequest/1.0.3/jquery.xdomainrequest.min.js" type="text/javascript" charset="utf-8"></script>
```

### 第二种：Jquery开启跨域模式

```
jQuery.support.cors = true
```