---
title: 《Spring 实战》解读
subtitle: 《Spring 实战》解读
author: lazytime
url_suffix: note2
tags:
  - Spring
categories:
  - Spring
keywords: 《Spring 实战》解读
description: 《Spring 实战》解读
copyright: true
date: 2020-07-26 17:33:41
---

# 《Spring 实战》解读

## 基本概念

DI：依赖反转

AOP：切面编程



<!-- more -->

### Spring应用上下文

**AnnotationConfigApplicationContext** ：从一个或多个基于 Java 的配置类中加载 Spring 应用上下文。

**AnnotationConfigWebApplicationContext** ：从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文。

**ClassPathXmlApplicationContext** ：从类路径下的一个或多个 XML 配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。

**FileSystemXmlapplicationcontext** ：从文件系统下的一个或多个 XML 配置文件中加载上下文定义。

**XmlWebApplicationContext** ：从 Web 应用下的一个或多个 XML 配置文件中加载上下文定义。

### Spring的生命周期

![Spring生命周期](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200213191203.png?ynotemdtimestamp=1595748679079)

1. Spring 对 bean 进行实例化；
2. Spring 将值和 bean 的引用注入到 bean 对应的属性中；
3. 如果 bean 实现了 BeanNameAware 接口， Spring 将 bean 的 ID 传递给 setBean-Name() 方法；
4. 如果 bean 实现了 BeanFactoryAware 接口， Spring 将调用 setBeanFactory() 方法，将 BeanFactory 容器实例传入；
5. 如果 bean 实现了 ApplicationContextAware 接口， Spring 将调用 setApplicationContext() 方法，将 bean 所在的应用上下文的引用传入进来；
6. 如果 bean 实现了 BeanPostProcessor 接口， Spring 将调用它们的 post-ProcessBeforeInitialization() 方法；
7. 如果 bean 实现了 InitializingBean 接口， Spring 将调用它们的 after-PropertiesSet() 方法。类似地，如果 bean 使用 init- method 声明了初始化方法，该方法也会被调用；
8. 如果 bean 实现了 BeanPostProcessor 接口， Spring 将调用它们的 post-ProcessAfterInitialization() 方法；
9. 此时， bean 已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；
10. 如果 bean 实现了 DisposableBean 接口， Spring 将调用它的 destroy() 接口方法。同样，如果 bean 使用 destroy-method 声明了销 毁方法，该方法也会被调用

### Spring 模块 （4.0为例）

![Spring模块](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200213191506.png?ynotemdtimestamp=1595748679079)

### 版本新特性

3.1 ： 注解编程

3.2：增强SpringMVC，加入更多的支持

4.0：Websocket加入，JDK1.8的支持以及Lambada表达式

## Spring的Bean

### 装配Bean的三种方式：

1. 自动装配（扫描）
2. XML装配
3. 通过JAVA代码

### 自动装配

> Q：**为Bean命名有什么影响？**
>
> A： **@Inject** 和 **@Autowired** 的注入会受到影响
>
> Q：**Bean默认的命名规则？**
>
> A：类名的首字母小写
>
> Q：除了@Bean定义名称之外的办法？
>
> A：**@Named**

### JAVA配置

需要注意Bean是单例的，如果引用对象也存在引用，那么对应的引用对象也是单利的

### XML配置

1. 使用 <constructor-arg> 元素进行构造器参数的注入
2. 由于xml 配置非常的繁琐，推荐Java配置

## 更高级的Bean装配

### profile Bean

`@Profile` 注解指定某个 bean 属于哪一个 profile

> 3.2之前：@profile只能作用于类

### 激活profile

```
spring.profiles.active` 和 `spring.profiles.default
```

通常情况下会选择`dev`

### 条件化 Bean

**@Conditional**：SpringBoot的核心

运作基本原理：

它通过给定的 `ConditionContext`对象进而得到`Environment`对象，并使用这个对象检查环境中是否存在名为 magic 的环境属性

> 通过 ConditionContext ，我们可以做到如下几点：
>
> - 借助 `getRegistry()` 返回的 BeanDefinitionRegistry 检查 bean 定义；
> - 借助 `getBeanFactory()`返回的 ConfigurableListableBeanFactory 检查 bean 是否存在，甚至探查 bean 的属性；
> - 借助 `getEnvironment()` 返回的 Environment 检查环境变量是否存在以及它的值是什么；
> - 读取并探查 `getResourceLoader()` 返回的 ResourceLoader 所加载的资源；
> - 借助 `getClassLoader()`返回的 ClassLoader 加载并检查类是否存在。

#### 标示首选的 bean

`@Primary`：当自动装配Bean遇到歧义的时候可以设置首选Bean

#### 限定自动装配的 bean

`@Qualifier`：注解是使用限定符的主要方式

注意：另一种使用方式是定义一个注解然后使用限定装配符号实现自己标记

## Bean的作用域

单例（ Singleton ）：在整个应用中，只创建 bean 的一个实例。

原型（ Prototype ）：每次注入或者通过 Spring 应用上下文获取的时候，都会创建一个新的 bean 实例。

会话（ Session ）：在 Web 应用中，为每个会话创建一个 bean 实例。

请求（ Rquest ）：在 Web 应用中，为每个请求创建一个 bean 实例。

配置方式：`@Scope`

注意在创建时候使用的是代理而不是真正的New 一个对象

## 运行时注入

Spring提供了两种办法来实现属性注入

- 属性占位符（ Property placeholder ）
- Spring 表达式语言（ SpEL ）

# SpringMVC

### DispatcherServelet

DispatcherServlet 是 Spring MVC 的核心。在这里请求会第一次接触到框架，它要负责将请求路由到其他的组件之中。

`AbstractAnnotationConfigDispatcherServletInitializer`核心实现类

## Spring Web Flow

## 是什么：

一种基于会话的流程式框架

## 为什么会有web Flow

在解决web与用户的交互流程方面做了很大的帮助和改进，能够实现一种类似OA的流程式业务

## 怎么做：

1. 看官网：https://projects.spring.io/spring-webflow/
2. 简书博客：https://www.jianshu.com/p/6c44902a8e4e
3. 对比：https://www.cnblogs.com/Piers/p/6588290.html

## 总结：

由于大量的XML配置已经不适合当前时代的发展了，还是建议使用MVC吧

某些一对一的服务下面可能会有奇效

# Spring Security

## 是什么？

保护web 应用程序的权限，以及保护程序安全的一个框架

## 为什么？

防止非法用户的攻击行为，对于用户的权限进行限定

## 同类产品比较：

**Apach Shiro**：Spring4 之后倾向于使用Apach Shiro的支持，因为该框架配置更加灵活，实现更加简单，最关键是不仅仅限制在Web框架！！！

## 怎么做：

1. 看官网：https://spring.io/projects/spring-security
2. 百度或者谷歌查看案例实战

此框架只有遇到实际应用才能掌握，不做详细笔记

## 注意事项：

- 配置用户存储；
- 指定哪些请求需要认证，哪些请求不需要认证，以及所需要的权限；
- 提供一个自定义的登录页面，替代原来简单的默认登录页。

# AMQP

## 是什么？

一种新的消息通信协议，不同于JMS，提供更加灵活的方式进行JAVA的消息推送

## 和JMS的区别

不同于使用“通道”的概念，AMQP使用`Exchange`作为消息的暂存区域

> Exchange的类型：
>
> Direct ：如果消息的 routing key 与 binding 的 routing key 直接匹配的话，消息将会路由到该队列上； Topic ：如果消息的 routing key 与 binding 的 routing key 符合通配符匹配的话，消息将会路由到该队列上； Headers ：如果消息参数表中的头信息和值都与 bingding 参数表中相匹配，消息将会路由到该队列上； Fanout ：不管消息的 routing key 和参数表的头信息 / 值是什么，消息将会路由到所有队列上。