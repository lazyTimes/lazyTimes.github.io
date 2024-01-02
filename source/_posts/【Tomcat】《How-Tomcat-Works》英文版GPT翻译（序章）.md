---
title: 【Tomcat】《How Tomcat Works》英文版GPT翻译（序章）
subtitle: 原书GPT翻译
url_suffix: howtomcatgpt0
author: lazytime
tags:
  - java
categories:
  - java
keywords: howtomcatwork
description: gpt内容翻译
copyright: true
date: 2024-01-02 16:36:20
---

# Introduction

## Overview

Welcome to How Tomcat Works. 

This book dissects Tomcat 4.1.12 and 5.0.18 and explains the internal workings of its free, open source, and most popular servlet container code-named Catalina. 

Tomcat is a complex system, consisting of many different components. Those who want to learn how Tomcat works often do know where to start. What this book does is provide the big picture and then build a simpler version of each component to make understanding that component easier. Only after that will the real component be explained. 

You should start by reading this Introduction as it explains the structure of the book and gives you the brief outline of the applications built. The section "Preparing the Prerequisite Software" gives you instructions on what software you need to download, how to make a directory structure for your code, etc.

欢迎来到《深入剖析Tomcat》。本书对Tomcat 4.1.12和5.0.18进行了剖析，解释了其内部工作原理，这是一个名为Catalina的免费、开源和最受欢迎的Servlet容器。

Tomcat是一个复杂的系统，由许多不同的组件组成。那些想要了解Tomcat工作原理的人通常不知道从哪里开始。本书提供了整体框架，并构建了每个组件的简化版本，以便更容易理解该组件。只有在此之后才会解释真正的组件。

您应该从阅读本介绍开始，因为它解释了本书的结构，并为您提供了所构建应用程序的简要概述。"准备先决条件软件"（Preparing the Prerequisite Software）一节为您提供了下载软件、创建代码目录结构等指令。

<!-- more -->

# Who This Book Is for

This book is for anyone working with the Java technology.

- This book is for you if you are a servlet/JSP programmer or a Tomcat user and you are interested in knowing how a servlet container works.

- It is for you if you want to join the Tomcat development team because you need to first learn how the existing code works.

- If you have never been involved in web development but you have interest in software development in general, then you can learn from this book how a large application such as Tomcat was designed and developed.

- If you need to configure and customize Tomcat, you should read this book.

To understand the discussion in this book, you need to understand object-oriented programming in Java as well as servlet programming. 

If you are not familiar with the latter, there are a plethora of books on sendets, including Budi's Java for the Web with Servlets, JSP, and EJB. 

To make the material easier to understand, each chapter starts with background information that will be required to understand the topic in discussion.

这本书适用于任何使用Java技术的人。

- 如果你是一个servlet/JSP程序员或者一个Tomcat用户，并且你对servlet容器的工作原理感兴趣，那么这本书适合你。

- 如果你想加入Tomcat开发团队，因为你需要先了解现有代码的工作原理，那么这本书适合你。

- 如果你从未参与过Web开发，但对软件开发有兴趣，那么你可以通过这本书了解一个大型应用程序如何设计和开发，比如Tomcat。

- 如果你需要配置和定制Tomcat，你应该阅读这本书。

为了理解本书中的讨论内容，你需要了解Java的面向对象编程以及servlet编程。

如果你对后者不熟悉，有很多关于servlet的书籍可供参考，包括Budi的《Java for the Web with Servlets, JSP, and EJB》。

为了更容易理解材料，每一章都以背景信息开始，这些信息将有助于理解所讨论的主题。

# How A Servlet Container Works

A servlet container is a complex system. However, basically there are three things

that a servlet container does to service a request for a servlet:

- Creating a request object and populate it with information that may be used by the invoked servlet, such as parameters, headers, cookies, query string, URI, etc. A request object is an instance of the javax.servlet.ServletRequest interface or the javax.servlet.http.ServletRequest interface.

- Creating a response object that the invoked servlet uses to send the response to the web client. A response object is an instance of the javax.servlet.ServletResponse interface or the javax.servlet.http.ServletResponse interface.

- Invoking the service method of the servlet, passing the request and response objects. Here the servlet reads the values from the request object and writes to the response object.

As you read the chapters, you will find detailed discussions of Catalina servlet container.

servlet 容器是一个复杂的系统。不过，基本上有三件事

服务程序容器为服务程序请求所做的三件事：

创建一个**请求对象**，并填充它与被调用的Servlet可能使用的信息，例如参数、头部、cookie、查询字符串、URI等。请求对象是`javax.servlet.ServletRequest`接口或`javax.servlet.http.ServletRequest`接口的实例。

创建一个响应对象，被调用的Servlet使用它将响应发送给Web客户端。响应对象是`javax.servlet.ServletResponse`接口或`javax.servlet.http.ServletResponse`接口的实例。

调用Servlet的service方法，传递请求和响应对象。在这里，Servlet从请求对象中读取值，并写入响应对象。

在阅读章节时，您将找到有关Catalina servlet容器的详细讨论。

# Catalina Block Diagram

Catalina is a very sophisticated piece of software, which was elegantly designed and developed. It is also modular too. Based on the tasks mentioned in the section "How A Servlet Container Works", you can view Catalina as consisting of two main modules: the connector and the container. 

The block diagram in Figure I.1 is, of course, simplistic. Later in the following chapters you will unveil all smaller components one by one.

Now, back to Figure I.1, the connector is there to *connect* a request with the container. Its job is to construct a request object and a response object for each HTTP request it receives. It then passes processing to the container. The container receives the request and response objects from the connector and is responsible for invoking the servlet's service method.

Bear in mind though, that the description above is only the tip of the iceberg. There are a lot of things that a container does. For example, before it can invoke a servlet's service method, it must load the servlet, authenticate the user (if required), update the session for that user, etc. It's not surprising then that a container uses many different modules for processing. For example, the manager module is for processing user sessions, the loader is for loading servlet classess, etc.

Catalina是一款非常复杂的软件，设计和开发都非常优雅。它也是模块化的。根据“Servlet容器的工作原理”部分中提到的任务，你可以将Catalina视为由两个主要模块组成：连接器和容器。 

当然，**图I.1**中的块图是简化的。在接下来的章节中，你将逐个揭示所有较小的组件。

现在，回到图I.1，连接器的作用是将请求与容器进行连接。

它的任务是为每个接收到的HTTP请求构建一个请求对象和一个响应对象。然后将处理传递给容器。容器从连接器接收请求和响应对象，并负责调用Servlet的service方法。

请记住，上面的描述只是冰山一角。容器还有很多其他功能。例如，在调用Servlet的service方法之前，它必须加载Servlet，对用户进行身份验证（如果需要），更新该用户的会话等等。因此，容器使用许多不同的模块进行处理是不足为奇的。例如，管理模块用于处理用户会话，加载器用于加载Servlet类等等。

# **Tomcat 4 and 5** 

This book covers both Tomcat 4 and 5. Here are some of the differences between the two:

- Tomcat 5 supports Servlet 2.4 and JSP 2.0 specifications, Tomcat 4 supports Servlet 2.3 and JSP 1.2.

- Tomcat 5 has a more efficient default connector than Tomcat 4.

- Tomcat 5 shares a thread for background processing whereas Tomcat 4's  components all have their own threads for background processing. Therefore, Tomcat 5 uses less resources in this regard.

- Tomcat 5 does not need a mapper component to find a child component, therefore simplifying the code.

这本书涵盖了Tomcat 4和5的内容。以下是这两个版本之间的一些区别：

- Tomcat 5支持Servlet 2.4和JSP 2.0规范，而Tomcat 4支持Servlet 2.3和JSP 1.2。

- Tomcat 5的默认连接器比Tomcat 4更高效。

- Tomcat 5在后台处理时共享一个线程，而Tomcat 4的组件都有自己的后台处理线程。因此，在这方面Tomcat 5使用的资源更少。

- Tomcat 5不需要一个映射器组件来查找子组件，从而简化了代码。

# **Overview of Each Chapter**

There are 20 chapters in this book. The first two chapters serve as an introduction.

本书共 20 章。前两章为导论。 

**Chapter 1**：explains how an HTTP server works and Chapter 2 features a simple servlet container. The next two chapters focus on the connector and Chapters 5 to 20 cover each of the components in the container. The following is the summary of each of the chapters.

**第1章**：解释HTTP服务器的工作原理，第2章介绍一个简单的Servlet容器。接下来的两章重点介绍连接器，第5章至第20章介绍容器中的各个组件。以下是各章的摘要。

> Note For each chapter, there is an accompanying application similar to the 
>
> component being explained.
>
> 注意： 每一章都附有与所讲解组件类似的应用程序。

**Chapter 1** starts this book by presenting a simple HTTP server. To build a working HTTP server, you need to know the internal workings of two classes in the java.net package: Socket and ServerSocket. There is sufficient background information in this chapter about these two classes for you to understand how the accompanying application works.

**第一章**通过介绍一个简单的HTTP服务器来开始本书。要构建一个可工作的HTTP服务器，您需要了解java.net包中两个类的内部工作原理：Socket和ServerSocket。本章提供了关于这两个类的足够背景信息，以便您理解附带应用程序的工作原理。



**Chapter 2**：explains how simple servlet containers work. This chapter comes with two servlet container applications that can service requests for static resources as well as very simple servlets. In particular, you will learn how you can create request and response objects and pass them to the requested servlet's service method. There is also a servlet that can be run inside the servlet containers and that you can invoke from a web browser.

**第二章**：解释了简单的Servlet容器的工作原理。本章附带两个Servlet容器应用程序，可以处理对静态资源以及非常简单的Servlet的请求。特别是，您将学习如何创建请求和响应对象，并将它们传递给所请求的Servlet的service方法。还有一个可以在Servlet容器中运行的Servlet，您可以从Web浏览器中调用。



**Chapter 3**： presents a simplified version of Tomcat 4's default connector. The application built in this chapter serves as a learning tool to understand the connector discussed in Chapter 4.

**第三章**：介绍了Tomcat 4的默认连接器的简化版本。本章中构建的应用程序作为一个学习工具，用于理解第四章中讨论的连接器。



**Chapter 4** presents Tomcat 4's default connector. This connector has been deprecated in favor of a faster connector called Coyote. Nevertheless, the default connector is simpler and easier to understand.

**第4章**介绍了Tomcat 4的默认连接器。这个连接器已经被一个更快的连接器Coyote所取代。尽管如此，默认连接器更简单，更容易理解。



**Chapter 5**：discusses the container module. A container is represented by the org.apache.catalina.Container interface and there are four types of containers: engine, host, context, and wrapper. This chapter offers two applications that work with contexts and wrappers.

**第5章**讨论了容器模块。容器由org.apache.catalina.Container接口表示，有四种类型的容器：engine、host、context和wrapper。本章提供了两个与上下文和包装器一起工作的应用程序。



**Chapter 6**：explains the Lifecycle interface. This interface defines the lifecycle of a Catalina component and provides an elegant way of notifying other components of events that occur in that component. In addition, the Lifecycle interface provides an elegant mechanism for starting and stopping all the components in Catalina by one single start/stop.

**第6章**解释了Lifecycle接口。这个接口定义了Catalina组件的生命周期，并提供了一种优雅的方式来通知其他组件该组件中发生的事件。此外，Lifecycle接口通过一个单一的启动/停止方法提供了一个优雅的机制来启动和停止Catalina中的所有组件。



**Chapter 7**：covers loggers, which are components used for recording error messages and other messages.

**第7章**介绍了记录错误消息和其他消息的日志记录器。



**Chapter 8**： explains about loaders. A loader is an important Catalina module responsible for loading servlet and other classes that a web application uses. This chapter also shows how application reloading is achieved.

**第8章**解释了加载器。加载器是一个重要的Catalina模块，负责加载Web应用程序使用的servlet和其他类。本章还展示了如何实现应用程序重新加载。



**Chapter 9**： discusses the manager, the component that manages sessions in session management. It explains the various types of managers and how a manager can persist session objects into a store. At the end of the chapter, you will learn how to build an application that uses a StandardManager instance to run a servlet that uses session objects to store values.

**第9章**讨论了管理器，即会话管理中负责管理会话的组件。它解释了各种类型的管理器以及管理器如何将会话对象持久化到存储中。在本章结束时，您将学习如何构建一个使用StandardManager实例的应用程序，以运行一个使用会话对象存储值的servlet。



**Chapter 10**： covers web application security constraints for restricting access to certain contents. You will learn entities related to security such as principals, roles, login config, authenticators, etc. You will also write two applications that install an authenticator valve in the StandardContext object and uses basic authentication to authenticate users.

**第10章**：介绍了用于限制对某些内容访问的Web应用程序安全约束。您将学习与安全相关的实体，例如主体、角色、登录配置、认证器等。您还将编写两个应用程序，将一个认证器阀门安装在StandardContext对象中，并使用基本身份验证对用户进行身份验证。



**Chapter 11**：  explains in detail the org.apache.catalina.core.StandardWrapper class that represents a servlet in a web application. In particular, this chapter explains how filters and a servlet's service method are invoked. The application accompanying this chapter uses StandardWrapper instances to represents servlets.

**第11章**：详细解释了org.apache.catalina.core.StandardWrapper类，该类表示Web应用程序中的一个Servlet。特别是，本章解释了如何调用过滤器和Servlet的service方法。本章附带的应用程序使用StandardWrapper实例来表示Servlet。



**Chapter 12**：  covers the org.apache.catalina.core.StandardContext class that represents a web application. In particular this chapter discusses how a StandardContext object is configured, what happens in it for each incoming HTTP request, how it supports automatic reloading, and how Tomcat 5 shares a thread that executes periodic tasks in its associated components.

**第12章**：介绍了代表Web应用程序的org.apache.catalina.core.StandardContext类。特别是，本章讨论了如何配置StandardContext对象，每个传入的HTTP请求中会发生什么，它如何支持自动重新加载，以及Tomcat 5如何共享执行其关联组件中的定期任务的线程。



**Chapter 13**：presents the two other containers: host and engine. You can also find the standard implementation of these two containers: org.apache.catalina.core.StandardHost and org.apache.catalina.core.StandardEngine.

**第13章**：介绍了另外两个容器：主机和引擎。您还可以找到这两个容器的标准实现：org.apache.catalina.core.StandardHost和org.apache.catalina.core.StandardEngine。



**Chapter 14**： offers the server and service components. A server provides an elegant start and stop mechanism for the whole servlet container, a service serves as a holder for a container and one or more connectors. The application accompanying this chapter shows how to use a server and a service.

**第14章**：介绍了服务器和服务组件。服务器提供了整个Servlet容器的优雅启动和停止机制，服务作为容器和一个或多个连接器的持有者。本章附带的应用程序演示了如何使用服务器和服务。



**Chapters 15**： explains the configuration of a web application through Digester, an exciting open source project from the Apache Software Foundation. For those not initiated, this chapter presents a section that gently introduces the digester library and how to use it to convert the nodes in an XML document to Java objects. It then explains the ContextConfig object that configures a StandardContext instance.

**第15章**：通过Apache Software Foundation的令人兴奋的开源项目Digester详细解释了Web应用程序的配置。对于那些不熟悉的人，本章介绍了一个小节，温和地介绍了digester库以及如何使用它将XML文档中的节点转换为Java对象。然后解释了配置StandardContext实例的ContextConfig对象。



**Chapter 16**： explains the shutdown hook that Tomcat uses to always get a chance to do clean-up regardless how the user stops it (i.e. either appropriately by sending a shutdown command or inappropriately by simply closing the console.)

**第16章**：解释了Tomcat使用的关闭挂钩，以便始终有机会进行清理，无论用户如何停止它（即通过发送关闭命令或仅仅关闭控制台）。



**Chapter 17**： discusses the starting and stopping of Tomcat through the use of batch files and shell scripts.

**第17章**：通过批处理文件和shell脚本讨论了Tomcat的启动和停止。



**Chapter 18**： presents the deployer, the component responsible for deploying and installing web applications.

**第18章**：介绍了部署者，这个组件负责部署和安装Web应用程序。



**Chapter 19**： discusses a special interface, ContainerServlet, to give a servlet access to the Catalina internal objects. In partPreparing the Prerequicular, it discusses the Manager application that you can use to manage deployed applications.

**第19章**：讨论了一个特殊的接口ContainerServlet，用于使Servlet能够访问Catalina内部对象。在部分准备先决条件中，它讨论了您可以使用的管理器应用程序来管理已部署的应用程序。



**Chapter 20**： discusses JMX and how Tomcat make its internal objects manageable by creating MBeans for those objects.

**第20章**：讨论了JMX以及Tomcat如何通过为这些对象创建MBean使其内部对象可管理。

**The Application for Each Chapter** 

Each chapter comes with one or more applications that focus on a specific component in Catalina. Normally you'll find the simplified version of the component being explained or code that explains how to use a Catalina component. All classes and interfaces in the chapters' applications reside in the ex[*chapter**number*].pyrmont package or its subpackages. For example, the classes in the application in Chapter 1 are part of the ex01.pyrmont package.

**每章的应用程序**

每章都配有一个或多个应用程序，重点介绍Catalina中的特定组件。通常，您会找到被解释的组件的简化版本或解释如何使用Catalina组件的代码。所有章节应用程序中的类和接口都位于ex[*chapter**number*].pyrmont包或其子包中。例如，第1章中应用程序中的类属于ex01.pyrmont包。



# **Preparing the Prerequisite Software** 

The applications accompanying this book run with J2SE version **1.4**. The zipped source files can be downloaded from the authors' web site www.brainysoftware.com. It contains the source code for Tomcat 4.1.12 and the applications used in this book. Assuming you have installed J2SE 1.4 and your path environment variable includes  the location of the JDK, follow these steps:

1. Extract the zip files. All extracted files will reside in a new directory called **HowTomcatWorks.** HowTomcatWorks is your working directory. There will  be several subdirectories under HowTomcatWorks, including **lib** (containing  all needed libraries), **src** (containing the source files), **webroot** (containing an HTML file and three sample servlets), and **webapps** (containing sample applications).

2. Change directory to the working directory and compile the java files. If you are using Windows, run the **win-compile.bat** file. If your computer is a Linux machine, type the following: (don't forget to chmod the file if necessary) ./linux-compile.sh Note More information can be found in the Readme.txt file included in the ZIP file.

附带本书的应用程序运行在 J2SE 版本 **1.4** 上。可以从作者的网站 www.brainysoftware.com 下载压缩的源文件。它包含了 Tomcat 4.1.12 的源代码以及本书中使用的应用程序。假设您已经安装了 J2SE 1.4，并且您的路径环境变量包含了 JDK 的位置，请按照以下步骤进行操作：

1. 解压缩 ZIP 文件。所有解压后的文件将位于一个名为 **HowTomcatWorks** 的新目录中。HowTomcatWorks 是您的工作目录。在 HowTomcatWorks 下将有几个子目录，包括 **lib**（包含所有所需的库文件）、**src**（包含源文件）、**webroot**（包含一个 HTML 文件和三个示例 servlet）、以及 **webapps**（包含示例应用程序）。
2. 切换到工作目录并编译 Java 文件。如果您使用的是 Windows，运行 **win-compile.bat** 文件。如果您的计算机是 Linux 机器，请输入以下命令：（如果需要，请不要忘记为文件设置执行权限）./linux-compile.sh 更多信息可以在 ZIP 文件中附带的 Readme.txt 文件中找到。
