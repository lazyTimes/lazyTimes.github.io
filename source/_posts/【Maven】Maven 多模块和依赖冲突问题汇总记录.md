---
title: 【Maven】Maven 多模块和依赖冲突问题汇总记录
subtitle: 多模块和依赖冲突问题汇总记录
url_suffix: mavenmodules
author: lazytime
tags:
  - Maven
categories:
  - Maven
keywords: maven,多模块
description: maven多模块如何在idea中创建，以及maven常见的依赖问题介绍和处理方法
copyright: true
date: 2020-11-22 19:46:50
---

# 【Maven】Maven 多模块和依赖冲突问题汇总记录

# 目录

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122191750.png)


[TOC]

# 前言：

今天学习和总结了一下maven的相关知识点，发现一些比较基础的东西居然也会忘记，这里对于一些日常工作中可能会遇到的问题进行了一下汇总。



# idea怎么创建maven多module的项目

## 首先了解上面是多module？

一句话概括就是：一个父模块作为版本控制多个子模块，子模块负责接入到父模块当中作为整个项目的过程。

## 多Module管理项目的几种方式：

1. 按照单模块拆分为多个子模块，比如将MVC三层架构拆分为 xxx-service，xxx-dao，xxx-model，不过这种方式个人感觉比较二，目前以业务模块拆分比较多，迁移到微服务比如用springcloude或者dubbo 的时候非常好用。
2. 按照业务模块拆分，这种模式使用的比较多，也比较多见。

<!-- more -->

## 创建一个多module项目(idea2019.3.3版本)

### 创建一个父pom项目：

1. 打开idea，选择`create new project`

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122114914.png)

2. 选择maven项目，同时不选任何的预加载设置

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122115055.png)

3. 父pom配置如下：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122115139.png)

4. 删除`src` 目录

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122115235.png)

### 创建子模块，引入到父pom里面

1. 同样右击项目工程，选择`new module`，然后选择`maven`，这时候会出现父模块以及对应的子模块名称

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122121402.png)

2. 此时在父模块里面发现引入了子模块的内容

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122121551.png)

## 子父模块的区别：

### 父pom.xml文件内容：

```xml
<groupId>org.zxd</groupId>
<artifactId>taglib</artifactId>
<packaging>pom</packaging>
<version>1.0.0</version>

<modules>
    <module>taglib-core</module>
</modules>
```

分为两个部分，一个部分是父pom的声明，包含gourpId，artifactId，打包方式**必须是pom**，因为使用了**聚合模型**，同时在父pom里面指定版本号，子模块不填写version会默认使用父pom的version号

```xml
<modules>
    <module>taglib-core</module>
</modules>
```

上面表示当前引入的子模块内容

### 子pom.xml文件内容：

```xml
<!-- 引用自父pom -->
<parent>
    <artifactId>taglib</artifactId>
    <groupId>org.zxd</groupId>
    <version>1.0.0</version>
</parent>
<!-- 打包方式为jar包 -->
<packaging>jar</packaging>
<modelVersion>4.0.0</modelVersion>

<artifactId>taglib-core</artifactId>
<version>1.0.0</version>
```

## 子模块之间进行互相的依赖

在下面的pom中可以在任意的子模块引入对应的父模块依赖

注意由于`<parent>`这个标签会递归继承，所以要注意子依赖不要和依赖引入不同版本的依赖，这样容易造成冲突

```xml
<dependency>
    <groupId>org.zxd</groupId>
    <artifactId>taglib-core</artifactId>
    <version>1.0.0</version>
    <!-- 这里需要注释掉编译的作用域 -->
    <!--<scope>compile</scope>-->
</dependency>
```



## 将上面的项目改造为spring-boot多模块项目：

### 改造父pom文件：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

parent指向`springboot-started`

子模块只需要引入父pom的内容



### Spring boot maven plugin问题

在打包spring boot项目时，需要使用如下插件：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

如果在用maven构建多模块项目时，不要将此插件放到parent pom中，否则如果有sub module不是spring boot应用，在打包时就会出错。**只将该插件添加到是spring boot项目的子模块**。



# MAVEN依赖冲突问题：

## 依赖的传递原则：

1. 最短路径原则
2. 最先声明原则

## maven的依赖引入策略

### 最短路径原则：

我有下面两个依赖jar包，A和B，他们都引入了C这个依赖，这时候如果有如下的引用

A -> C（3.3）

B -> A（3.3）

B  -> C（3.4）

此时如果把B打包，得到版本号是3.4，但是如果B去掉C的依赖，那就是走A->C的传递依赖，很简单

验证：

1. 我假设我有一个web包引入了common-lang3，版本是3.4的版本

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.4</version>
</dependency>
```

2. 此时又引入了一个公用包，里面也有这个引用：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.3</version>
</dependency>
```

3. 如果此时在本地引用3.4版本，那就是3.4的版本，否则就死3.3的版本，不管声明顺序谁先谁后



### 最先声明原则：

如果两个jar包的寻址路径一致，那么谁先优先声明，先引入谁

验证：

下面两个依赖分配对应两个module，引入模块的这个模块暂定为 **C** 模块。

```xml
<!-- 引入core包内容 -->
<dependency>
    <groupId>org.zxd</groupId>
    <artifactId>taglib-core</artifactId>
    <version>1.0.0</version>
    <!--            <scope>compile</scope>-->
</dependency>
<!-- 引入db包的内容 -->
<dependency>
    <groupId>org.zxd</groupId>
    <artifactId>taglib-db</artifactId>
    <version>1.0.0</version>
</dependency>
```

此时 `taglib-core`  中的依赖版本如下，暂定为 **A** 模块：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>2.5.0</version>
</dependency>
```

而`taglib-db` 中的依赖版本如下，暂定为 **B** 模块：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.0.1</version>
</dependency>
```

此时将整个web项目打包，可以看到web项目里面的版本如下：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122181117.png)

可以很明显的看到如果引入关系是这样的：

C -> A,B

A -> curator-client 2.5

B -> curator-client 4.1

这样的链路最终打包出来的效果是  C -> A -> curator-client 2.5

这样也就造成了很多情况下我们**编译**和**运行**时完全没有问题的，**甚至有可能打包都是正常**的，但是到最后运行的时候突然报错，要谨防这种依赖版本的问题，好在一般公司的项目都有经理负责控制版本依赖，这种错误算是低级错误，但是在如今框架满天飞的时代，依赖管理的版本控制问题依然需要注意！！！



## 如何解决依赖冲突的问题

### 锁定版本法

一般情况下我们会在父pom文件里面管理，可以使用`<dependencyManagement>`这个这个标签来管理所有子模块的版本依赖，子模块如果指定自己的版本，这里发现打出来的包依然是父pom指定的版本，版本管理使用如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-client</artifactId>
            <version>4.1.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**锁定版本法可以打破2个依赖传递的原则，优先级为最高**

> 版本锁定可以排除一些exclude标签，不同模块用不同版本的jar包本身也不符合规范，所以这种方式较为稳妥



## 什么情况下会出现Jar包冲突问题

**只有高版本Jar包不向下兼容，或者新增了某些低版本没有的API才有可能导致这样的问题**



## 如何查找和发现jar包冲突？

### 1. 利用idea的maven视图工具

直接使用一个图说明一下：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122185753.png)

可以通过这个工具查看依赖在哪个模块重复引用，同时如果有冲突会显示红线，这个视图非常的直观，可以帮助依赖管理人员去处理依赖重复引用或者引用版本不一致的问题，进行`<exclude>`操作

### 2. Idea `Maven Helper` 插件

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122190207.png)

如何使用？

安装完成之后，随便找一个pom.xml文件，按照如下的图例提示进行操作，对于冲突的内容，右击`exclude`就可以排除依赖：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122190726.png)

排除完依赖之后，点击左上角的`Refresh UI` 刷新一下UI的界面：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122190841.png)



### 3. maven命令工具：

`mvn dependency:tree -Dverbose`，有时候如果我们没有idea的情况下，可以使用这个命令来处理，执行的结果参考如下：

> PS: 此处一定不要省略`-Dverbose`参数，要不然是**不会**显示被**忽略**的包的

```
or:curator-client:jar:4.1.0:compile
[INFO] |  |  +- org.apache.zookeeper:zookeeper:jar:3.5.4-beta:compile
[INFO] |  |  |  +- commons-cli:commons-cli:jar:1.2:compile
[INFO] |  |  |  +- log4j:log4j:jar:1.2.17:compile
[INFO] |  |  |  +- org.apache.yetus:audience-annotations:jar:0.5.0:compile
[INFO] |  |  |  \- io.netty:netty:jar:3.10.6.Final:compile
[INFO] |  |  +- com.google.guava:guava:jar:20.0:compile
[INFO] |  |  \- org.slf4j:slf4j-api:jar:1.7.30:compile
[INFO] |  +- commons-codec:commons-codec:jar:1.15:compile
[INFO] |  +- commons-collections:commons-collections:jar:3.2.2:compile
[INFO] |  +- commons-beanutils:commons-beanutils:jar:1.9.4:compile
[INFO] |  +- commons-configuration:commons-configuration:jar:1.10:compile
[INFO] |  |  \- commons-lang:commons-lang:jar:2.6:compile

```

总体上来说还是比较直观的，非常方便和好用。

## 如何写一个干净依赖关系的`POM`文件

- 尽量在父POM中定义`<dependencyManagement>`，来进行本项目一些依赖版本的管理，这样可以从很大程度上解决一定的冲突
- 最少依赖jar包原则
- 使用`mvn dependency:analyze-only`命令用于检测那些声明了但是没被使用的依赖，如有有一些是你自己声明的，那尽量去掉
- 使用`mvn dependency:analyze-duplicate`命令用来分析重复定义的依赖，清理那些重复定义的依赖



### `dependency:analyze-only` 命令

在idea - `Terminal`里面，可以看到对应的依赖被下载

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122182537.png)

执行完之后我的运行结果如下，这里报错的原因是打包时候默认去阿里云仓库寻找依赖，这里需要配置一下：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201122183942.png)

```shell
[ERROR] Failed to execute goal on project taglib-web: Could not resolve dependencies for project org.zxd:taglib-web:war:1.0.0: The following artifacts could not be resolved: org.zxd:
taglib-core:jar:1.0.0, org.zxd:taglib-db:jar:1.0.0: Failure to find org.zxd:taglib-core:jar:1.0.0 in http://maven.aliyun.com/nexus/content/repositories/central/ was cached in the loc
al repository, resolution will not be reattempted until the update interval of alimaven has elapsed or updates are forced -> [Help 1]
```

大致意思就是说再阿里云仓库找不到对应的依赖引入。

解决方式如下：

找到maven的安装路径下的`apache-maven-3.6.3\conf`下面的`setting.xml`，找到如下配置：

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
<!-- 这里配置本地仓库的位置 -->
  <localRepository>D:\soft\apache-maven-3.6.3\rep</localRepository>
```

此时重新执行一下：`dependency:analyze-only` 命令

```
[INFO] --- maven-dependency-plugin:3.1.2:analyze-only (default-cli) @ taglib-web ---
[WARNING] Unused declared dependencies found:
[WARNING]    org.zxd:taglib-core:jar:1.0.0:compile
[WARNING]    org.apache.commons:commons-lang3:jar:3.4:compile
[WARNING]    org.springframework.boot:spring-boot-starter-web:jar:2.4.0:compile
[WARNING]    org.springframework.boot:spring-boot-starter-test:jar:2.4.0:test
[WARNING]    org.neo4j.driver:neo4j-java-driver:jar:1.5.0:compile
[WARNING]    commons-codec:commons-codec:jar:1.10:compile
[WARNING]    commons-collections:commons-collections:jar:3.2.2:compile
[WARNING]    commons-beanutils:commons-beanutils:jar:1.9.4:compile
[WARNING]    commons-configuration:commons-configuration:jar:1.10:compile
[WARNING]    commons-fileupload:commons-fileupload:jar:1.3:compile
[WARNING]    commons-logging:commons-logging:jar:1.2:compile
[WARNING]    org.apache.httpcomponents:httpclient:jar:4.4.1:compile
[WARNING]    org.apache.poi:poi-ooxml:jar:3.17:compile
[WARNING]    org.mybatis:mybatis:jar:3.4.0:compile
[WARNING]    org.mybatis:mybatis-spring:jar:1.3.0:compile
[WARNING]    com.github.pagehelper:pagehelper:jar:5.1.2:compile
```

### `mvn dependency:analyze-duplicate` 命令

```
[INFO] No duplicate dependencies found in <dependencies/> or in <dependencyManagement/>
```

如果没有其他信息，代表没有重复依赖的引入