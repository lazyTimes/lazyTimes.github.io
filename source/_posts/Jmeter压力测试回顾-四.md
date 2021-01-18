---
title: Jmeter压力测试回顾(四)
subtitle: Jmeter压力测试回顾最后部分
url_suffix: jmeter-test4
author: lazytime
tags:
  - jmeter
categories:
  - 压力测试
keywords: Jmeter压力测试回顾最后部分
description: Jmeter压力测试回顾最后部分
copyright: true
date: 2021-01-18 23:35:19
---

# 回顾Jmeter压力测试（四）

下面部分为非GUI部分的内容实战。

<!-- more -->

## 非GUI部分

## 阿里云安装jmeter（jmeter5）

1. 配置JDK1.8的环境，Linux配置Jdk请自行查找相关资料
2. 阿里云安装jmeter，移动到自己喜欢的位置

> 下载方式：`wget https://apachemirror.sg.wuchna.com//jmeter/binaries/apache-jmeter-5.4.tgz`

3. 启动非GUI测试：我们可以在window上使用gui版本编写好对应的jmx，然后再执行压测。



### Jmeter非GUI界面 参数讲解(必须掌握)

在进行实战之前了解一下基本的参数：

官方配置文件地址： http://jmeter.apache.org/usermanual/get-started.html

#### 基础命令参数：

+ **-h** 	帮助
+ **-n**     非GUI模式
+ **-t** 	指定要运行的 JMeter 测试脚本文件
+ **-l**  	记录结果的文件 每次运行之前，(要确保之前没有运行过,即xxx.jtl不存在，不然报错)
+ **-r** 	 Jmter.properties文件中指定的所有远程服务器
+ **-e**  	在脚本运行结束后生成html报告
+ **-o**  	用于存放html报告的目录（目录要为空，不然报错）

案例如下：

### 如何使用linux jmeter压测？

下面的教程为构建一个最简单的压测方式：

1. 构建一个简单的http压测请求，使用如下的方式：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210118223554.png)

2. 保存为`.jmx`后缀的文件
3. 将文件的内容上传到服务器，命令行进入到对应的Bin目录下面
4. 执行命令：`./jmeter -n -t ./plan/LinuxPlan.jmx -l ./plan/result.jtl -e -o ./resultreport`

> 上面的命令效果如下：：
>
> -l 指定了结果文件的存放位置
>
> -o 指定html报告的目录位置
>
> -t 指定了脚本文件的存放位置
>
> -n 代表使用非GUI模式运行



### Jmeter压测减少资源使用的一些建议，即压测结果更准确

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

## Jmeter压测结果聚合报告分析

一般情况下，我们可以根据`-o`这个参数在Linux中生成对应的聚合报告，根据聚合报告我们可以分析压测的结果

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210118224902.png)

根据上面的聚合报告，我们可以从`index.html`当中看到基本的聚合报告分析内容和结果。

### dashboard结果分析：


​	
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
![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210118224707.png)

#### Test and Report informations

+ Source file：jtl文件名
+ Start Time ：压测开始时间
+ End Time ：压测结束时间
+ Filter for display：过滤器
+ Lable:sampler：采样器名称

#### APDEX(Application performance Index)

apdex:应用程序性能指标,范围在0~1之间，1表示达到所有用户均满意

+ T(Toleration threshold)：可接受阀值
+ F(Frustration threshold)：失败阀值

#### Requests Summary:

+ OK:成功率
+ KO:失败率

#### Statistics 统计数据

这一部分是主要内容：

+ lable: sampler的名称
+ Samples: 一共发出去多少请求,例如10个用户，循环10次，则是 100
+ Average: 平均响应时间
+ Median: 中位数，也就是 50％ 用户的响应时间
+ 90% Line : 90％ 用户的响应不会超过该时间 （90% of the samples took no more than this time. The remaining samples at least as long as this）
+ 95% Line : 95％ 用户的响应不会超过该时间
+ 99% Line : 99％ 用户的响应不会超过该时间
+ min : 最小响应时间
+ max : 最大响应时间
+ Error%：错误的请求的数量/请求的总数
+ **Throughput： 吞吐量——默认情况下表示每秒完成的请求数（Request per Second) 可类比为qps
  		KB/Sec: 每秒接收数据量**

### charts讲解

#### Over Time（随着时间的变化）

+ Response Times Over Time：响应时间变化趋势
+ Response Time Percentiles Over Time (successful responses)：最大，最小，平均，用户响应时间分
+ Active Threads Over Time：并发用户数趋势
+ Bytes Throughput Over Time：每秒接收和请求字节数变化，蓝色表示发送，黄色表示接受
+ Latencies Over Time：平均响应延时趋势
+ Connect Time Over Time	：连接耗时趋势

#### Throughput

+ Hits Per Second (excluding embedded resources):每秒点击次数
+ Codes Per Second (excluding embedded resources)：每秒状态码数量
+ Transactions Per Second：即TPS，每秒事务数
+ Response Time Vs Request：响应时间和请求数对比
+ Latency Vs Request：延迟时间和请求数对比

#### Response Times

+ Response Time Percentiles：响应时间百分比
+ Response Time Overview：响应时间概述
+ Time Vs Threads：活跃线程数和响应时间
+ Response Time Distribution：响应时间分布图

# 安装常见问题

问题1：

```
[root@iZwz95j86y235aroi85ht0Z bin]# ./jmeter-server
	Created remote object: UnicastServerRef2 [liveRef: [endpoint:[:39308](local),objID:[24e78a63:16243c70661:-7fff, 7492480871343944173]]]
	Server failed to start: java.rmi.RemoteException: Cannot start. Unable to get local host IP address.; nested exception is:
	java.net.UnknownHostException: iZwz95j86y235aroi85ht0Z: iZwz95j86y235aroi85ht0Z: Name or service not known
	An error occurred: Cannot start. Unable to get local host IP address.; nested exception is:
	java.net.UnknownHostException: iZwz95j86y235aroi85ht0Z: iZwz95j86y235aroi85ht0Z: Name or service not known
```

解决方式：

```
hostname  命令获取机器名称，追加一个映射  iZwz95j86y235aroi85ht0Z
		vim /etc/hosts
			127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
			::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
			120.79.160.143 iZwz95j86y235aroi85ht0Z

windows用户 修改c:\windows\system32\drivers\etc\hosts文件，增加一条域名 与IP的映射
```

2. 问题如下：

```
[root@iZwz95j86y235aroi85ht0Z bin]# ./jmeter-server
	Server failed to start: java.rmi.server.ExportException: Listen failed on port: 0; nested exception is:
	java.io.FileNotFoundException: rmi_keystore.jks (No such file or directory)
	An error occurred: Listen failed on port: 0; nested exception is:
	java.io.FileNotFoundException: rmi_keystore.jks (No such file or directory)
```

解决：

```
拥有RMI over SSL的有效密钥库，或者禁用了SSL。
1、禁用SSL
  jmeter.property里面 server.rmi.ssl.disable 改为 true，表示禁用
```

3、问题：

```
[root@iZ949uw2xehZ bin]# ./jmeter
		Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 1073741824, 0) failed; error='Cannot allocate memory' (errno=12)
		#
		# There is insufficient memory for the Java Runtime Environment to continue.
		# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
		# An error report file with more information is saved as:
		# /usr/local/jmeter/apache-jmeter-4.0/bin/hs_err_pid5855.log
```

解决：

```
编辑jmeter
搜索 : "${HEAP:="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"}"
改变初始堆内存和最大堆内存
```

4、仅修改 server_port 即可,下面两者一样

server.rmi.localport=8899 表示slave server启动显示的端口

server_port=8899  表示master机器要远程连接的端口   即 remote_hosts=xxxx:8899

5、jmeter 分布式性能测试（多网卡配置）

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

6、确定在controller机器上安装jdk,版本和jmeter一致，配置环境变量：Java_home等

+ 在Agent机器上安装jdk，配置环境变量：Java_home和JMeter_home

+ 安装目录不要带空格，最好都是简短的英文路径

7、master机器启动后会拷贝jmx文件到slave机器，所以不需要在每台slave机器上也上传一份jmx，只需要在master机器上上传一份jmx脚本即可。

	如果使用csv进行参数化，则需要把参数文件在每台slave上拷一份且路径需要设置成一样的。
	
	总样本数 = 线程数 * 循环次数 * 执行机总数

8、连接失败原因排查

以下步骤进行排查：

     		1. jmeter-server是否启动；
     		2. 是否联网
          	3. ping 服务器IP是否畅通.
          	4. telnet 端口 192.168.3.10 1099
          	5. 检查服务器的防火墙是否关闭。
          	6. 阿里云安全策略是否正常

9、出现："could not find ApacheJmeter_core.jar"

	**解决：在Agent机器安装jdk，并设置环境变量**

10、出现：”Bad call to remote host"

解决：检查被控制机器上的jmeter-server有没有启动，或者remote_hosts的配置是否正确。