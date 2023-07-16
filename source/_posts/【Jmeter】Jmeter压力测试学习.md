---
title: Jmeter压力测试学习
subtitle: 实战  压力Jmeter测试
author: lazytime
url_suffix: note20
tags:
  - Java
categories:
  - Java
keywords: jmeter 压力测试
description: 学习如何使用jmeter压力测试
copyright: true
date: 2020-08-29 22:44:19前言：
---



## 前言：

Jmeter 作为实操性的软件，更注重练习，理论作为参考掌握即可，不需要去具体的纠结



# 1. 目前用的常用测试工具对比

## 1、loadrunner

​	性能稳定，压测结果及细粒度大，可以自定义脚本进行压测，但是太过于重大，功能比较繁多

​	1、loadrunner

## 2、apache ab(单接口压测最方便)

​	模拟多线程并发请求,ab命令对发出负载的计算机要求很低，既不会占用很多CPU，也不会占用太多的内存，但却会给目标服务器造成巨大的负载, 简单DDOS攻击等

## 3、webbench

​	webbench首先fork出多个子进程，每个子进程都循环做web访问测试。子进程把访问的结果通过pipe告诉父进程，父进程做最终的统计结果。



<!-- more -->

# 2. Jmeter基本介绍和使用场景

## 1、压测不同的协议和应用

​	1) Web - HTTP, HTTPS (Java, NodeJS, PHP, [ASP.NET](http://ASP.NET), …) 	2) SOAP / REST Webservices 	3) FTP 	4) Database via JDBC 	5) LDAP 轻量目录访问协议 	6) Message-oriented middleware (MOM) via JMS 	7) Mail - SMTP(S), POP3(S) and IMAP(S) 	8) TCP等等

## 2、使用场景及优点

1）功能测试 	

2）压力测试 	

3）分布式压力测试 	

4）纯java开发 	

5）上手容易，高性能 	

6）提供测试数据分析 	

7）各种报表数据图形展示

# 3. 本地快速安装Jmeter4.x

## 1、需要安装JDK8。或者JDK9,JDK10 (建议不要使用jre10)

## 2、快速下载

​	windows： http://mirrors.tuna.tsinghua.edu.cn/apache//jmeter/binaries/apache-jmeter-4.0.zip 	

mac或者linux：http://mirrors.tuna.tsinghua.edu.cn/apache//jmeter/binaries/apache-jmeter-4.0.tgz

## 3、文档地址：http://jmeter.apache.org/usermanual/get-started.html

## 4、建议安装JDK环境，虽然JRE也可以，但是压测https需要JDK里面的 keytool工具

# 4. jemeter 目录结构

## bin:核心可执行文件，包含配置

jmeter.bat: windows启动文件： 	

jmeter: mac或者linux启动文件： 	

jmeter-server：mac或者Liunx分布式压测使用的启动文件  

jmeter-server.bat：mac或者Liunx分布式压测使用的启动文件 	

jmeter.properties: 核心配置文件 

## extras：插件拓展的包

lib:核心的依赖包 	

ext:核心包 	

junit:单元测试包

# 5. 改变jmeter的GUI界面语言版本

## 1、控制台修改

​	`menu -> options -> choose language`

## 2、配置文件修改

​	`bin目录 -> jmeter.properties 	默认 #language=en 	改为 language=zh_CN`

# 6. jemeter 菜单介绍

## 1、添加->threads->线程组（控制总体并发）

- 线程数：虚拟用户数。一个虚拟用户占用一个进程或线程
- 准备时长（Ramp-Up Period(in seconds)）：全部线程启动的时长，比如100个线程，20秒，则表示20秒内100个线程都要启动完成，每秒启动5个线程
- 循环次数：每个线程发送的次数，假如值为5，100个线程，则会发送500次请求，可以勾选永远循环

## 2、线程组->添加-> Sampler(采样器) -> Http （一个线程组下面可以增加几个Sampler）

- 名称：采样器名称
- 注释：对这个采样器的描述
- web服务器：
  - 默认协议是http
  - 默认端口是80
  - 服务器名称或IP ：请求的目标服务器名称或IP地址
- 路径：服务器URL
- **Use multipart/from-data for HTTP POST** ：当发送POST请求时，使用Use multipart/from-data方法发送，默认不选中。

## 3、查看测试结果

### 线程组->添加->监听器->察看结果树

# 7. 断言的基本使用

## 1. 增加断言: 线程组 -> 添加 -> 断言 -> 响应断言

apply to(应用范围): 	Main sample only: 仅当前父取样器 进行断言，一般一个请求，如果发一个请求会触发多个，则就有sub sample（比较少用）

## 2.要测试的响应字段：

- 响应文本：即响应的数据，比如json等文本
- 响应代码：http的响应状态码，比如200，302，404这些
- 响应信息：http响应代码对应的响应信息，例如：OK, Found
- Response Header: 响应头

## 3.模式匹配规则：

- 包括：包含在里面就成功
- 匹配：响应内容完全匹配，不区分大小写
- equals：完全匹配，区分大小写

## 4.断言结果监听器: 线程组-> 添加 -> 监听器 -> 断言结果

里面的内容是sampler采样器的名称 	断言失败，查看结果树任务结果颜色标红(通过结果数里面双击不通过的记录，可以看到错误信息)

## 5.每个sample下面可以加单独的结果树，然后同时加多个断言，最外层可以加个结果树进行汇总

# 8. Jmeter压测结果聚合报告分析

## 1.新增聚合报告：线程组->添加->监听器->聚合报告（Aggregate Report）

- lable: sampler的名称
- Samples: 一共发出去多少请求,例如10个用户，循环10次，则是 100
- Average: 平均响应时间
- Median: 中位数，也就是 50％ 用户的响应时间
- 90% Line : 90％ 用户的响应不会超过该时间 （90% of the samples took no more than this time. The remaining samples at least as long as this）
- 95% Line : 95％ 用户的响应不会超过该时间
- 99% Line : 99％ 用户的响应不会超过该时间
- min : 最小响应时间
- max : 最大响应时间
- Error%：错误的请求的数量/请求的总数
- **Throughput： 吞吐量——默认情况下表示每秒完成的请求数（Request per Second) 可类比为qps 	KB/Sec: 每秒接收数据量**



## 2.为什么使用：很多变量在全局中都有使用，或者测试数据更改，可以在一处定义，四处使用(比如服务器地址)

### 1、线程组->add -> Config Element(配置原件)-> User Definde Variable（用户定义的变量）

### 2、引用方式${XXX}，在接口中变量中使用

### 3、原始查看结果树和非原生查看（基础按钮）

# 9. CSV可变参数压测（重要）

**实战操作jmeter读取CSV和Txt文本文件里面的参数进行压测**

## 1、线程组->add -> Config Element(配置原件)-> CSV data set config (CSV数据文件设置)

## 2、在读取的配置文件里面，同时使用多个自定义参数

如果是多个参数需要同时引用，则在CSV数据文件里面设置加多个字段 Variabled names(comma-delitited): csv_name,csv_pwd

# 10、JDBC request压测Mysql

## 1、Thread Group -> add -> sampler -> jdbc request

## 2、jar包添加 mysql-connector-java-5.1.30.jar

## 3、JDBC connection Configuration 配置

**JDBC request->add -> config element -> JDBC connection configuration**

```
核心配置
	Max Number of connections : 最大连接数
	MAX wait :最大等待时间
	Auto Commit: 是否自动提交事务

	DataBase URL : 数据库连接地址 jdbc:mysql://127.0.0.1:3306/blog
	JDBC Driver Class : 数据库驱动，选择对应的mysql
	username:数据库用户名
	password:数据库密码
```

# 11. request压测Mysql, select语句

- 个人测试1

\

- 个人测试2
  - 注意：需要添加一个全局的debug采集器采集查询结果
  - 可以看到如图所示结果

## 1. Debug Sampler使用（结果树中查看）

​	Thread Group -> add -> sampler -> debug sampler

## 2. 参数讲解：(sql结尾不要加";")

- variable name of pool declared in JDBC connection configuration（和配置文件同名）
- Query Type 查询类型
- parameter values 参数值
- parameter types 参数类型
- variable names sql执行结果变量名
- result variable names 所有结果当做一个对象存储
- query timeouts 查询超时时间
- handle results 处理结果集

# 12. 分布式压测（学习重点）

官网教程： http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html

- 普通压测：单台机可以对目标机器产生的压力比较小，受限因素包括CPU，网络，IO等
- 分布式压测：利用多台机器向目标机器产生压力，模拟几万用户并发访问

# 13. Jmeter分布式压测原理(了解)

1、总控机器的节点master，其他产生压力的机器叫“肉鸡” server

2、master会把压测脚本发送到 server上面

3、执行的时候，server上只需要把jmeter-server打开就可以了，不用启动jmeter

4、结束后，server会把压测数据回传给master,然后master汇总输出报告

5、配置详情

# 14. 阿里云安装jmeter (守护进程：nohup java -jar xxxxx &)

## 1. 下载方式：wget http://apache.osuosl.org//jmeter/binaries/apache-jmeter-4.0.tgz

## 2. 直接到官网下载对应tag.gz包，并在服务器解压

# 15. Jmeter非GUI界面 参数讲解(必须掌握)

官方配置文件地址： http://jmeter.apache.org/usermanual/get-started.html

- **-h** 帮助
- **-n** 非GUI模式
- **-t** 指定要运行的 JMeter 测试脚本文件
- **-l** 记录结果的文件 每次运行之前，(要确保之前没有运行过,即xxx.jtl不存在，不然报错)
- **-r** Jmter.properties文件中指定的所有远程服务器
- **-e** 在脚本运行结束后生成html报告
- **-o** 用于存放html报告的目录（目录要为空，不然报错）

jmeter -n -t linux_users_api.jmx -l result.jtl -e -o /usr/local/softwate/jmeter/temp/ResultReport

jmeter -n -t /Users/jack/Desktop/linux_users_api.jmx -l result.jtl -e -o /Users/jack/Desktop/person/jmeter/temp

个人： /usr/local/software/jmeter/project/temp 。。。。

## 1.阿里云非GUI jmeter 压测

jmx目录：/usr/local/software/jmeter/temp

示例

jmeter -n -t /usr/local/software/jmeter/temp/linux_users_api.jmx -l /usr/local/software/jmeter/temp/jtl/result.jtl

利用软件从阿里云Centos服务器下载压测报告，讲解Jtl文件，并怎么查看文件 可以通过打开jmeter，新建线程组->summary report->浏览文件 进行查看

## 2.讲解Jmeter压测减少资源使用的一些建议，即压测结果更准确

1、使用非GUI模式：jmeter -n -t test.jmx -l result.jtl

2、少使用Listener， 如果使用-l参数，它们都可以被删除或禁用。

3、在加载测试期间不要使用“查看结果树”或“查看结果”表监听器，只能在脚本阶段使用它们来调试脚本。

4、包含控制器在这里没有帮助，因为它将文件中的所有测试元素添加到测试计划中。]

5、不要使用功能模式,使用CSV输出而不是XML

6、只保存你需要的数据,尽可能少地使用断言

7、如果测试需要大量数据，可以提前准备好测试数据放到数据文件中，以CSV Read方式读取。

8、用内网压测，减少其他带宽影响压测结果

9、如果压测大流量，尽量用多几个节点以非GUI模式向服务器施压

官方推荐 ：http://jakarta.apache.org/jmeter/usermanual/best-practices.html#lean_mean

## 3. 把Jmtere压测结果转换为Html

jmeter -n -t /usr/local/software/jmeter/temp/linux_users_api.jmx -l /usr/local/software/jmeter/temp/jtl/result.jtl -e -o /usr/local/software/jmeter/temp/result讲解压测报告 html里面Dashboard的核心指标

### 1、dashboard讲解

```
	1）Test and Report informations
		Source file：jtl文件名
		Start Time ：压测开始时间
		End Time ：压测结束时间
		Filter for display：过滤器
		Lable:sampler采样器名称	

	2）APDEX(Application performance Index)
		apdex:应用程序性能指标,范围在0~1之间，1表示达到所有用户均满意
		T(Toleration threshold)：可接受阀值
		F(Frustration threshold)：失败阀值

	3）Requests Summary
		OK:成功率
		KO:失败率
	4）Statistics 统计数据
		lable:sampler采样器名称

		samples:请求总数，并发数*循环次数
		KO:失败次数
		Error%:失败率

		Average:平均响应时间
		Min:最小响应时间
		Max:最大响应时间
		90th pct: 90%的用户响应时间不会超过这个值（关注这个就可以了）
		2ms,3ms,4,5,2,6,8,3,9

		95th pct: 95%的用户响应时间不会超过这个值
		99th pct: 99%的用户响应时间不会超过这个值 (存在极端值)
		throughtput:Request per Second吞吐量 qps

		received:每秒从服务器接收的数据量
		send：每秒发送的数据量
```

### 2、charts讲解

```
1)Over Time（随着时间的变化）
	Response Times Over Time：响应时间变化趋势
	Response Time Percentiles Over Time (successful responses)：最大，最小，平均，用户响应时间分布
	Active Threads Over Time：并发用户数趋势
	Bytes Throughput Over Time：每秒接收和请求字节数变化，蓝色表示发送，黄色表示接受
	Latencies Over Time：平均响应延时趋势
	Connect Time Over Time	：连接耗时趋势

1)Throughput
	Hits Per Second (excluding embedded resources):每秒点击次数
	Codes Per Second (excluding embedded resources)：每秒状态码数量
	Transactions Per Second：即TPS，每秒事务数
	Response Time Vs Request：响应时间和请求数对比
	Latency Vs Request：延迟时间和请求数对比

1)Response Times
	Response Time Percentiles：响应时间百分比
	Response Time Overview：响应时间概述
	Time Vs Threads：活跃线程数和响应时间
	Response Time Distribution：响应时间分布图
```

# 16、Linux服务器上jmeter进行分布式压测

## 1、压测注意事项

```
the firewalls on the systems are turned off or correct ports are opened.
系统上的防火墙被关闭或正确的端口被打开。

all the clients are on the same subnet.
所有的客户端都在同一个子网上。

the server is in the same subnet, if 192.x.x.x or 10.x.x.x IP addresses are used. If the server doesn't use 192.xx or 10.xx IP address, there shouldn't be any problems.
如果使用192.x.x.x或10.x.x.x IP地址，则服务器位于同一子网中。 如果服务器不使用192.xx或10.xx IP地址，则不应该有任何问题。

Make sure JMeter can access the server.
确保JMeter可以访问服务器。

Make sure you use the same version of JMeter and Java on all the systems. Mixing versions will not work correctly.
确保在所有系统上使用相同版本的JMeter和Java。 混合版本将无法正常工作。

You have setup SSL for RMI or disabled it.
您已为RMI设置SSL或将其禁用。

官网地址 http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html

压测注意事项：一定要用内网IP，不用用公网IP,用ping去检查
```

## 2、专业名字

master:司令 slave：

奴隶 target：目标

### 参考

地址：http://jmeter.apache.org/images/screenshots/distributed-names.svg 

地址：http://jmeter.apache.org/images/screenshots/distributed-jmeter.svg

## 3、远程拷贝(内网地址)：

scp -r /usr/local/software/jdk-8u141-linux-x64.tar.gz [root@172.18.230.233](http://mailto:root@172.18.230.233):/usr/local/software

scp -r /usr/local/software/jmeter/apache-jmeter-4.0.tgz [root@172.18.230.233](http://mailto:root@172.18.230.233):/usr/local/software/jmeter

## 4、启动

./jmeter-server 或者 nohup ./jmeter-server &

## 5、检查启动是否成功

ps -ef|grep jmeter-server

ps aux|grep jmeter-server

# 17、Jmeter分布式压测

## 本地非GUI分布式压测 -r

jmeter -n -t /Users/jack/Desktop/remote.jmx -r -l /Users/jack/Desktop/jtl/result.jtl -e -o /Users/jack/Desktop/result

## 压测结果

./jmeter -n -t /Users/jack/Desktop/remote.jmx -r -l /Users/jack/Desktop/jtl/result.jtl -e -o /Users/jack/Desktop/result

## 压测报告

```
Creating summariser <summary>
Created the tree successfully using /Users/jack/Desktop/remote.jmx
Configuring remote engine: 172.20.10.3:8899
Using local port: 8899
Configuring remote engine: 172.20.10.11:8899
Starting remote engines
Starting the test @ Thu Mar 29 23:21:13 CST 2018 (1522336873931)
Remote engines have been started
Waiting for possible Shutdown/StopTestNow/Heapdump message on port 4445
summary =      4 in 00:00:22 =    0.2/s Avg:  5582 Min:    94 Max: 21006 Err:     1 (25.00%)
Tidying up remote @ Thu Mar 29 23:21:36 CST 2018 (1522336896842)
... end of run
```

## 压测修改master节点信息

jemeter.properties 值是slave机器的ip+端口号，如果有多个，用逗号分隔 remote_hosts=192.168.0.102:8899,192.168.0.101:8899 server.rmi.ssl.disable=true (前面有说到)

## 启动slave机器，注意要同个网段，ip地址用内网ip

./jmeter-server Using local port: 8899 Created remote object: UnicastServerRef2 [liveRef: [endpoint:[192.168.0.102:8899](http://local),objID:[3a585a4d:162724586ab:-7fff, 3963132813614033916]]]

## 相关资料：

https://www.cnblogs.com/Fine-Chan/p/6233823.html https://blog.csdn.net/liujingqiu/article/details/52635289https://www.cnblogs.com/puresoul/p/4844539.html

# 18.、测试基本内容

**配置元件=》前置处理器=》定时器=》采样器=》后置处理器=》断言=》监听器**

# 19、安装常见问题

## 1、问题：无法解析

```
[root@iZwz95j86y235aroi85ht0Z bin]# ./jmeter-server
	Created remote object: UnicastServerRef2 [liveRef: [endpoint:[:39308](local),objID:[24e78a63:16243c70661:-7fff, 7492480871343944173]]]
	Server failed to start: java.rmi.RemoteException: Cannot start. Unable to get local host IP address.; nested exception is:
	java.net.UnknownHostException: iZwz95j86y235aroi85ht0Z: iZwz95j86y235aroi85ht0Z: Name or service not known
	An error occurred: Cannot start. Unable to get local host IP address.; nested exception is:
	java.net.UnknownHostException: iZwz95j86y235aroi85ht0Z: iZwz95j86y235aroi85ht0Z: Name or service not known
```

### 解决：

```
hostname  命令获取机器名称，追加一个映射  iZwz95j86y235aroi85ht0Z
		vim /etc/hosts
			127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
			::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
			120.79.160.143 iZwz95j86y235aroi85ht0Z

windows用户 修改c:\windows\system32\drivers\etc\hosts文件，增加一条域名 与IP的映射
```

## 2.问题：禁用了SSL

```
[root@iZwz95j86y235aroi85ht0Z bin]# ./jmeter-server
	Server failed to start: java.rmi.server.ExportException: Listen failed on port: 0; nested exception is:
	java.io.FileNotFoundException: rmi_keystore.jks (No such file or directory)
	An error occurred: Listen failed on port: 0; nested exception is:
	java.io.FileNotFoundException: rmi_keystore.jks (No such file or directory)
```

### 解决：

```
拥有RMI over SSL的有效密钥库，或者禁用了SSL。
1、禁用SSL
  jmeter.property里面 server.rmi.ssl.disable 改为 true，表示禁用
```

## 3、问题：

```
[root@iZ949uw2xehZ bin]# ./jmeter
		Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 1073741824, 0) failed; error='Cannot allocate memory' (errno=12)
		#
		# There is insufficient memory for the Java Runtime Environment to continue.
		# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
		# An error report file with more information is saved as:
		# /usr/local/jmeter/apache-jmeter-4.0/bin/hs_err_pid5855.log
```

### 解决：

```
编辑jmeter
搜索 : "${HEAP:="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"}"
改变初始堆内存和最大堆内存
```

## 4、仅修改 server_port 即可,下面两者一样

server.rmi.localport=8899 表示slave server启动显示的端口

server_port=8899 表示master机器要远程连接的端口 即 remote_hosts=xxxx:8899

## 5、-----

```
<!-- jmeter 分布式性能测试（多网卡配置） -->
我们要在多网卡的服务器上开启RMI服务的话必须指定IP，使他们能够在同一个网段内。 

需要以下几步（假定所有机器都在10.120.11.*网段,agent服务器为linux,controller服务器为windows）：

1、 修改agent服务器，指定agent机器的IP
修改jmeter-server文件
# vi jmeter-server
修改RMI_HOST_DEF=-Djava.rmi.server.hostname=xxx.xxx.xxx.xxx(需要连接的IP)

2、修改server服务器，指定server机器的IP

修改jmeter.bat文件 

新增set rmi_host=-Djava.rmi.server.hostname=10.120.11.214

修改set ARGS=%DUMP% %HEAP% %NEW% %SURVIVOR% %TENURING% %PERM% %DDRAW% %rmi_host%
```

## 6、确定在controller机器上安装jdk,版本和jmeter一致，配置环境变量：Java_home等

- 在Agent机器上安装jdk，配置环境变量：Java_home和JMeter_home
- 安装目录不要带空格，最好都是简短的英文路径

## 7、master机器启动后会拷贝jmx文件到slave机器，所以不需要在每台slave机器上也上传一份jmx，只需要在master机器上上传一份jmx脚本即可。

```
如果使用csv进行参数化，则需要把参数文件在每台slave上拷一份且路径需要设置成一样的。

总样本数 = 线程数 * 循环次数 * 执行机总数
```

## 8、连接失败原因排查

#### 以下步骤进行排查：

```
1. jmeter-server是否启动；
	2. 是否联网
	3. ping 服务器IP是否畅通.
	4. telnet 端口 192.168.3.10 1099
	5. 检查服务器的防火墙是否关闭。
	6. 阿里云安全策略是否正常
```

## 9、"could not find ApacheJmeter_core.jar"

​	**解决：在Agent机器安装jdk，并设置环境变量**

## 10、”Bad call to remote host"

解决：检查被控制机器上的jmeter-server有没有启动，或者remote_hosts的配置是否正确。