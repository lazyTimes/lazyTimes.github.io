---
title: Jmeter压力测试回顾(二)
subtitle: jmeter压力测试回顾
url_suffix: jmeter-test1
author: lazytime
tags:
  - jmeter
categories:
  - 压力测试
keywords: jmeter压力测试回顾
description: jmeter压力测试回顾
copyright: true
date: 2021-01-18 23:35:05
---

# 回顾Jmeter压力测试（二）

本部分内容为第一小节，介绍第一部分的Jmter测试内容，新建测试计划之后，点击`添加`,可以看到一个测试计划单所有菜单内容。

<!-- more -->

主要介绍的菜单为常用菜单，对于个人不常用的功能会进行忽略。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109121653.png)

## 测试计划菜单介绍

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109181129.png)

## 线程（用户）

+ 线程组：线程组元素是任何测试计划的起点。所有控制器和采样器必须在线程组下
+ setUp Thread Group：一种特殊类型的ThreadGroup的，可用于执行预测试操作。这些线程的行为完全像一个正常的线程组元件。不同的是，这些类型的线程执行测试前进行定期线程组的执行。
+  teardown thread group：一种特殊类型的ThreadGroup的，可用于执行测试后动作。这些线程的行为完全像一个正常的线程组元件。不同的是，这些类型的线程执行测试结束后执行定期的线程组。

> 这里可能还是不太懂，可以参考junit的setup ，teardown

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109181048.png)

## 配置元件

配置元件（config element）用于提供对静态数据配置的支持。CSV Data Set config 可以将本地数据文件形成数据池（Data Pool），而对应于HTTP Request Sampler和 TCP Request Sampler等类型的配制无件则可以修改Sampler的默认数据。（例如，HTTP Cookie Manager 可以用于对 HTTP Request Sampler 的cookie 进行管理）

## 监听器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109181245.png)

监听器，顾名思义就是用来监听测试结果的，可以看到对应的请求需要配置不同的监听器。最常用的功能是 **查看结果树**，**聚合报告**等，在后续的功能介绍中会进行具体的使用：

## 定时器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109182306.png)

定时器个人没有进行过实践，略过。。。。。

## 前置处理器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109182410.png)

用于在实际的请求发出之前对即将发出的请求进行特殊处理。例如，HTTP URL重写修复符则可以实现URL重写，当RUL中有sessionID 一类的session信息时，可以通过该处理器填充发出请求的实际的sessionID 。

## 后置处理器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109182703.png)

用于对Sampler 发出请求后得到的服务器响应进行处理。一般用来提取响应中的特定数据（类似LoadRunner测试工具中的关联概念）。例如，XPath Extractor 则可以用于提取响应数据中通过给定XPath 值获得的数据。

## 断言：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109182827.png)

断言用于检查测试中得到的相应数据等是否符合预期，断言一般用来设置检查点，用以保证性能测试过程中的数据交互是否与预期一致。

> 请注意，断言适用于其[范围内的](#scoping_rules)所有采样器。要将声明限制为单个采样器，请将该声明添加为采样器的子代。

## 测试片段：

“测试片段”元素是一种特殊类型的[控制器](#controllers)，它与线程组元素位于同一级别的“测试计划”树上。它与线程组的区别在于，除非[模块控制器](../usermanual/component_reference.html#Module_Controller)或[Include_Controller](../usermanual/component_reference.html#Include_Controller)引用它，否则它不会执行。该元素仅用于测试计划中的代码重用

## 非测试元件：

可以在这里找到对应的Http代理服务器设置，可以配置http代理，方便进行代理服务器进行并发测试。

## 各元件启动顺序：

1. 配置元素
2. 预处理器
3. 计时器
4. 取样器
5. 后处理器（除非SampleResult为`null`）
6. 断言（除非SampleResult为`null`）
7. 侦听器（除非SampleResult为`null`）

> 请注意，计时器，断言，预处理器和后处理器仅在有适用于其的采样器时才进行处理。逻辑控制器和采样器按照它们在树中出现的顺序进行处理。其他测试元素将根据其发现范围和测试元素的类型进行处理。[在一个类型内，元素按照它们在树中出现的顺序进行处理]。

# 线程组的菜单介绍：

## 取样器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109184654.png)



采样器告诉JMeter将请求发送到服务器并等待响应。它们按照在树中出现的顺序进行处理。控制器可用于修改采样器的重复次数。

JMeter采样器包括：

- FTP请求
- HTTP请求（也可用于SOAP或REST Web服务）
- JDBC请求
- Java对象请求
- JMS请求
- JUnit测试请求
- LDAP要求
- 邮件要求
- 操作系统进程请求
- TCP请求

**切记在测试计划中添加一个侦听器**，否则最终的结果是看不到的

## 逻辑控制器：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210109190006.png)



逻辑控制器使您可以自定义JMeter用于确定何时发送请求的逻辑，为了理解逻辑控制器对测试计划的影响，考虑以下测试树

> - Test Plan
>
> - - Thread Group
>
>   - - Once Only Controller
>
>     - - Login Request (an [HTTP Request](../usermanual/component_reference.html#HTTP_Request))
>
>     - Load Search Page (HTTP Sampler)
>
>     - **Interleave Controller**
>
>     - - **Search "A" (HTTP Sampler)**
>       - **Search "B" (HTTP Sampler)**
>       - **HTTP default request (Configuration Element)**
>
>     - HTTP default request (Configuration Element)
>
>     - Cookie Manager (Configuration Element)

可以看到，使用逻辑控制器可以组合出各种复杂的请求。