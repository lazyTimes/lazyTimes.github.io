---
title: 【Kafka】编译 Kafka2.7 源码并搭建源码环境（Ver 2.7.2）
subtitle: kafka源码环境搭建
author: lazytime
url_suffix: kafka-zk
tags:
  - Kafka
categories:
  - Kafka
keywords: Kafka
description: Kafka
copyright: true
date: 2023-03-07 21:17:48
---
# 前言

Kafka 是通过 Scala 和 Java共同编写的语言，之所以选择**2.7.2**的版本是因为这个版本的Kafka是最后一版本保留ZK的版本。

> 为什么不直接部署最新版代码？
>
> 因为过去很长一段时间Kafka都是和ZK配合的，并且有很多成熟项目都使用了带ZK的Kafka，去ZK的Kafka还有不少的路要走。

# 环境准备

-   JDK：1.8.0_351
-   Scala：2.12.8
-   Gradle：6.6
-   Zookeeper：3.4.14

# Kafka 2.7.2

建议fork一个官方的分支到自己的仓库，方便自己学习的时候添加注释等内容。

地址：[https://github.com/apache/kafka](https://github.com/apache/kafka)

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230304143229.png)


## 安装JDK 1.8

JDK的教程玩网上有非常多，寻找合适的JDK8的版本即可。

[https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html)

<!-- more -->

## 安装 Scala 2.12.8

### 下载

下载地址为：[https://www.scala-lang.org/download/2.12.8.html](https://www.scala-lang.org/download/2.12.8.html)

个人为Win 11，直接下载Windows 安装版本：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212201119.png)

个人是window系统，直接点击下一步即可，安装过程这里省略了。

### 配置 SCALA_HOME

安装完成之后是在对应的操作系统配置环境变量，个人在Path变量中增加**SCALA_HOME**，并且指定地址即可。因为个人是Windows 安装版本安装，已经自动配置了环境变量。安装完成之后结果如下：

```sh
C:\Users\adong>scala -version
Scala code runner version 2.12.8 -- Copyright 2002-2018, LAMP/EPFL and Lightbend, Inc.
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212201419.png)

## 安装 gradle 6.6

下载地址：[https://services.gradle.org/distributions/](https://services.gradle.org/distributions/)

我们选择想要安装的发布版本，**gradle-x.x-bin.zip** 是需要下载的安装发布版，**gradle-x.x-src.zip** 是源码，**gradle-x.x-all.zip** 则是下载全部的文件。这里我选择的是`gradle-6.6.1-bin.zip`。

### 配置 Gradle 环境变量

直接把gradle 的安装目录bin地址贴到环境变量的Path当中。

```
E:\adongstack\mysoft\gradle-6.6.1\bin
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212203101.png)

然后是在`cmd`当中验证即可：

```sh
C:\Users\adong>gradle -v

Welcome to Gradle 6.6.1!

Here are the highlights of this release:
 - Experimental build configuration caching
 - Built-in conventions for handling credentials
 - Java compilation supports --release flag

For more details see https://docs.gradle.org/6.6.1/release-notes.html


------------------------------------------------------------
Gradle 6.6.1
------------------------------------------------------------

Build time:   2020-08-25 16:29:12 UTC
Revision:     f2d1fb54a951d8b11d25748e4711bec8d128d7e3

Kotlin:       1.3.72
Groovy:       2.5.12
Ant:          Apache Ant(TM) version 1.10.8 compiled on May 10 2020
JVM:          1.8.0_333 (Oracle Corporation 25.333-b02)
OS:           Windows 11 10.0 amd64
```

## 安装 Zookeeper 3.4.14

个人建议安装到虚拟机的Linux系统，安装的操作如下：

```sh
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

国内的访问速度比较慢，这里直接上外网拖了一个安装包放到网盘，读者根据网盘下载即可：

> 链接：https://pan.baidu.com/s/1R6zz5pffuwV8D7B-N6PV7Q?pwd=wopn 
	提取码：wopn 

###  创建 data 文件夹

在Zookeeper 3.4.14的目录当中准备`data`文件夹。

```sh
[zxd@localhost zookeeper-3.4.14]$ mkdir data
mkdir: cannot create directory ‘data’: Permission denied
[zxd@localhost zookeeper-3.4.14]$ sudo mkdir data
[sudo] password for zxd: 
[zxd@localhost zookeeper-3.4.14]$ ls
bin  conf  data  dist-maven
```

### 修改 zoo.cfg 中的 data 属性

我们将Zookeeper的数据目录执行刚刚新建的data文件夹。

```sh
cd conf
vim zoo_sample.cfg zoo.cfg
```

`vim zoo_sample.cfg zoo.cfg`，我们修改相关的配置，默认目录在`/tmp/zookeeper`，这里个人改为`/opt/zookeeper/zookeeper-3.4.14/data`

```sh
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
#dataDir=/tmp/zookeeper
dataDir=/opt/zookeeper/zookeeper-3.4.14/data
# the port at which the clients will connect
clientPort=2181
```

修改之后我们使用`-x` 应用修改。

## 启动Zookeeper

编辑完成之后，进入到安装目录的bin目录当中启动Zookeeper。

```sh
[zxd@localhost zookeeper-3.4.14]$ cd bin/
[zxd@localhost bin]$ sudo ./zkServer.sh start 
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/zookeeper-3.4.14/bin/../conf/zoo.cfg
grep: /opt/zookeeper/zookeeper-3.4.14/bin/../conf/zoo.cfg: No such file or directory
mkdir: cannot create directory ‘’: No such file or directory
Starting zookeeper ... STARTED
```

发现没有启动成功，根据提示我们需要zoo.cfg的配置，这部分为Zk的内容，这里就直接跳过了。

## Kafka 2.7.2 源码下载

下载`kafka-2.7.2-src.tgz`包之后，解压到对应的目录，接下来需要导入到Idea当中。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212212305.png)

## 导入Idea

下载源码之后在Idea当中导入Kafka项目。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212213914.png)



![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212213934.png)

导入项目之后，我们进入到Setting， 设置Gradle的Home地址。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212214041.png)

这里把Gradle的Home地址设置为自己本机安装的，不使用Iead自带的。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212214002.png)

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212214219.png)

### 修改 build.gradle

接下来还不能导 jar 包，需要把镜像文件下载服务器更换为国内的私服，否则会相当慢，直接导致 "time out" 报错。进入 kafka 源码包，修改 **build.gradle** 文件，在原来配置上，添加阿里的私服配置。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212214419.png)

下面的内容复制到 **build.gradle** 文件的对应位置：

```xml
maven {
	url 'http://maven.aliyun.com/nexus/content/groups/public/'
}
maven {
	url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
}
```

> 文件当中查找`buildscript`可以快速定位。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212214501.png)

注意最后需要 **调整一下位置**，优先从阿里云的库获取：

```sh
buildscript {  
  repositories {  
    maven {  
      url 'http://maven.aliyun.com/nexus/content/groups/public/'  
    }  
    maven {  
      url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'  
    }  
    mavenCentral()  
    jcenter()  
    maven {  
      url "https://plugins.gradle.org/m2/"  
    }  
  
  }
  ......
```

接着还需要复制到`allproject`当中：

```sh
maven {
	url 'http://maven.aliyun.com/nexus/content/groups/public/'
}
maven {
	url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
}
```

复制到的位置大概如下：

```gradle
allprojects {  
  
  repositories {  
    maven {  
      url 'http://maven.aliyun.com/nexus/content/groups/public/'  
    }  
    maven {  
      url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'  
    }  
    mavenCentral()  
  
  }
```

替换之后耐心等待依赖下载完成：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212221849.png)

在依赖安装完成之后，我们可以点一下Idea顶部的小锤子，最后的提示编译成功：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212223300.png)

### 安装 Idea scala 插件

Idea导入源码之后的`.scala`文件是没有语法提示的，我们安装插件之后可以正常阅读scala的源码：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212225657.png)

插件安装完成之后需要重启。访问`kafka.scala`，如果相关的代码出现颜色和语法提示说明插件安装成功：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212225903.png)

## 选择 scala jdk

如果读者是第一次搭建Kafka和使用scala，大概率会出现下面的提示：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212230210.png)

根据提示设置`scala jdk`的安装目录：

```
C:\Program Files (x86)\scala\bin
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212230345.png)

设置完成之后一切就大功告成了。

## 编译和构建 Kafka 源码

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230212223700.png)


这里个人不太熟悉Idea对于gradle的使用，最后用了gradle的命令进行构建，构建之后出现下面的内容表示正确：

`gradle`命令是用来下载和更新 Gradle 套件（Gradle Wrapper）的，`gradle jar`是用 Gradle 套件构建 Kafka 工程，生成 Jar 文件。

下面是使用`gradle` 命令的结果：

```sh
adong@Adong-pc MINGW64 /e/adongstack/project/selfUp/kafka-2.7/kafka-2.7.2-src
$ gradle
Starting a Gradle Daemon, 1 busy Daemon could not be reused, use --status for details

> Configure project :
Building project 'core' with Scala version 2.13.3
Building project 'streams-scala' with Scala version 2.13.3

> Task :help

Welcome to Gradle 6.6.1.

To run a build, run gradle <task> ...

To see a list of available tasks, run gradle tasks

To see a list of command-line options, run gradle --help

To see more detail about a task, run gradle help --task <task>

For troubleshooting, visit https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.6.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 44s
1 actionable task: 1 executed

```

下面使用`gradle jar`的编译结果：

```sh
adong@Adong-pc MINGW64 /e/adongstack/project/selfUp/kafka-2.7/kafka-2.7.2-src
$ gradle jar

> Configure project :
Building project 'core' with Scala version 2.13.3
Building project 'streams-scala' with Scala version 2.13.3

> Task :clients:processMessages
MessageGenerator: processed 121 Kafka message JSON files(s).

> Task :raft:processMessages
MessageGenerator: processed 1 Kafka message JSON files(s).
<=------------> 12% EXECUTING [2m 6s]

> Task :clients:processTestMessages
MessageGenerator: processed 2 Kafka message JSON files(s).
<==-----------> 21% EXECUTING [3m 10s]

> Task :streams:processMessages
MessageGenerator: processed 1 Kafka message JSON files(s).

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.6.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 3m 58s
87 actionable tasks: 87 executed

```

最后使用下面的命令：

```sh
gradle clean build -x test
```

但是这种命令构建过程会出现一些报错，可以把报错所需要的依赖jar包放到下面：

```
E:\adongstack\project\selfUp\kafka-2.7\kafka-2.7.2-src\gradle\wrapper
```

要构建整个 Kafka 工程并打包出一个可运行的二进制环境命令如下

```gradle
gradle clean releaseTarGz
```

### 其他构建命令

还有其他gradle的构建命令：

```
构建 jar包并运行
./gradle jar

构建项目，看你是idea工具还是eclipse
./gradle idea
./gradle eclipse

构建源码包
./gradle srcJar

构建javadoc文档
./gradle aggregatedJavadoc

清理并构建
./gradle clean
```

### 单元测试

Kafka当中同样存在很多单元测试，下面是一些核心模块的单元测试命令。

core 模块单元测试：

```gradle
gradle core:test
```

client 模块单元测试：

```gradle
gradle clients:test
```

模块当中的某个子模块单元测试：

```gradle
gradle connect:[submodule]:test
```

> Connect 工程下细分了多个子模块，比如 api、runtime 等，需要显式地指定要 测试的子模块名才能进行测试。

stream 模块单元测试。

```gradle
gradle connect:[submodule]:test
```

个人实际使用之后会发现有部分报错信息：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230213070600.png)

除了上面的整个模块的单元测试，如果只想要测试某个模块下的某个类，可以使用下面的方法：

单独对某一个具体的测试用例进行测试，比如测试 **LogTest** 类：

```gradle
gradle core:test --tests kafka.log.LogTest
```

个人的执行结果如下：

```gradle
$ gradle core:test --tests kafka.log.LogTest

> Configure project :
Building project 'core' with Scala version 2.13.3
Building project 'streams-scala' with Scala version 2.13.3

> Task :core:test
kafka.log.LogTest.testLogDelete failed, log available in E:\adongstack\project\selfUp\kafka-2.7\kafka-2.7.2-src\core\build\reports\testOutput\kafka.log.LogTest.testLogDelete.test.stdout

kafka.log.LogTest > testLogDelete FAILED
    org.apache.kafka.common.errors.KafkaStorageException: Error while deleting log for kafka-936488 in dir C:\Users\adong\AppData\Local\Temp\kafka-7162272829353105740

        Caused by:
        java.nio.file.FileSystemException: C:\Users\adong\AppData\Local\Temp\kafka-7162272829353105740\kafka-936488\00000000000000000000.timeindex -> C:\Users\adong\AppData\Local\Temp\kafka-7162272829353105740\kafka-936488\00000000000000000000.timeindex.deleted: ▒▒һ▒▒▒▒▒▒▒▒▒▒ʹ▒ô▒▒ļ▒▒▒▒▒▒▒▒޷▒▒▒▒ʡ▒

```

# Kafka目录结构

**bin 目录**：保存 Kafka 工具行脚本，我们熟知的 kafka-server-start 和 kafka-consoleproducer 等脚本都存放在这。

**clients 目录**：保存 Kafka 客户端代码，比如生产者和消费者的代码都在该目录下。 

**config 目录**：保存 Kafka 的配置文件，其中比较重要的配置文件是 server.properties。 

**connect 目录**：保存 Connect 组件的源代码。Kafka Connect 组件是用来实现 Kafka 与外部系统之间的**实时数据传输**的。 

**core 目录**：保存 Broker 端代码。Kafka 服务器端代码全部保存在该目录下。

**streams 目录**：保存 Streams 组件的源代码。Kafka Streams 是实现 Kafka **实时流处理**的组件。

**server 目录**：顾名思义，它是 Kafka 的服务器端主代码，里面的类非常多，很多关键的 Kafka 组件都存放在这里，比如状态机、Purgatory 延时机制等。

> Kafka Streams is a client library for building applications and microservices, where the input and output data are stored in Kafka clusters.

提供一个基于 Kafka 的流式处理类库，直接提供具体的类给开发者调用，整个应用的运行方式主要由开发者控制，方便使用和调试。
    
Kafka Streams 是一个用来构建流处理程序的库，特别是其输入是一个 Kafka topic，输出是另一个 Kafka topic 的程序（或者是调用外部服务，或者是更新数据库，或者其它）。它使得你以一种分布式以及容错的方式来做这件事情。

**coordinator 包**：保存了消费者端的 GroupCoordinator 代码和用于事务的 TransactionCoordinator 代码。对 coordinator 包进行分析，特别是对消费者端的 GroupCoordinator 代码进行分析，是我们弄明白 Broker 端协调者组件设计原理的关 键。

# 其他目录结构

除了上面的核心目录之外，我们还可以在项目根路径看到一些其他模块：

**network 包**：封装了 Kafka 服务器端网络层的代码，特别是 SocketServer.scala 这个文件，是 Kafka 实现 Reactor 模式的具体操作类，非常值得一读。

**consumer 包**：后面会丢弃该包，用 clients 包下 consumer 相关类代替。

**tools 包**：工具类。

**log 包**：保存了 Kafka 最核心的日志结构代码，包括日志、日志段、索引文件等, 另外，该包下还封装了 Log Compaction 的实现机制，是非常重要的源码包。

**checkstyle 目录**：代码规范，自动化检测。
    
> Checkstyle 是什么，类似于代码规范的自动化检测插件，国内最为经典的是阿里巴巴的规约插件，而最出名的检查插件为 `google style guide`，Checkstyle 就是以类似这种风格开发出的一个自动化插件，来辅助判断代码格式是否满足规范。

该目录下的文件定义了工程代码格式的规范，我们可以在 build.gradle 中看到相关 checkstyle 的配置和自动化代码格式化配置：
    
**checkstyle 配置（build.gradle）**

```scala
def checkstyleConfigProperties(configFileName) {  
  [importControlFile: "$rootDir/checkstyle/$configFileName",  
   suppressionsFile: "$rootDir/checkstyle/suppressions.xml",  
   headerFile: "$rootDir/checkstyle/java.header"]  
}

checkstyle {  
  configProperties = checkstyleConfigProperties("import-control-core.xml")  
}
```

**connect 目录**：保存 Connect 组件的源代码。 Kafka Connect 组件是用来实现 Kafka 与外部系统之间的实时数据传输的。

**docs 目录**：Kafka 设计文档以及组件相关结构图。

**examples 目录**：Kafka 样例相关目录。

**gradle 目录**：gradle 的脚本和依赖包定义等相关文件。

**jmh-benchmarks 目录**：Kafka 代码微基准测试相关类。

> JMH，即 Java Microbenchmark Harness，是**专门用于代码微基准测试的工具套件**。Micro Benchmark 是什么意思？简单的来说就是**基于方法层面的基准测试**，精度可以达到微秒级。当你定位到热点方法，希望进一步优化方法性能的时候，就可以使用 JMH 对优化的结果进行量化的分析。

JMH 比较典型的应用场景有：
    
 -  准确的知道某个方法需要执行多长时间，以及执行时间和输入之间的相关性；
 -   对比接口不同实现在给定条件下的吞吐量，找到最优实现。

**kafka-logs 目录**：server.properties 文件中配置 log.dirs 生成的目录。

**log4j-appender 目录**：
    
> A log4j appender that produces log messages to Kafka

这个目录里面就一个 KafkaLog4jAppender 类。
    
**raft 目录**：raft 一致性协议相关模块。

**tests 目录**：此目录的内容介绍如何进行 Kafka 系统集成和性能测试。

**tools 目录**：工具类模块。

**vagrant 目录**：介绍如何在 Vagrant 虚拟环境中运行 Kafka，提供了相关的脚本文件和说明文档。
    
> Vagrant 是一个基于 Ruby 的工具，用于创建和部署虚拟化开发环境。它使用 Oracle 的开源 VirtualBox 虚拟化系统，使用 Chef 创建自动化虚拟环境。

# 参考

[https://segmentfault.com/a/1190000040790524](https://segmentfault.com/a/1190000040790524)

