---
title: Maven私服Nexus搭建教程
subtitle: 这个人很懒，不想写副标题
author: lazytime
url_suffix: mavennexus
tags:
  - Maven
categories:
  - JAVA
keywords: Maven,java
description: 使用maven搭建私服
copyright: true
date: 2020-07-26 11:16:10
---

# Nexus 私服搭建基本流程

## Nexus 是什么？

用于实现Maven构建私人仓库的一个工具。

可以使用nexus 统一管理我们到依赖，不再需要再从中央仓库下载对应的jar包

<!-- more -->

## 什么是Nexus 私服？

私服是在局域网的一种特殊的远程仓库，目的是代理远程仓库及部署第三方构件。有了私服之后，当 Maven 需要下载jar包时，先请求私服，私服上如果存在则下载到本地仓库。否则，私服直接请求外部的远程仓库，将jar包下载到私服，再提供给本地仓库下载。

## maven的依赖管理

**简介：依赖是maven最为用户熟知的特性之一，单个项目的依赖管理并不难，但是要管理几个或者几十个模块的时，那这个依赖应该怎么管理**

- 依赖的传递性

  - 传递性依赖是在maven2中添加的新特征，这个特征的作用就是你不需要考虑你依赖的库文件所需要依赖的库文件，能够将依赖模块的依赖自动的引入。

- 依赖的作用范围

  - compile

    - 这是默认范围，编译依赖对项目所有的classpath都可用。此外，编译依赖会传递到依赖的项目

  - provided

    - 表示该依赖项将由JDK或者运行容器在运行时提供，也就是说由Maven提供的该依赖项我们只有在编译和测试时才会用到，而在运行时将由JDK或者运行容器提供。

  - runtime

    - 表明编译时不需要依赖，而只在运行时依赖

  - test

    - 只在编译测试代码和运行测试的时候需要，应用的正常运行不需要此类依赖。

  - system

    - 系统范围与provided类似，不过你必须显式指定一个本地系统路径的JAR，此类依赖应该一直有效，Maven也不会去仓库中寻找它。

      ```
      <project>  
        ...  
        <dependencies>  
          <dependency>  
            <groupId>sun.jdk</groupId>  
            <artifactId>tools</artifactId>  
            <version>1.5.0</version>  
            <scope>system</scope>  
            <systemPath>${java.home}/../lib/tools.jar</systemPath>  
          </dependency>  
        </dependencies>  
        ...  
      </project> 
      ```

- import

  - 范围只适用于<dependencyManagement>部分。表明指定的POM必须使用<dependencyManagement>部分的依赖。因为依赖已经被替换，所以使用import范围的依赖并不影响依赖传递。

- 依赖的两大原则

  - 路径近者优先

    ```
    A > B > C-1.0
    A > C-2.0
    ```

  - 第一声明优先

    ```
    A > B > D-1.0
    A > C > D-2.0
    ```

- 依赖的管理

  - 依赖排除
    - 任何可传递的依赖都可以通过 "exclusion" 元素被排除在外。举例说明，A 依赖 B， B 依赖 C，因此 A 可以标记 C 为 "被排除的"
  - 依赖可选
    - 任何可传递的依赖可以被标记为可选的，通过使用 "optional" 元素。例如：A 依赖 B， B 依赖 C。因此，B 可以标记 C 为可选的， 这样 A 就可以不再使用 C。

## 如何解决jar包冲突

**简介：当出现jar包冲突时，我们应该如何快速定位和处理jar包冲突问题**

- 命令: `mvn dependency:tree -Dverbose`

## Nexus私服的秘密花园

简介：介绍nexus服务器预置的仓库

- 类型介绍
  - hosted：是本地仓库，用户可以把自己的一些jar包，发布到hosted中，比如公司的第二方库
  - proxy，代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。不允许用户自己上传jar包，只能从中央仓库下载
  - group，仓库组，用来合并多个hosted/proxy仓库，当你的项目希望在多个repository使用资源时就不需要多次引用了，只需要引用一个group即可
  - virtual，虚拟仓库基本废弃了。
- 预置仓库
  - Central：该仓库代理Maven中央仓库，其策略为Release，因此只会下载和缓存中央仓库中的发布版本构件。
  - Releases：这是一个策略为Release的宿主类型仓库，用来部署正式发布版本构件
  - Snapshots：这是一个策略为Snapshot的宿主类型仓库，用来部署开发版本构件。
  - 3rd party：这是一个策略为Release的宿主类型仓库，用来部署无法从maven中央仓库获得的第三方发布版本构件，比如IBM或者oracle的一些jar包（比如classe12.jar），由于受到商业版权的限制，不允许在中央仓库出现，如果想让这些包在私服上进行管理，就需要第三方的仓库。
  - Public Repositories：一个组合仓库

## Nexus 下载地址

[Nexus 官方网站](https://www.sonatype.com/nexus-repository-oss)

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/jiuhe/20200318141611.png?ynotemdtimestamp=1595729096959)

## 快速入门

### 搭建Maven Nexus

1. 进入 [Nexus 官方网站](https://www.sonatype.com/nexus-repository-oss)
2. 在首页选择 `“GET PREPOSITORY OSS”`
3. 页面拉倒最下面，点击`OSS2`

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200327201355.png?ynotemdtimestamp=1595729096959)

1. 由于Maven Nexus 是收费的，这里我们需要使用免费的版本

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200327201558.png?ynotemdtimestamp=1595729096959)

1. 下载之后，会得到如下内容

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200328234944.png?ynotemdtimestamp=1595729096959)

1. 此时调出 `cmd`, 进入目录`D:\java\Nexus\nexus-latest-bundle\nexus-2.14.15-01\bin`
2. 运行 `nexus.bat start`，可以发现会`拒绝访问`

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200328235057.png?ynotemdtimestamp=1595729096959)

1. 使用管理员模式运行
   - 运行 `nexus.bat install` 安装
   - `nexus.bat start` 启动服务

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329170837.png?ynotemdtimestamp=1595729096959)

1. 访问 http://localhost:8081/nexus进入到nexus 页面

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329002328.png?ynotemdtimestamp=1595729096959)

1. 进入主页

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329164131.png?ynotemdtimestamp=1595729096959)

### 建立第一个个人仓库

1. 完成上一节内容之后，进入主页面，点击右上角`Log In` 登陆系统

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329165902.png?ynotemdtimestamp=1595729096959)

1. Nexus 默认到用户名称和密码为 `admin` 以及 `admin123`，输入账户名和密码之后登陆系统

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329170010.png?ynotemdtimestamp=1595729096959)

1. 登陆之后会发现界面发生了改变，因为此时我们拥有了对应到权限

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329170040.png?ynotemdtimestamp=1595729096959)

1. 点击 `add`，选择 `Hosted Repository` 改含义会在后面进行解释

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329170809.png?ynotemdtimestamp=1595729096959)

1. 如图所示，基本上只需要填写很少到内容，就可以完成一个仓库到创建，点击下方到`save`按钮即可创建一个仓库

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329170521.png?ynotemdtimestamp=1595729096959)

1. 查看配置,可以看到改仓库被放到了一个默认的地址里面

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200329171028.png?ynotemdtimestamp=1595729096959)

### 将Jar包项目加入到个人仓库

这里以一个自己的项目作为测试

Maven依赖如下:

```
<dependency>
    <groupId>org.smart4j</groupId>
    <artifactId>smart-framwork</artifactId>
    <version>1.0.0</version>
</dependency>
```

我们需要先登录Maven Nexus，具备管理员的权限，然后点击`3rd party`(三方依赖)，截图内容如下

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405114219.png?ynotemdtimestamp=1595729096959)

依照截图填入如下的依赖

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405114639.png?ynotemdtimestamp=1595729096959)

> GAV Definition: 定义GAV
>
> Auto Guess：自动猜测
>
> Group：同Maven 到 group定义
>
> Artifact：同Maven 到 Artifact定义
>
> Version：版本号
>
> Packaging：打包方式
>
> 下方需要上传依赖对应到jar包

上传之后需要添加到maven Nexus

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405115034.png?ynotemdtimestamp=1595729096959)

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405115136.png?ynotemdtimestamp=1595729096959)

等待上传，上传成功之后会给予对应到提示信息

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405115216.png?ynotemdtimestamp=1595729096959)

上传完成之后，我们可以点击`Browse Index`看到自己之前上传的依赖

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405115356.png?ynotemdtimestamp=1595729096959)

> 默认情况下我们到公共访问地址如下：
>
> http://localhost:8081/nexus/content/groups/public 公共仓库的访问地址
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405115744.png?ynotemdtimestamp=1595729096959)

### 将个人仓库的jar包添加到项目当中

把jar包添加到nexus之后，我们可以尝试让nexus来管理我们到项目依赖了

接下来介绍如何使用maven nexus 来关联我们到依赖

1. 我们需要建立一个存在pom文件的项目，把对应到pom依赖引入

> 注意：事先查看nexus公有仓库是否存在依赖

1. 在Maven 的 `conf` 下面增加配置，以个人为例进入`D:\java\apach\apache-maven-3.6.0\conf`，修改`setting.xml`,添加如下内容

```
<!-- 添加 nexus 访问权限 -->
<servers>	
	<server>
	  <id>xdclass</id>
	  <username>admin</username>
	  <password>admin123</password>	
	</server>

  </servers>


<mirrors>
	<!-- 自定义 nexus -->
	 <mirror>
        <id>xdclass</id>
        <mirrorOf>nexus,central</mirrorOf>
        <url>http://localhost:8081/nexus/content/groups/public/</url>
		<name>local nexus</name>  
    </mirror>
	<!-- 阿里云配置 -->
	<mirror>  
      <id>alimaven</id>  
      <name>aliyun maven</name>  
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
      <mirrorOf>central</mirrorOf>          
	</mirror>  
	
</mirrors>
<profiles>
	<profile>
        <id>xd</id>
        <repositories>
            <repository>
                <id>local-nexus</id>
                <!-- 本地仓库路径 -->
                <url>http://localhost:8081/nexus/content/groups/public/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <!-- 开启快照 -->
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
            </repository>
        </repositories>
    </profile>
</profiles>

<!-- 需要 开启 Profile 配置 -->
<activeProfiles>
    <activeProfile>xd</activeProfile>
</activeProfiles>
```

1. 在IDEA中重新导入依赖

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405142135.png?ynotemdtimestamp=1595729096959)

### 将个人到Maven项目发布到Nexus管理

本节介绍如何把自己的项目发布到nexus ，这里单独开了一个仓库进行配置

在进行本节内容之前，查看Maven 到 `setting.xml`内容

```
<!-- 用于发布正式版本 -->
<server>
    <id>public</id>
    <username>admin</username>
    <password>admin123</password>	
</server>
<!-- 测试 -->
<server>
    <id>lazytime</id>
    <username>admin</username>
    <password>admin123</password>	
</server>
```

1. 在需要发布源代码到mavne项目`pom.xml`添加如下内容

```
 <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>

            </plugins>
        </pluginManagement>
    </build>


    <distributionManagement>
        <repository>
            <!-- 此id 必须对应setting 里面到 server id 否则会没有权限无法部署 -->
            <id>public</id>
            <url>http://localhost:8081/nexus/content/repositories/lazytime</url>
        </repository>

    </distributionManagement>
```

1. 以IDEA为例，`deloyer` 项目到 `maven Nexus`

运行`Maven Deloy`发布项目到nexus

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405162751.png?ynotemdtimestamp=1595729096959)

1. 查看`Maven Nexus` ，查看发布的项目内容

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405162923.png?ynotemdtimestamp=1595729096959)

> 如果不能发布，请查看仓库是否允许重新部署:
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405164619.png?ynotemdtimestamp=1595729096959)

### 使用Snapshots 快照快捷管理项目迭代

使用上一节办法存在很大到缺点：

1. 每次改动源代码就需要发布一个新版本，使用者就需要更新pom文件到项目版本号，或者本地仓库删掉旧依赖，重新引入依赖，这样非常麻烦
2. 每次更新项目都需要通知使用者更新版本号

如何解决如上问题呢？

那么我们就需要使用`快照`，快照相当于项目到一个副本，我们可以在开发到时候使用快照，发布到时候在使用正式到版本号进行处理，使nexus的管理更加方便

1. 打开maven nexus 主页，登陆管理员账户，查看如下内容

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405164857.png?ynotemdtimestamp=1595729096959)

1. 在`Maven` 的`setting`文件当中添加如下内容，加入到对应的账号

```
<server>
		<id>snapshots</id>
		<username>admin</username>
		<password>admin123</password>	
	</server>
```

1. 修改项目版本号，一定要依照maven到 `snaphoto` 规则，否则部署快照版本会失败

```
<modelVersion>4.0.0</modelVersion>
<groupId>org.smart4j</groupId>
<artifactId>smart-framwork</artifactId>
<!-- 写法一定要规范 -->
<version>1.0-SNAPSHOT</version>
```

1. 执行`maven deloy` 部署配置

注意待发布项目的`pom.xml`配置如下

```
<distributionManagement>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>
```

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405170455.png?ynotemdtimestamp=1595729096959)

1. 查看`Maven Nexus`是否发布成功

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405170339.png?ynotemdtimestamp=1595729096959)

> 再次强调，如果部署失败，请查看仓库是否允许重复部署

1. 接下来我们试下快照是如何解决版本迭代的问题的

由于使用了快照版本，需要更改依赖如下:

```
<dependency>
    <groupId>org.smart4j</groupId>
    <artifactId>smart-framwork</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

1. 我们添加了一个main方法，然后点击IDEA `deploy`，部署快照版本的项目

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405201713.png?ynotemdtimestamp=1595729096959)

1. 我们在引用到项目里面可以看到项目已经引用到了本地当中

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200405202232.png?ynotemdtimestamp=1595729096959)