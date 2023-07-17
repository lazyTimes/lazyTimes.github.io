---
title: 【Java】关于 AccessController.doPrivileged 方法的个人理解
subtitle: AccessController.doPrivileged 相关使用和理解
url_suffix: doPrivileged
author: lazytime
tags:
  - Java
categories:
  - Java
keywords: ['doPrivileged']
description: 特权访问的相关内容
copyright: true
date: 2023-07-16 17:39:37
---
# 引言

**AccessController.doPrivileged** 这个方法调用通常存在于JDK的一些源码中，但是查阅相关资料介绍大差不差，比较难理解，这里简单整理资料做一个笔记，待日后有更深层次实践和理解之后回顾。

> 当然更多情况这个方法的作用大概率会被遗忘，不过根据英文单词一知半解也无伤大雅。

# 什么是特权访问？

从`Privileged`单词，个人联想到有关Java的安全管理的知识点，于是上网做了个科普。

在Java中执行应用程序分为**本地执行**和**远程执行**。本地代码调用执行是访问安全的，因为它受限于机器的JVM，同样它也可以获取JVM并且可以访问一切本地资源。而远程代码则不同，它基于**沙箱机制**，只能访问限定在JVM特定的运行范围中的资源。

与其说是“授权”，不如说是在JVM中进行一些敏感操作的时候，实际上还需要一层“黑盒”来加限制。Java的安全管理基本上可以理解为JVM的**防火墙**，运行一些“可接受”的操作，但是禁止“越权”访问。

<!-- more -->

# `AccessController.doPrivileged` 源码注释解读

```java
/**
Performs the specified PrivilegedExceptionAction with privileges enabled. The action is performed with all of the permissions possessed by the caller's protection domain.
If the action's run method throws an unchecked exception, it will propagate through this method.
Note that any DomainCombiner associated with the current AccessControlContext will be ignored while the action is performed.
*/
@CallerSensitive  
public static native <T> T  
    doPrivileged(PrivilegedExceptionAction<T> action)  
    throws PrivilegedActionException;
```

在**启用权限的情况下执行**指定的`PrivilegedExceptionAction`。

该操作以**调用者的保护域**所拥有的所有权限执行。如果该操作的运行方法抛出一个未选中的异常，该异常将通过该方法传播。

请注意，在执行该操作时，与当前`AccessControlContext`关联的`DomainCombiner`将被忽略。

这一段比较难理解的地方是我们需要弄清楚**拥有特权的调用者具体指的是什么？** 所以我们再看看看一下 `java.security.AccessController` 整个类的**JavaDoc**:  

> A caller can be marked as being "privileged" (see doPrivileged and below). When making access control decisions, the checkPermission method stops checking if it reaches a caller that was marked as "privileged" via a doPrivileged call without a context argument (see below for information about a context argument). If that caller's domain has the specified permission, no further checking is done and checkPermission returns quietly, indicating that the requested access is allowed. If that domain does not have the specified permission, an exception is thrown, as usual. 

其中提到的**no further checking is done**的意思是指**stack中的checking** ，也就是JVM中的栈帧，他被封装在虚拟机栈的不同线程的栈内存当中。

下面举个具体的例子方便更好理解：

假如一个TestService2，文件操作在1，stack为（1,2,3为checking顺序）  

3 . file:/D:/Workspaces/ExchangeConnect_V2_Trunk_Maven_workspace/ServiceCentre/bin/*  

2 . file:/c:/TestService-1.0.jar  

1.  file:/c:/TestService2-1.0.jar  

此时 checking顺序为  **1->2->3**。

1. 如果`doPrivileged`是在2中调用，那么1,2需要具有权限，3不再进行检查 。

2. 如果`doPrivileged`是在1中调用，那么1需要具有权限，2,3不再进行检查 。

注意：  

1.  这里容易理解错误的地方是**checking顺序**，例如一个调用链 MethodA->MethodB->MethodC（这里的3个方法需要在3个不同的ProtectionDomain中），doPrivileged 在 MethodB 中，很容易理解成**检查A,B而不检查C**，实际上stack中检查顺序为**C->B->A**，也就是**检查C,B而不检查A**  

> JVM 虚拟机栈的栈帧是先进后出的结构。 

2. **ServiceCentre** 不需要太多权限，而Service就需要使用doPrivileged来避免受到ServiceCentre的权限限制（如果service有足够的权限），**Equinox**中有很多这样的例子（Equinox扮演Service的角色）。

# 个人理解

`AccessController.doPrivileged`是一个在`AccessController`类中的静态方法。调用这个方法之后意味着它的代码主体可以享受"privileged(特权)"。

也就是说如果一个应用开启了安全管理，其他应用想要访问一些**授权方法**时，必须使用**doPrivileged方法开启特权**才行。

# 什么时候使用？

基本上属于看完就忘的一个东西，除了在JDK源码中对于一些敏感操作进行保护之外，很难看到使用场景。

不过在`StackFlow`上还是有大佬给出了自己的看法，具体可以看下面这个帖子：

[https://stackoverflow.com/questions/2233761/when-should-accesscontroller-doprivileged-be-used](https://stackoverflow.com/questions/2233761/when-should-accesscontroller-doprivileged-be-used)

> Imagine you've built an application that provides a number of services to pluggable modules. So your app and its services are trusted code. The pluggable modules, however, are not necessarily trusted and are loaded in their own class loaders (and have their own protection domains).

设想您构建了一个应用程序，为可插拔模块提供大量服务。因此，您的应用程序及其服务都是可信代码。然而，可插拔模块并不一定是可信的，它们被加载到自己的类加载器中（并且有自己的保护域）。

> When a pluggable module invokes a service, you are implementing custom security checks ("does pluggable module X have permission to use this service"). But the service itself might require some core Java permission (read a system property, write to a file, etc). The code that requires these permissions is wrapped in a doPrivileged() so that the insufficient permissions from the untrusted pluggable modules are effectively ignored - only the privileges of your trusted services module apply.

当可插拔模块调用服务时，您需要执行自定义的安全检查（"可插拔模块X是否有权限使用该服务"）。但服务本身可能需要一些核心 Java 权限（读取系统属性、写入文件等）。

**需要这些权限的代码被封装在 doPrivileged() 中，这样就可以有效地忽略来自不信任的可插拔模块的不充分权限** - 只有信任的服务模块的权限才适用。

# 写到最后

结合上面的内容，我们基本可以做一个总结：对于需要一些需要利用敏感资源进行网络交互场景，有时候会出现 **没有访问权限**的情况，这时候需要用`doPrivileged()`包裹相关操作代码来获取权限。

这个方法最常用和最红熟悉的使用场景，是Socket当中获取IO流的时候，此时的线程需要受到封装保护和授权。

**java.net.Socket#getInputStream**

```java
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

# 参考资料

## 什么时候使用AccessController.doPrivileged？

[https://stackoverflow.com/questions/2233761/when-should-accesscontroller-doprivileged-be-used](https://stackoverflow.com/questions/2233761/when-should-accesscontroller-doprivileged-be-used)
