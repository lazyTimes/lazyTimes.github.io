---
title: 【RocketMq】NameServ启动脚本分析（Ver4.9.4）
subtitle: NameServ启动脚本分析
author: lazytime
url_suffix: rocketjiaoben
tags:
  - RocketMq
categories:
  - RocketMq
keywords: NameServ启动脚本分析
description: NameServ启动脚本分析
copyright: true
date: 2023-02-20 07:07:06
---
# NameServ启动脚本分析

## mqnamesrv 启动命令

这里直接摘录了官方文档：

**Start NameServer**

```shell
### Start Name Server first
$ nohup sh mqnamesrv &
```

```shell
### Then verify that the Name Server starts successfully
$ tail -f ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success...
```

## mqnamesrv 脚本

```bash
#!/bin/sh
# 从环境变量当中获取RocketMq环境变量地址
if [ -z "$ROCKETMQ_HOME" ] ; then
  ## resolve links - $0 may be a link to maven's home
  ## 解决链接问题 - $0 可能是maven的主页链接
  # PS：$0 是脚本的命令本身
  PRG="$0"

  # need this for relative symlinks
  # 需要相关链接
  while [ -h "$PRG" ] ; do
    ls=`ls -ld "$PRG"`
    link=`expr "$ls" : '.*-> \(.*\)$'`
    if expr "$link" : '/.*' > /dev/null; then
      PRG="$link"
    else
      PRG="`dirname "$PRG"`/$link"
    fi
  done
  # 暂存当前的执行路径
  saveddir=`pwd`

  ROCKETMQ_HOME=`dirname "$PRG"`/..

  # make it fully qualified
  # 拼接获取RocketMQ绝对路径
  ROCKETMQ_HOME=`cd "$ROCKETMQ_HOME" && pwd`
  # 跳转到当前暂存的命令执行路径
  cd "$saveddir"
fi

export ROCKETMQ_HOME

# 关键： 执行runserver.sh脚本，携带logback的日志xml配置，以及传递JVM的启动main方法的入口类绝对路径
sh ${ROCKETMQ_HOME}/bin/runserver.sh 
-Drmq.logback.configurationFile=$ROCKETMQ_HOME/conf/rmq.namesrv.logback.xml 
org.apache.rocketmq.namesrv.NamesrvStartup $@
```

最开始的`mqnamesrv.sh `脚本获取环境变量的部分看不懂其实没啥影响，大略有个印象即可，当然可以截取部分的命令到Linux运行测试一下就明白了，比如准备环境变量等等，最后一句话比较关键。

注意最后的两个字符`$@`，这两个字符的作用如下：

**$@** ：表示所有脚本参数的内容。

**$#** ：表示返回所有脚本参数的个数。

再次强调前面的一大坨获取环境变量看不懂没关系，看懂核心的执行脚本即可。

# runserver.sh 脚本

runserver.sh 的脚本内容如下：

```bash
#!/bin/sh

#===========================================================================================
# Java Environment Setting
#===========================================================================================
error_exit ()
{
    echo "ERROR: $1 !!"
    exit 1
}

find_java_home()
{
    # uname 是获取Linux内核参数的指令，不带任何参数获取当前操作系统的类型，比如Linux就是“Linux”的文本
    case "`uname`" in
        Darwin)
            JAVA_HOME=$(/usr/libexec/java_home)
        ;;
        *)
        # 可以简单认为获取到javac命令的绝对路径，然后执行两次cd..操作，以此作为JDK的路径
        # 比如 /opt/jdk/bin/javac dirname 两次之后就是 /opt/jdk
            JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
        ;;
    esac
}

# 调用函数
find_java_home

# 读取JAVA命令的执行地址
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

# export 导出的临时环境变量，只适用当前SHELL连接
# JAVA 命令的执行地址,设置为环境变量
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
# $0 代表当前的请求传递的第一个参数，根据上一个脚本可以知道是：${ROCKETMQ_HOME}/bin/runserver.sh
export BASE_DIR=$(dirname $0)/..
# 因为需要启动JVM进程，需要从ROCKETMQ_HOME的conf和lib路径告诉JDK找依赖包以及相关的配置文件
export CLASSPATH=.:${BASE_DIR}/conf:${BASE_DIR}/lib/*:${CLASSPATH}

#===========================================================================================
# JVM Configuration
#===========================================================================================
# The RAMDisk initializing size in MB on Darwin OS for gc-log
# 在 Darwin OS 上为 gc-log 初始化 RAMDisk 的大小（以 MB 为单位）
DIR_SIZE_IN_MB=600

choose_gc_log_directory()
{
    # Darwin 操作系统需要特殊处理，忽略
    case "`uname`" in
        Darwin)
            if [ ! -d "/Volumes/RAMDisk" ]; then
                # create ram disk on Darwin systems as gc-log directory
                DEV=`hdiutil attach -nomount ram://$((2 * 1024 * DIR_SIZE_IN_MB))` > /dev/null
                diskutil eraseVolume HFS+ RAMDisk ${DEV} > /dev/null
                echo "Create RAMDisk /Volumes/RAMDisk for gc logging on Darwin OS."
            fi
            GC_LOG_DIR="/Volumes/RAMDisk"
        ;;
        *)
         # 
            # check if /dev/shm exists on other systems
            # 检查 /dev/shm 是否存在于其他系统上
            # What Is /dev/shm And Its Practical Usage
            # https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html
            if [ -d "/dev/shm" ]; then
                GC_LOG_DIR="/dev/shm"
            else
                GC_LOG_DIR=${BASE_DIR}
            fi
        ;;
    esac
}

choose_gc_options()
{
    # 根据JDK的版本选择合适的GC参数，RocketMq最低需要JDK8，所以如果是1开头就是10以及之后的JDK版本
    # Example of JAVA_MAJOR_VERSION value: '1', '9', '10', '11', ...
    # '1' means releases befor Java 9
    JAVA_MAJOR_VERSION=$("$JAVA" -version 2>&1 | sed -r -n 's/.* version "([0-9]*).*$/\1/p')
    if [ -z "$JAVA_MAJOR_VERSION" ] || [ "$JAVA_MAJOR_VERSION" -lt "9" ] ; then
      # 小于JDK 9 版本的参数
      # 堆内存（初始堆内存）为 4 g，新生代 2g，其他空间为 2g。元空间初始化128m，最大的扩容元空间为320mb
      JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      # g1收集器在jdk11得到并行Full GC能力，而zgc在jdk11版本处于实验状态，这里选择了比较稳妥的 CMS 老年代垃圾回收器
      # UseCMSCompactAtFullCollection：CMS垃圾在进行了Full GC时，对老年代进行压缩整理，处理掉内存碎片
      # CMSParallelRemarkEnabled 使用CMS老年代收集器
      JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:-UseParNewGC"
      JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
      JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
    else
	  # JDK8 之后的参数
      JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
      JAVA_OPT="${JAVA_OPT} -Xlog:gc*:file=${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log:time,tags:filecount=5,filesize=30M"
    fi
}

choose_gc_log_directory
choose_gc_options
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

$JAVA ${JAVA_OPT} $@
```

`runserver.sh` 脚本的内容，内容比较多，这里拆分讲解。

## 前置准备工作

首先是有关环境变量的配置获取以及查找**JAVA_HOME**。

```bash
#!/bin/sh

#===========================================================================================
# Java Environment Setting
#===========================================================================================
error_exit ()
{
    echo "ERROR: $1 !!"
    exit 1
}

find_java_home()
{
    # uname 是获取Linux内核参数的指令，不带任何参数获取当前操作系统的类型，比如Linux就是“Linux”的文本
    case "`uname`" in
        Darwin)
            JAVA_HOME=$(/usr/libexec/java_home)
        ;;
        *)
        # 可以简单认为获取到javac命令的绝对路径，然后执行两次cd..操作，以此作为JDK的路径
        # 比如 /opt/jdk/bin/javac dirname 两次之后就是 /opt/jdk
            JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
        ;;
    esac
}

# 调用函数
find_java_home

# 读取JAVA命令的执行地址
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

# export 导出的临时环境变量，只适用当前SHELL连接
# JAVA 命令的执行地址,设置为环境变量
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
# $0 代表当前的请求传递的第一个参数，根据上一个脚本可以知道是：${ROCKETMQ_HOME}/bin/runserver.sh
export BASE_DIR=$(dirname $0)/..
# 因为需要启动JVM进程，需要从ROCKETMQ_HOME的conf和lib路径告诉JDK找依赖包以及相关的配置文件
export CLASSPATH=.:${BASE_DIR}/conf:${BASE_DIR}/lib/*:${CLASSPATH}
```

### 错误处理

首先是开头部分，如果出现异常就打印错误参数。在SH脚本文件中，`$1`代表了跟在脚本后面的第一个参数，比如`./script.sh filename1 dir1`，则`$1 = filename1`：

```java
error_exit ()
{
	// $1 通常是调用函数传入的第一个参数，比如下文的：Please set the JAVA_HOME variable in your environment, We need java(x64)!
    echo "ERROR: $1 !!"
    exit 1
}
```

### 查找JDK Home

接着是查找 JAVA_HOME 的位置：

```sh
find_java_home()
{
    # uname 是获取Linux内核参数的指令，不带任何参数获取当前操作系统的类型，比如Linux就是“Linux”的文本
    case "`uname`" in
        Darwin)
            JAVA_HOME=$(/usr/libexec/java_home)
        ;;
        *)
        # 可以简单认为获取到javac命令的绝对路径，然后执行两次 cd.. 操作，以此作为JDK的路径
        # 比如 /opt/jdk/bin/javac dirname 两次之后就是 /opt/jdk
            JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
        ;;
    esac
}
```

**uname** 是获取Linux内核参数的指令，不带任何参数获取当前操作系统的类型，比如Linux就是“Linux”的文本：

```sh
xander@xander:~$ uname 
Linux
```

这里通过uname 查找内核，如果是 darwin 的操作系统获取路径要特殊一些。而其他的方式则是`$(dirname $(dirname $(readlink -f $(which javac))))`层层查找：

```java
[zxd@localhost ~]$ echo $(dirname $(dirname $(readlink -f $(which javac))));
/opt/jdk8
```

这些可以简单认为获取到`javac`命令的绝对路径，然后执行两次`cd..`操作，以此作为JDK的路径。最简单的验证方法是放到Linux上执行一下：

```java
[zxd@localhost ~]$ echo $(readlink -f $(which javac))
/opt/jdk8/bin/javac
```

```java
[zxd@localhost ~]$ echo $(dirname $(dirname $(readlink -f $(which javac))));
/opt/jdk8
```

### 取JAVA命令的执行地址

这里比较简单，最后一句调用了error_exit函数，对应了第一个参数就是要打印的值。

```java
# 读取JAVA命令的执行地址
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"
```

```sh
# export 导出的临时环境变量，只适用当前SHELL连接
# 取JAVA 命令的执行地址,设置为环境变量
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
# $0 通常代表脚本名称本身，这里获取出来的结果是：${ROCKETMQ_HOME}
export BASE_DIR=$(dirname $0)/..
# 因为需要启动JVM进程，需要把ROCKETMQ_HOME的conf和lib路径告诉JDK找依赖包以及相关的配置文件
export CLASSPATH=.:${BASE_DIR}/conf:${BASE_DIR}/lib/*:${CLASSPATH}
```

上面需要注意的点：
1. export 导出的临时环境变量，只适用当前SHELL连接。
2. `$0` 代表当前j脚本名称本身，`../`和`dirname` 结合类似`../../`效果，`$(dirname $0)/..`为Rocketmq的安装目录地址。
3. 需要把`ROCKETMQ_HOME`的`conf`和`lib`路径告诉JDK找依赖包以及相关的配置文件。

```java
JAVA="$JAVA_HOME/bin/java"
BASE_DIR=${ROCKETMQ_HOME}/bin
CLASSPATH=.:${ROCKETMQ_HOME}/bin/conf:${BASE_DIR}/lib/*:${CLASSPATH}
```

## JVM 配置部分

之后往下的部分是JVM的配置部分，这部分我们拆分成两个部分来讲，最关键的是JVM参数配置部分，最开始是获取JVM 的GC_LOG地址，这里用`uname`识别操作系统，

```sh
# 在 Darwin OS 上为 gc-log 初始化 RAMDisk 的大小（以 MB 为单位）
DIR_SIZE_IN_MB=600

choose_gc_log_directory()
{
    # Darwin 操作系统需要特殊处理，忽略
    case "`uname`" in
        Darwin)
            if [ ! -d "/Volumes/RAMDisk" ]; then
                # create ram disk on Darwin systems as gc-log directory
                DEV=`hdiutil attach -nomount ram://$((2 * 1024 * DIR_SIZE_IN_MB))` > /dev/null
                diskutil eraseVolume HFS+ RAMDisk ${DEV} > /dev/null
                echo "Create RAMDisk /Volumes/RAMDisk for gc logging on Darwin OS."
            fi
            GC_LOG_DIR="/Volumes/RAMDisk"
        ;;
        *)
         # 
            # check if /dev/shm exists on other systems
            # 检查 /dev/shm 是否存在于其他系统上
            # What Is /dev/shm And Its Practical Usage
            # https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html
            if [ -d "/dev/shm" ]; then
                GC_LOG_DIR="/dev/shm"
            else
                GC_LOG_DIR=${BASE_DIR}
            fi
        ;;
    esac
}
```

在Linux系统中，会检查 `/dev/shm `是否存在系统上，如果是就把GC_LOG挂载到`/dev/shm`，否则就当前RocketMQ的安装目录`GC_LOG_DIR=${BASE_DIR}`。

这里补充一下`/dev/shm `是什么？在Linux中，这块空间起很大的作用，因为他不是硬盘而是一块内存空间，默认为VM的一半大小，使用`df -h`命令可以看到：

```sh
xander@xander:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              389M  1.7M  388M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   14G  7.3G  5.8G  56% /
tmpfs                              1.9G     0  1.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  247M  1.6G  14% /boot
tmpfs                              389M  4.0K  389M   1% /run/user/1000

```

RocketMq为什么要使用这一块空间？

tmpfs有以下优势：

1. 动态文件系统的大小。

2. tmpfs 文件系统会完全驻留在 RAM 中，拥有近似内存的读写速度。

而缺点仅仅是 tmpfs 数据在重新启动之后不会保留，可以做一些脚本备份的操作。

#### tmpfs 和 /dev/shm 解释

这块空间的专业名词叫做：tmpfs（虚拟内存系统），**tmpfs最大的特点就是它的存储空间在VM(virtual memory)，VM是由linux内核里面的vm子系统管理的**。

VM中又划分为RM (real memory) 和 swap，RM 就是VM实际可用的内存空间，而swap是用于辅助VM在RM不够的时候牺牲硬盘作为内存空间使用，同样RM还会把不常用的数据放到Swap。

tmpfps = RM (real memory) + swap。

tmpfs默认的大小是RM的一半，假如你的物理内存是1024M，那么tmpfs默认的大小就是512M。可以通过mount命令扩大这块空间大小。

tmpfps的存在意义是可以动态的扩容和缩小，并且只要不使用这块空间它本身没有任何的内存占用（0字节）（零成本还好用），而一旦使用则可以把读写停留在内存保证数据瞬间完成，但是代价是这块空间不具备记忆功能，重启之后不会被保留。

参考资料：
- [What Is /dev/shm And Its Practical Usage](https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html)

## JVM核心参数

核心参数的部分就是JVM的启动参数配置，也是脚本最为核心部分。

## 小于JDK9的启动参数

对应了 **gc_options** 的上半部分，首先判断JDK版本小于9之前的情况：

```sh
    if [ -z "$JAVA_MAJOR_VERSION" ] || [ "$JAVA_MAJOR_VERSION" -lt "9" ] ; then
      # 堆内存（初始堆内存）为 4 g，新生代 2g，其他空间为 2g。元空间初始化128m，最大的扩容元空间为320mb
      JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      # g1收集器在jdk11得到并行Full GC能力，而zgc在jdk11版本处于实验状态，这里选择了比较稳妥的 CMS 老年代垃圾回收器
      # UseCMSCompactAtFullCollection：CMS垃圾在进行了Full GC时，对老年代进行压缩整理，处理掉内存碎片
      # CMSParallelRemarkEnabled 使用CMS老年代收集器
      JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:-UseParNewGC"
      JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
      JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
```

首先是JVM堆大小和元空间大小分配：

```sh
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

### -server

启用`-server`时新生代默认采用**并行收集**，其他情况下，默认不启用。`-server`策略为：新生代使用并行清除，老年代使用单线程`Mark-Sweep-Compact`的垃圾收集器。

### 堆大小设置

设置JVM的堆大小，堆内存（初始堆内存）为 4g，新生代 2g，其他空间为 2g，元空间初始化128M，最大的扩容元空间为320M。

> 如果是个人机器配置比较低，建议把这几个值调小一些。


下面的参数比较关键，RocketMq在JDK8没有选择G1而是使用了CMS，因为G1收集器在jdk11才得到并行Full GC能力，而ZGC在JDK11版本处于实验状态，在JDK8 用不成熟的G1不太合适。

```sh
# UseCMSCompactAtFullCollection：CMS垃圾在进行了Full GC时，对老年代进行压缩整理，处理掉内存碎片
# CMSParallelRemarkEnabled 使用CMS老年代收集器
JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:-UseParNewGC"
```

### UseCMSCompactAtFullCollection

CMS垃圾在进行了Full GC时，对老年代进行压缩整理，处理掉内存碎片

### CMSParallelRemarkEnabled 

老年代收集器指定为CMS，在进行了Full GC时对老年代进行压缩整理，处理掉内存碎片。

```sh
-XX:+UseCMSCompactAtFullCollection
```

### CMS垃圾收集器

这里补充一下CMS垃圾收集器的知识点：

CMS是基于**标记清除**算法实现的多线程老年代垃圾回收器。CMS为**响应时间优先**的垃圾回收器，适合于应用服务器，如网络游戏，电商等和电信领域的应用。Rocketmq本身就是诞生于电商平台使用CMS是比较稳妥的。

### CMS垃圾收集器特点

-   FUll GC大部分时间和应用程序并行，代价是增加CPU开销。
-   并发FULL GC短暂停顿。
-   用户线程和回收线程并行。

垃圾回收算法：标记清除算法

之后是对应的CMS常用优化参数。

### -XX:+UseCMSCompactAtFullCollection

设置此参数之后，CMS垃圾在进行了Full GC时，对老年代进行压缩整理，处理掉内存碎片。RocketMq 的脚本也开启了每次Full Gc之后进行碎片整理。

### -XX:CMSFullGCsBeforeCompaction=1

FUll GC 之后对老年代进行压缩整理，处理掉内存碎片。RocketMq 的默认应对策略是积极的进行内存碎片整理，缩小老年代的大小，因为RocketMq需要的是高响应时间。

### CMSInitiatingOccupancyFraction=70

**CMSInitiatingOccupancyFraction=70** 表示当老年代达到70%时，触发CMS垃圾回收。
-   计算老年代最大使用率（\_initiating\_occupancy）
-   大于等于0则直接取百分号
-   小于0则根据公式来计算

如果使用默认值，则老年代触发回收的比例是动态的，不同的JDK版本可能会有不同的默认值，

```sh
((100 - MinHeapFreeRatio) + 
 (double) ( CMSTriggerRatio * MinHeapFreeRatio) / 100.0) 
 / 100.0
```

## -XX:SoftRefLRUPolicyMSPerMB=0

官方解释是：Soft reference 在虚拟机中比在客户集中存活的更长一些。

先说一下结论，个人认为这个参数设置为0是**不应该**的，至少需要设置个1000或者500（半秒）的缓冲，为什么呢？

我们**假设我们不小心把这个值设置为0有什么后果呢**？

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230214080954.png)

如果`-XX:SoftRefLRUPolicyMSPerMB=0`，会导致上面clock公式的计算结果为0。

这个结果为0，就是软饮用被频繁回收导致触发频繁的GC，JVM发现每次反射分配的对象马上就会被回收掉，然后接着又会通过代理生成代理对象，**导致每次soft软引用的对象一旦分配就会马上被回收**.

结论就是这个值为0，**反射机制导致动态代理类不断的被新增，但是这部分对象又被马上回收掉，导致方法区的垃圾对象越来越多**，会导致很多垃圾无法完全回收。

为什么RocketMq默认要把`-XX:SoftRefLRUPolicyMSPerMB=0`值设置为0？

我们接着分析，`-XX:SoftRefLRUPolicyMSPerMB=0`，但是使用的是服务器模式（-server），在server模式下，会用 **-Xmx** 参数获取空闲空间大小。  

空闲的空间越大，软引用就会活的越久，如果设置的值过大，很可能因为框架反射类创建的软引用过多但是因为存在空闲时间计算又没法回收的情况。

把这个值设置为0，基本上就是说不让反射产生的一些meta对象在GC之后回收不掉，直接通过1次GC就给他摆平了。但是个人角度来看未免过于极端，个人认为设置为0是不合适的。

> 参考案例：
> 这里在网上找到电子表格缓存业务因为设置 **-XX:SoftRefLRUPolicyMSPerMB=0** 导致的问题例子。
> 当JVM参数中配置了 **-XX:SoftRefLRUPolicyMSPerMB=0** 参数，这个参数是控制SoftReference缓存时间，而我们的电子表格的缓存都是存储SoftReference里边的，当设置了这个参数设置为0的时候，任意操作，只要是触发了gc，这时候就会清空了电子表格缓存，导致即使在内存足够的情况下，缓存也不生效了。

清除频率可以用命令行参数` -XX:SoftRefLRUPolicyMSPerMB=<N>`来控制，可以指定每兆堆空闲空间的 Soft reference 保持存活（一旦它饮用后不可达了）的毫秒数，这意味着每兆堆中的空闲空间中的 soft reference 会（在最后一个强引用被回收之后）存活1秒钟。

当然并不是什么时候 **-XX:SoftRefLRUPolicyMSPerMB=0**都是错的，因为 soft reference 只会在垃圾回收时才会被清除，而垃圾回收并不总在发生。系统默认为一秒，如果觉得客户端不需要任何保留，改为 `-XX:SoftRefLRUPolicyMSPerMB=0` 可以及时清理干净数据。

RocketMq的做法个人理解为想要让垃圾回收尽可能的回收干净对象，因为RocketMq并不是十分吃JVM堆内存，更多的是需要页缓存，况且NameServ本身比较轻量级。

还有一个原因是软引用这东西能不用就尽量不用，风险比较大。

### -XX:+CMSClassUnloadingEnabled

老年代启用CMS，但默认是不会回收永久代(Perm)的。此处启用对Perm区启用类回收，防止Perm区内存垃圾对象堆满（**需要与+CMSPermGenSweepingEnabled同时启用**）。

> **-XX:+CMSPermGenSweepingEnabled**：
>
> 同上，为了避免Perm区满引起的Full GC，开启并发收集器回收Perm区选项。

但是实际上这篇帖子上指出 [https://stackoverflow.com/questions/3717937/cmspermgensweepingenabled-vs-cmsclassunloadingenabled](https://stackoverflow.com/questions/3717937/cmspermgensweepingenabled-vs-cmsclassunloadingenabled "https://stackoverflow.com/questions/3717937/cmspermgensweepingenabled-vs-cmsclassunloadingenabled")

`-XX：+CMSClassUnloadEnabled `和 `-XX：+CMSPermGenSweepingEnabled` 在 Java 1.7 中不可用，但是选项`-XX：+CMSClassUnloadEnabled`对于Java 1.7仍然有效。换句话说在JDK1.7之后被建议使用`-XX：+CMSClassUnloadEnabled`。

JVM1.7之前是什么情况？为什么会**需要与+CMSPermGenSweepingEnabled同时启用**

下面这篇文章有评论进行了解释：

[用户对问题“CMSPermGenSweepingEnabled vs CMSClassUnloadingEnabled”的回答 - 问答 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/ask/sof/114185/answer/102415098 "用户对问题“CMSPermGenSweepingEnabled vs CMSClassUnloadingEnabled”的回答 - 问答 - 腾讯云开发者社区-腾讯云 (tencent.com)")

1.6 JVM的`CMSPermGenSweepingEnabled`参数做的唯一事情就是打印消息，它的处理方式与1.5不同。要使`CMSClassUnloadingEnabled`生效，还必须设置`UseConcMarkSweepGC`。



### -XX:SurvivorRatio=8

CMS 用的是标记清除的算法，使用CMS还是传统的**新生代和老年代的分代概念**，这里RocketMq用的是默认的分代分区策略，给了新生代更多的使用空间。它定义了新生代中Eden区域和Survivor区域（From幸存区或To幸存区）的比例，默认为8，也就是说Eden占新生代的8/10，From幸存区和To幸存区各占新生代的1/10.

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230219212846.png)


### -UseParNewGC

`-XX:+UseParNewGC` 设置年轻代为多线程收集。可与CMS收集同时使用，ParNew 在Serial基础上实现的**多线程收集器**。

### 日志参数配置

总得来说 RocketMq 在JDK8的版本使用了老牌的`-XX:+UseConcMarkSweepGC` CMS垃圾收集器 和` -XX:+UseParNewGC` CMS垃圾 垃圾收集器。

```sh
 -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
```

`-verbose:gc`和`-XX:+PrintGCDetails`在官方文档中有说明两者**功能一样**，都用于**垃圾收集时的信息打印**。

但是也有不同点：

**-verbose:gc** 是 稳定版本，参见：[http://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html "http://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html")

**-XX:+PrintGC** 是非稳定版本。

**-XX:+PrintGCDateStamps**，日志中添加时间标志（日志每行开头显示自从JVM启动以来的时间，单位为秒）

注意`-XX:+PrintGCDateStamps`  打印GC发生时的[时间戳](https://so.csdn.net/so/search?q=时间戳\&spm=1001.2101.3001.7020 "时间戳")，搭配` -XX:+PrintGCDetails` 使用，**不可以独立使用**

日志输出示例：

```纯文本
2014-01-03T12:08:38.102-0100: [GC 66048K->53077K(251392K), 0,0959470 secs]

2014-01-03T12:08:38.239-0100: [GC 119125K->114661K(317440K), 0,1421720 secs]
```

`-XX:+UseGCLogFileRotation` 、`-XX:NumberOfGCLogFiles=5`、 `-XX:GCLogFileSize=30m`。UseGCLogFileRotation 会根据后面两个参数的设置不断的**轮询替换GC日志**，这里最多保留了150M的GC日志，后续再进行写入就会从第一个文件开始替换。

>  150M日志足够及时处理大部分问题，并且不会出现历史日志长期驻留磁盘的问题。但是这个参数需要谨慎设置，如果设置过小容易导致关键GC 日志丢失。

## JDK9之后的启动参数

应该说是比较关键的版本，很明显都是围绕G1垃圾收集器做的优化：

```java
    else
      JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
      JAVA_OPT="${JAVA_OPT} -Xlog:gc*:file=${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log:time,tags:filecount=5,filesize=30M"
```

第一行

堆内存分配基本没啥区别：堆内存（初始堆内存）为 4g，新生代 2g，其他空间为 2g。元空间初始化128m，最大的扩容元空间为320mb。

第二行

垃圾回收器的关键配置：

```java
-XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0
```

### -XX:+UseG1GC

使用G1垃圾收集器。需要注意JDK8的G1垃圾收集器是“残血”版本。

### -XX:G1HeapRegionSize

一个Region的大小可以通过参数`-XX:G1HeapRegionSize`设定，取值范围从1M到32M，且是2的指数。如果不设定，那么G1会根据Heap大小自动决定。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230219215610.png)


关键部分为`region_size = 1 << region_size_log`，左移的操作也就是2的倍数的概念。最后的参数判断会让region最大为32M，最小为1M，不允许超过这个范围。

### -XX:G1ReservePercent

`-XX:G1ReservePercent G1会保留一部分堆内存用来防止分配不了的情况，默认是10。

官方对于这个参数的介绍如下：

> \-XX:G1ReservePercent=10
> Sets the percentage of reserve memory to keep free so as to reduce the risk of to-space overflows. The default is 10 percent. When you increase or decrease the percentage, make sure to adjust the total Java heap by the same amount. This setting is not available in Java HotSpot VM, build&#x20;
> 设置保留内存的百分比，以减少到空间溢出的风险。默认是10%。当你增加或减少这个百分比时，请确保预留足够的空间并且调整Java堆大小。注意这个设置在Java HotSpot VM中是不可用的。

扩大这个数值可以保证在进行GC 的时候提供更多堆内存保证存活空间存放晋升老年代的Region.

**G1为分配担保预留的空间比例**：通过`-XX:G1ReservePercent`指定，默认**10%**。也就是老年代会预留10%的空间来给新生代的对象晋升，如果经常发生新生代晋升失败而导致 Full GC，那么可以适当调高此阈值。但是调高此值同时也意味着**降低了老年代的实际可用空间**。

### -XX:InitiatingHeapOccupancyPercent=30

**触发全局并发标记的老年代使用占比**，默认值45%。

默认值45%的含义是也就是老年代占堆的比例超过45%。如果Mixed GC周期结束后老年代使用率还是超过45%,那么会再次触发全局并发标记过程，这样就会导致频繁的老年代GC，影响应用吞吐量。

当然调大这个值的代价是可能导致年轻代谨升失败而导致FULL GC。RocketMq使用30的设置是让老年代提早的触发GC并且回收垃圾。

### -XX:SoftRefLRUPolicyMSPerMB=0

`-XX:SoftRefLRUPolicyMSPerMB=N `这个参数在是JVM系统参数和垃圾收集器无关。

`-XX:SoftRefLRUPolicyMSPerMB`参数，可以指定每兆堆空闲空间的软引用的存活时间，默认值是1000，也就是1秒。可以调低这个参数来触发更早的回收软引用。如果调高的话会有更多的存活数据，可能在GC后堆占用空间比会增加。 对于软引用，还是建议尽量少用，会增加存活数据量，增加GC的处理时间。


### 日志参数配置

配置和之前的版本是一样的，这里直接忽略了。

```sh
${JAVA_OPT} -Xlog:gc*:file=${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log:time,tags:filecount=5,filesize=30M"
```

# 总结


整个NameServ的启动脚本不算太复杂，这里简单归纳一下比较重要的点：
- JDK9之前没有用G1，因为不成熟，选择传统的CMS+ParNew经典组合。
- JDK9之后使用的是G1垃圾收集器，并且参数都是尽可能的给新生代预留空间。
- NameServ 新生代的压力会比较大，整体思路是尽可能的减少垃圾，通过积极的GC保证垃圾尽可能被回收。

# 写在最后

个人认为这种学习方式一举多得，还可以看到不少Shell脚本的使用技巧，挺不错的。

