---
title: 【Kafka】Kafka-Server-start.sh 启动脚本分析（Ver 2.7.2）
subtitle: Kafka-Server-start.sh启动脚本分析
author: Xander
url_suffix: kafkastart
tags:
  - Kafka
categories:
  - Kafka
keywords: Kafka,Kafka-Server-start.sh
description: Kafka-Server-start.sh启动脚本分析
copyright: true
date: 2023-02-27 07:42:33
---
# Kafka-Server-start.sh

```sh
  
if [ $# -lt 1 ];  
then  
 # 提示命令使用方法
 echo "USAGE: $0 [-daemon] server.properties [--override property=value]*" exit 1  
fi  
base_dir=$(dirname $0)  
  
if [ "x$KAFKA_LOG4J_OPTS" = "x" ]; then  
    export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$base_dir/../config/log4j.properties"  
fi  
  

if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then  
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"  
fi  
  
EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer -loggc'}  
  
COMMAND=$1  
case $COMMAND in  
  -daemon)  
    EXTRA_ARGS="-daemon "$EXTRA_ARGS  
    shift  
    ;;  
  *)  
    ;;  
esac  
  
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"
```

1. 判断参数有没有，参数个数小于1就提示用法；
2. 获取脚本当前路径赋值给变量 base_dir；
3. 判断日志参数 **KAFKA_LOG4J_OPTS** 是否为空，为空就给它一个值；
4. 判断堆参数 **KAFKA_HEAP_OPTS**是否为空，为空就默认给它赋值为 "-Xmx1G -Xms1G"，默认的堆空间指定为1G；
5. 判断启动命令中第一个参数是否为 `-daemon`，如果是就以**守护进程启动**（其实不是，是赋给另一个变量 EXTRA_ARGS）；
6. 执行命令。

最后一个脚本是执行另一个脚本：`kafka-run-class.sh`，这个脚本的内容比较复杂了。

<!-- more -->

# kafka-run-class.sh

```sh
#!/bin/bash
if [ $# -lt 1 ];
then
  echo "USAGE: $0 [-daemon] [-name servicename] [-loggc] classname [opts]"
  exit 1
fi

# CYGWIN == 1 if Cygwin is detected, else 0.
if [[ $(uname -a) =~ "CYGWIN" ]]; then
  CYGWIN=1
else
  CYGWIN=0
fi

if [ -z "$INCLUDE_TEST_JARS" ]; then
  INCLUDE_TEST_JARS=false
fi

# Exclude jars not necessary for running commands.
regex="(-(test|test-sources|src|scaladoc|javadoc)\.jar|jar.asc)$"
should_include_file() {
  if [ "$INCLUDE_TEST_JARS" = true ]; then
    return 0
  fi
  file=$1
  if [ -z "$(echo "$file" | egrep "$regex")" ] ; then
    return 0
  else
    return 1
  fi
}

base_dir=$(dirname $0)/..

if [ -z "$SCALA_VERSION" ]; then
  SCALA_VERSION=2.13.3
  if [[ -f "$base_dir/gradle.properties" ]]; then
    SCALA_VERSION=`grep "^scalaVersion=" "$base_dir/gradle.properties" | cut -d= -f 2`
  fi
fi

if [ -z "$SCALA_BINARY_VERSION" ]; then
  SCALA_BINARY_VERSION=$(echo $SCALA_VERSION | cut -f 1-2 -d '.')
fi

# run ./gradlew copyDependantLibs to get all dependant jars in a local dir
shopt -s nullglob
if [ -z "$UPGRADE_KAFKA_STREAMS_TEST_VERSION" ]; then
  for dir in "$base_dir"/core/build/dependant-libs-${SCALA_VERSION}*;
  do
    CLASSPATH="$CLASSPATH:$dir/*"
  done
fi

for file in "$base_dir"/examples/build/libs/kafka-examples*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done

if [ -z "$UPGRADE_KAFKA_STREAMS_TEST_VERSION" ]; then
  clients_lib_dir=$(dirname $0)/../clients/build/libs
  streams_lib_dir=$(dirname $0)/../streams/build/libs
  streams_dependant_clients_lib_dir=$(dirname $0)/../streams/build/dependant-libs-${SCALA_VERSION}
else
  clients_lib_dir=/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs
  streams_lib_dir=$clients_lib_dir
  streams_dependant_clients_lib_dir=$streams_lib_dir
fi


for file in "$clients_lib_dir"/kafka-clients*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done

for file in "$streams_lib_dir"/kafka-streams*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done

if [ -z "$UPGRADE_KAFKA_STREAMS_TEST_VERSION" ]; then
  for file in "$base_dir"/streams/examples/build/libs/kafka-streams-examples*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$CLASSPATH":"$file"
    fi
  done
else
  VERSION_NO_DOTS=`echo $UPGRADE_KAFKA_STREAMS_TEST_VERSION | sed 's/\.//g'`
  SHORT_VERSION_NO_DOTS=${VERSION_NO_DOTS:0:((${#VERSION_NO_DOTS} - 1))} # remove last char, ie, bug-fix number
  for file in "$base_dir"/streams/upgrade-system-tests-$SHORT_VERSION_NO_DOTS/build/libs/kafka-streams-upgrade-system-tests*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$file":"$CLASSPATH"
    fi
  done
  if [ "$SHORT_VERSION_NO_DOTS" = "0100" ]; then
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zkclient-0.8.jar":"$CLASSPATH"
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zookeeper-3.4.6.jar":"$CLASSPATH"
  fi
  if [ "$SHORT_VERSION_NO_DOTS" = "0101" ]; then
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zkclient-0.9.jar":"$CLASSPATH"
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zookeeper-3.4.8.jar":"$CLASSPATH"
  fi
fi

for file in "$streams_dependant_clients_lib_dir"/rocksdb*.jar;
do
  CLASSPATH="$CLASSPATH":"$file"
done

for file in "$streams_dependant_clients_lib_dir"/*hamcrest*.jar;
do
  CLASSPATH="$CLASSPATH":"$file"
done

for file in "$base_dir"/tools/build/libs/kafka-tools*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done

for dir in "$base_dir"/tools/build/dependant-libs-${SCALA_VERSION}*;
do
  CLASSPATH="$CLASSPATH:$dir/*"
done

for cc_pkg in "api" "transforms" "runtime" "file" "mirror" "mirror-client" "json" "tools" "basic-auth-extension"
do
  for file in "$base_dir"/connect/${cc_pkg}/build/libs/connect-${cc_pkg}*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$CLASSPATH":"$file"
    fi
  done
  if [ -d "$base_dir/connect/${cc_pkg}/build/dependant-libs" ] ; then
    CLASSPATH="$CLASSPATH:$base_dir/connect/${cc_pkg}/build/dependant-libs/*"
  fi
done

# classpath addition for release
for file in "$base_dir"/libs/*;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done

for file in "$base_dir"/core/build/libs/kafka_${SCALA_BINARY_VERSION}*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done
shopt -u nullglob

if [ -z "$CLASSPATH" ] ; then
  echo "Classpath is empty. Please build the project first e.g. by running './gradlew jar -PscalaVersion=$SCALA_VERSION'"
  exit 1
fi

# JMX settings
if [ -z "$KAFKA_JMX_OPTS" ]; then
  KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false "
fi

# JMX port to use
if [  $JMX_PORT ]; then
  KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sun.management.jmxremote.port=$JMX_PORT "
fi

# Log directory to use
if [ "x$LOG_DIR" = "x" ]; then
  LOG_DIR="$base_dir/logs"
fi

# Log4j settings
if [ -z "$KAFKA_LOG4J_OPTS" ]; then
  # Log to console. This is a tool.
  LOG4J_DIR="$base_dir/config/tools-log4j.properties"
  # If Cygwin is detected, LOG4J_DIR is converted to Windows format.
  (( CYGWIN )) && LOG4J_DIR=$(cygpath --path --mixed "${LOG4J_DIR}")
  KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:${LOG4J_DIR}"
else
  # create logs directory
  if [ ! -d "$LOG_DIR" ]; then
    mkdir -p "$LOG_DIR"
  fi
fi

# If Cygwin is detected, LOG_DIR is converted to Windows format.
(( CYGWIN )) && LOG_DIR=$(cygpath --path --mixed "${LOG_DIR}")
KAFKA_LOG4J_OPTS="-Dkafka.logs.dir=$LOG_DIR $KAFKA_LOG4J_OPTS"

# Generic jvm settings you want to add
if [ -z "$KAFKA_OPTS" ]; then
  KAFKA_OPTS=""
fi

# Set Debug options if enabled
if [ "x$KAFKA_DEBUG" != "x" ]; then

    # Use default ports
    DEFAULT_JAVA_DEBUG_PORT="5005"

    if [ -z "$JAVA_DEBUG_PORT" ]; then
        JAVA_DEBUG_PORT="$DEFAULT_JAVA_DEBUG_PORT"
    fi

    # Use the defaults if JAVA_DEBUG_OPTS was not set
    DEFAULT_JAVA_DEBUG_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=${DEBUG_SUSPEND_FLAG:-n},address=$JAVA_DEBUG_PORT"
    if [ -z "$JAVA_DEBUG_OPTS" ]; then
        JAVA_DEBUG_OPTS="$DEFAULT_JAVA_DEBUG_OPTS"
    fi

    echo "Enabling Java debug options: $JAVA_DEBUG_OPTS"
    KAFKA_OPTS="$JAVA_DEBUG_OPTS $KAFKA_OPTS"
fi

# Which java to use
if [ -z "$JAVA_HOME" ]; then
  JAVA="java"
else
  JAVA="$JAVA_HOME/bin/java"
fi

# Memory options
if [ -z "$KAFKA_HEAP_OPTS" ]; then
  KAFKA_HEAP_OPTS="-Xmx256M"
fi

# JVM performance options
# MaxInlineLevel=15 is the default since JDK 14 and can be removed once older JDKs are no longer supported
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true"
fi

while [ $# -gt 0 ]; do
  COMMAND=$1
  case $COMMAND in
    -name)
      DAEMON_NAME=$2
      CONSOLE_OUTPUT_FILE=$LOG_DIR/$DAEMON_NAME.out
      shift 2
      ;;
    -loggc)
      if [ -z "$KAFKA_GC_LOG_OPTS" ]; then
        GC_LOG_ENABLED="true"
      fi
      shift
      ;;
    -daemon)
      DAEMON_MODE="true"
      shift
      ;;
    *)
      break
      ;;
  esac
done

# GC options	
GC_FILE_SUFFIX='-gc.log'
GC_LOG_FILE_NAME=''
if [ "x$GC_LOG_ENABLED" = "xtrue" ]; then
  GC_LOG_FILE_NAME=$DAEMON_NAME$GC_FILE_SUFFIX

  # The first segment of the version number, which is '1' for releases before Java 9
  # it then becomes '9', '10', ...
  # Some examples of the first line of `java --version`:
  # 8 -> java version "1.8.0_152"
  # 9.0.4 -> java version "9.0.4"
  # 10 -> java version "10" 2018-03-20
  # 10.0.1 -> java version "10.0.1" 2018-04-17
  # We need to match to the end of the line to prevent sed from printing the characters that do not match
  JAVA_MAJOR_VERSION=$("$JAVA" -version 2>&1 | sed -E -n 's/.* version "([0-9]*).*$/\1/p')
  if [[ "$JAVA_MAJOR_VERSION" -ge "9" ]] ; then
    KAFKA_GC_LOG_OPTS="-Xlog:gc*:file=$LOG_DIR/$GC_LOG_FILE_NAME:time,tags:filecount=10,filesize=100M"
  else
    KAFKA_GC_LOG_OPTS="-Xloggc:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M"
  fi
fi

# Remove a possible colon prefix from the classpath (happens at lines like `CLASSPATH="$CLASSPATH:$file"` when CLASSPATH is blank)
# Syntax used on the right side is native Bash string manipulation; for more details see
# http://tldp.org/LDP/abs/html/string-manipulation.html, specifically the section titled "Substring Removal"
CLASSPATH=${CLASSPATH#:}

# If Cygwin is detected, classpath is converted to Windows format.
(( CYGWIN )) && CLASSPATH=$(cygpath --path --mixed "${CLASSPATH}")

# Launch mode
if [ "x$DAEMON_MODE" = "xtrue" ]; then
  nohup "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@" > "$CONSOLE_OUTPUT_FILE" 2>&1 < /dev/null &
else
  exec "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@"
fi

```

脚本内容很长，但是实际上只有最后一部分才是真正在完成启动操作：

```sh
# Launch mode
if [ "x$DAEMON_MODE" = "xtrue" ]; then
  nohup "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@" > "$CONSOLE_OUTPUT_FILE" 2>&1 < /dev/null &
else
  exec "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@"
fi
```

# Launch modes

在脚本最后一段是有关启动方式的提示。

```sh
# Launch mode
if [ "x$DAEMON_MODE" = "xtrue" ]; then
  nohup "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@" > "$CONSOLE_OUTPUT_FILE" 2>&1 < /dev/null &
else
  exec "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@"
fi
```

这段脚本说明了之前的一大堆脚本都是为了这里启动赋值进行的一系列操作，这里根据传递参数判断是否守护进程的方式启动。这里以使用比较多的 **守护进程**启动方式进行参数介绍（实际上两者差别不算很大）。

## KAFKA_HEAP_OPTS

**KAFKA_HEAP_OPTS** 出自最开头，判断堆参数 **KAFKA_HEAP_OPTS**是否为空，为空就默认给它赋值为 "-Xmx1G -Xms1G"。

## KAFKA_JVM_PERFORMANCE_OPTS

这个值代表了JVM的启动参数。

```sh
# JVM performance options
# MaxInlineLevel=15 is the default since JDK 14 and can be removed once older JDKs are no longer supported
# MaxInlineLevel=15 是自JDK 14以来的默认值，一旦旧的JDK不再支持，就可以删除。
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true"
fi

while [ $# -gt 0 ]; do
  COMMAND=$1
  case $COMMAND in
    -name)
      DAEMON_NAME=$2
      CONSOLE_OUTPUT_FILE=$LOG_DIR/$DAEMON_NAME.out
      shift 2
      ;;
    -loggc)
      if [ -z "$KAFKA_GC_LOG_OPTS" ]; then
        GC_LOG_ENABLED="true"
      fi
      shift
      ;;
    -daemon)
      DAEMON_MODE="true"
      shift
      ;;
    *)
      break
      ;;
  esac
done
```

## G1垃圾收集器

Kafka默认使用G1的垃圾收集器，本身最低JDK版本要求就是JDK1.8。

```sh
-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true
```

### MaxGCPauseMillis

`-XX:MaxGCPauseMillis`：GC最大的停顿毫秒数，暂停时间默认值200ms，如果设置比这个小的值，G1收集器会尽可能的达到这个预期设置。

因为Kafka是非常激进的高并发分布式消息队列，为了获取更高的并发，使用20ms的极限值值尽可能的减少GC时间，最后用极短GC的代价换取高吞吐，当然结果会导致垃圾回收不干净。

但Kafka对于JVM本身的堆内存占用并不是很多，默认20ms的停顿时间其实是可以放心使用的。

此外从Kafka的设计来看，更频繁的GC是为了尽可能的触发Full Gc，因为Full Gc是回收Direct Memory的条件，而Kafka大量使用了页缓存提高数据的Log的读写速度，底层用的也是Java的Direct Memory。

### InitiatingHeapOccupancyPercent

这个参数实际上出入比较大，根据源码分析在**JDK8b12**版本之后，以及**JDK11** 之前这个参数和官方的文档描述，这个值的含义是符合“整堆”来计算是否触发Mixed Gc，但是**JDK8b12**版本之后更高的补丁，以及**JDK11**之后就变了，它变成根据**老年代占整堆的比重**。

这样的出入问题源自此参数的**源码BUG**，这部分涉及源码的探讨就不讨论了，具体可以看[关于G1收集器参数InitiatingHeapOccupancyPercent的正确认知 - 豆大侠的菜地 (doudaxia.club)](https://doudaxia.club/index.php/archives/212/)这篇大佬的文章分析。

这里直接给出结论：

- 如果你使用的JDK版本在8b12之前，XX:InitiatingHeapOccupancyPercent是**整个堆**使用量与**堆总体容量的**比值；
- 如果你使用的JDK版本在8b12之后（包括大版本9、10、11....），那么`XX:InitiatingHeapOccupancyPercent`是**老年代大小与堆总体容量的比值**这种说法和修改之后的JVM源码符合。

整体算是一个隐藏已久的BUG，因为G1的垃圾收集器设计角度看，它更关心的是`Old Region`占满整个堆空间之前提前尽可能的进行回收，而不是简单的看看剩余空间在整个堆空间的占比，因为剩余空间不是一个十分可靠的衡量值。

为了验证上文大佬的说法，个人也去参阅JDK8的Oracle文档：[java (oracle.com)](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

```java
-XX:c=percent
    Sets the percentage of the heap occupancy (0 to 100) at which to start a concurrent GC cycle. It is used by garbage collectors that trigger a concurrent GC cycle based on the occupancy of the entire heap, not just one of the generations (for example, the G1 garbage collector).

    By default, the initiating value is set to 45%. A value of 0 implies nonstop GC cycles. The following example shows how to set the initiating heap occupancy to 75%:

    -XX:InitiatingHeapOccupancyPercent=75
```

关键字 **entire heap**，也就是简单的剩余空间和整堆的占比。这里同样接着翻阅了一下，直到**JDK12**版本，这个描述还是和JDK8的版本一致的。直到阅读长期支持的**JDK17**的文档，发现里面的说法终于变了：

[Garbage-First (G1) Garbage Collector (oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html#GUID-DA6296DD-9AAB-4955-8B5B-683651936155)

> `XX:InitiatingHeapOccupancyPercent` determines the initial value as a percentage of the size of the current old generation as long as there aren't enough observations to make a good prediction of the Initiating Heap Occupancy threshold. Turn off this behavior of G1 using the option`-XX:-G1UseAdaptiveIHOP`. In this case, the value of `-XX:InitiatingHeapOccupancyPercent` always determines this threshold.。
> “XX：启动堆占用百分比”将初始值确定为**当前老一代大小的百分比**，只要没有足够的观测值来很好地预测起始堆占用阈值。
> 使用选项'`-XX:-G1UseAdaptiveIHOP`'关闭G1的此行为。在这种情况下`-XX:InitiatingHeapOccupancyPercent`  启动堆占用百分比'的值始终确定此阈值。

所以这个值的真实含义和使用的JDK版本有关，并且JDK8的后续补丁版本也修复了这个问题，所以最终建议是升级JDK8的补丁版本，或者使用JDK11之后的版本。

### -XX:+ExplicitGCInvokesConcurrent

看似简单的参数，实际上又是隐藏这非常多的“坑”和细节，这里我们划分更多的小节慢慢细品。

#### 简单理解

这个参数是指通过使用`System.gc()`请求**启用并发 GC 的调用**，**默认禁用**。如果没有特殊的应用场景，大部分情况下这个参数都是被建议禁用的，而并发GC实际上就是CMS的并发回收处理。

个人在官方文档中搜到类似的参数描述：[Garbage-First Garbage Collector Tuning (oracle.com)](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector-tuning.htm#JSGCT-GUID-0BB3B742-A985-4D5E-A9C5-433A127FE0F6)。

> Other causes than Allocation Failure for a Full GC typically indicate that either the application or some external tool causes a full heap collection. If the cause is , and there is no way to modify the application sources, the effect of Full GCs can be mitigated by using or let the VM completely ignore them by setting . **External tools may still force Full GCs; they can be removed only by not requesting them.System.gc()-XX:+ExplicitGCInvokesConcurrent -XX:+DisableExplicitGC**

上面一大段的话大意指的是：阻止外部调用Full GC（也就是`System.gc()`）要么直接设置`-XX:+DisableExplicitGC`，要么设置`-XX:+ExplicitGCInvokesConcurrent`提高强制Full Gc的效率，阅读源码发现这两个参**数不能一起开启**，因为`-XX:+ExplicitGCInvokesConcurrent`需要关闭`-XX:+DisableExplicitGC`参数才能生效。

> 部分文章也解释仅仅建议在G1的垃圾收集器中可以使用`-XX:+ExplicitGCInvokesConcurrent`。其他垃圾收集器不建议使用。

####  Kafka官方修复BUG：-XX:+DisableExplicitGC 改为 -XX:+ExplicitGCInvokesConcurrent

为什么两者只能选其一使用，JDK 8 的JVM中存在类似的代码可以给予解释。

```c
bool GenCollectedHeap::should_do_concurrent_full_gc(GCCause::Cause cause) {
    // 检查参数 -XX:+DisableExplicitGC 和 -XX:+ExplicitGCInvokesConcurrent
  return UseConcMarkSweepGC &&
         ((cause == GCCause::_gc_locker && GCLockerInvokesConcurrent) ||
         // -XX:+ExplicitGCInvokesConcurrent 需要满足不配置-XX:+DisableExplicitGC的条件，才能判定为true
          (cause == GCCause::_java_lang_system_gc && ExplicitGCInvokesConcurrent));
}

void GenCollectedHeap::collect(GCCause::Cause cause) {
    // 检查参数 -XX:+DisableExplicitGC 和 -XX:+ExplicitGCInvokesConcurrent
  if (should_do_concurrent_full_gc(cause)) {
#ifndef SERIALGC
    // mostly concurrent full collection
    collect_mostly_concurrent(cause);
#else  // SERIALGC
    ShouldNotReachHere();
#endif // SERIALGC
  } else {
#ifdef ASSERT
    if (cause == GCCause::_scavenge_alot) {
      // minor collection only
      collect(cause, 0);
    } else {
      // Stop-the-world full collection
      // STW 进行Full Gc
      collect(cause, n_gens() - 1);
    }
#else
    // Stop-the-world full collection
    collect(cause, n_gens() - 1);
#endif
  }
}
```

collect里一开头就有个判断，如果`should_do_concurrent_full_gc`返回true，那会执行collect_mostly_concurrent做并行的回收。

回到Kafka的服务端参数，KafKa最初的服务端启动脚本中，此参数实际为`-XX:+DisableExplicitGC`，但是后续被指出会影响直接内存的回收性能，并且很可能会导致**直接内存无法被回收！** 

为什么会有这么严重 ? 这里先不急着分析，而是先看看作者的这个issue的提交：

[KAFKA-5470: Replace -XX:+DisableExplicitGC with -XX:+ExplicitGCInvokesConcurrent in kafka-run-class by ijuma · Pull Request #3371 · apache/kafka (github.com)](https://github.com/apache/kafka/pull/3371)

提交者的原话是：

> This is important because **Bits.reserveMemory calls System.gc()** hoping to free native
> memory in order to avoid throwing an OutOfMemoryException. This call is currently
> a no-op due to -XX:+DisableExplicitGC.
>
> It's worth mentioning that -XX:MaxDirectMemorySize can be used to increase the
> amount of native memory available for allocation of direct byte buffers.

简单来说就是**Bits.reserveMemory**里面会有`System.gc()`调用，通过程序强制调用Full Gc来回收掉native内存，所以建议在JVM参数中删掉`-XX:+DisableExplicitGC`，开启`System.gc();`并且通过添加`-XX:+ExplicitGCInvokesConcurrent`让`System.gc()`调用效率更高一些。

>  另外大佬这里还提了一嘴`-XX:MaxDirectMemorySize`可以用来提高可用于分配直接字节缓冲区的本地内存的数量。
>  大佬一句话就是一个知识点，牛呀。

####  Bits#reserveMemory

既然提交者提到了`Bits#reserveMemory`，这里就顺带贴一下官方jdk8的**java.nio.Bits#reserveMemory**源码方便理解：

```java
// These methods should be called whenever direct memory is allocated or
// freed.  They allow the user to control the amount of direct memory
// which a process may access.  All sizes are specified in bytes.

//每当分配直接内存或释放。 它们允许用户控制直接内存的数量进程可以访问的内容。 所有大小均以字节为单位指定。

static void reserveMemory(long size, int cap) {

    if (!memoryLimitSet && VM.isBooted()) {
        maxMemory = VM.maxDirectMemory();
        memoryLimitSet = true;
    }

    // optimist!
    if (tryReserveMemory(size, cap)) {
        return;
    }

    final JavaLangRefAccess jlra = SharedSecrets.getJavaLangRefAccess();

    // retry while helping enqueue pending Reference objects
    // which includes executing pending Cleaner(s) which includes
    // Cleaner(s) that free direct buffer memory
    while (jlra.tryHandlePendingReference()) {
        if (tryReserveMemory(size, cap)) {
            return;
        }
    }

    // trigger VM's Reference processing
    System.gc();

    // a retry loop with exponential back-off delays
    // (this gives VM some time to do it's job)
    boolean interrupted = false;
    try {
        long sleepTime = 1;
        int sleeps = 0;
        while (true) {
            if (tryReserveMemory(size, cap)) {
                return;
            }
            if (sleeps >= MAX_SLEEPS) {
                break;
            }
            if (!jlra.tryHandlePendingReference()) {
                try {
                    Thread.sleep(sleepTime);
                    sleepTime <<= 1;
                    sleeps++;
                } catch (InterruptedException e) {
                    interrupted = true;
                }
            }
        }

        // no luck
        throw new OutOfMemoryError("Direct buffer memory");

    } finally {
        if (interrupted) {
            // don't swallow interrupts
            Thread.currentThread().interrupt();
        }
    }
}
```

我们通过阅读JDK8的Nio包的这部分用于分配DirectMememory的一段代码，发现**每次**Direct Mememory进行实际的分配动作之前，都会调用这个方法检测是否有足够空间分配时都被调用，不过里面的逻辑奇奇怪怪的，初看确实有点摸不着头脑。

国外有网友直接痛骂了这一段代码是一坨Shit：[java.nio.Bits.reserveMemory uses a lock, calls System.gc, and is generally bad code... (google.com)](https://groups.google.com/g/mechanical-sympathy/c/20ajjb-YULw?pli=1)

```java
1.  ALL memory access requires a lock.  That's evil if you're allocating small chunks.

2.  The code to change the reserved memory counters is duplicated twice.  This is a great way to introduce bugs.  (how did this even get approved? do they not do code audits or require that commits be approved?)

3.  If you are out of memory we call System.gc... EVIL.  The entire way direct memory is reclaimed via GC is a horrible design.

4.  After GC they sleep 100ms.  What's that about?  Why 100ms?  Why not 1ms?  
```

1.  所有的内存访问都需要一个锁。 如果你分配的是小块的内存简直就是噩梦。
2.  改变保留内存计数器的代码重复了两次。 这是个引入错误的好方法。 (难道他们不进行代码审计或要求提交的代码必须得到批准吗？）
3.  如果你没有内存了，我们就调用System.gc...  通过GC回收直接内存的整个方式是一个可怕的设计。
4.  在GC之后，他们会休眠100ms。 那是什么意思？ 为什么是100ms？ 为什么不是1ms？ 

个人并不感冒这些评论，这里拎出`System.gc()`这行代码来分析具体意图。要看懂这一行代码的意图，我们需要了解DirectMemory关联的本机内存是如何清理的，这里就直接给出答案了。

JVM实际上是管不到DirectMemory的，需要依靠特殊的方式回收掉DirectMemory：

- 手动调用`unsafe.freeMemory()`进行释放，**netty**中`ByteBuf.release()`就是这种方式实现的；
- 利用**GC机制**在GC的过程中自动调用`unsafe.freeMemory()`释放被引用的直接内存；

这段代码作者的意图明显是显示调用`System.gc()`，尽可能回收不可达的`DirectByteBuffer`对象，也只有通过GC才会自动触发`unsafe.freeMemory()`的调用，释放直接内存。

至于其他代码.....这里不做过多评论。

####  Fix -XX:+DisableExplicitGC

基于以上种种原因，Kafka官方最终提交了一个Commit修复这个问题：

[Fix run class to work with Java 10 and use ExplicitGCInvokesConcurrent by ijuma · Pull Request #1329 · confluentinc/ksql (github.com)](https://github.com/confluentinc/ksql/pull/1329)

具体的调整细节可以看下面的连接，读者可以通过对比自己的下载Kafka启动脚本查看是否修复这个问题：

[KAFKA-5470: Replace -XX:+DisableExplicitGC with -XX:+ExplicitGCInvokesConcurrent in kafka-run-class by ijuma · Pull Request #3371 · apache/kafka (github.com)](https://github.com/apache/kafka/pull/3371/commits/acdd6e978c9be123cffafc16d1e84942d6cd4477)

####  其他参考资料

下面的这些参考资料可以帮助我们更深入的理解`-XX:+ExplicitGCInvokesConcurrent`参数附带的知识点：

-XX:+ExplicitGCInvokesConcurrent的含义：[What is JVM startup parameter: -XX:+ExplicitGCInvokesConcurrent? - yCrash Answers](https://answers.ycrash.io/question/what-is-jvm-startup-parameter--xxexplicitgcinvokesconcurrent?q=795)

官方的G1文档：[Java HotSpot Garbage Collection (oracle.com)](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html)

为什么仅限G1可以开启此参数来进行健康检查，其他垃圾收集器建议关闭此参数：[Health Check: Explicit Garbage Collection | Jira | Atlassian Documentation](https://confluence.atlassian.com/jirakb/health-check-explicit-garbage-collection-976778100.html)

[JVM源码分析之SystemGC完全解读 | HeapDump性能社区](https://heapdump.cn/article/154235)
- 为什么不能同时设置`-XX:+DisableExplicitGC `以及` -XX:+ExplicitGCInvokesConcurrent`
- 为什么CMS GC下-XX:+ExplicitGCInvokesConcurrent这个参数加了之后会比真正的Full GC好？
- 它如何做到暂停整个进程？
- 堆外内存分配为什么有时候要配合System.gc？
- Netty回收堆外内存的策略又是如何？

#### 小结

笔者也没有想到一个简单的参数能牵扯出这么多内容，这里做一个大概的总结：
- Kafka官方曾经禁用过`System.gc()`。
- 后面有大神分析了脚本和JDK的NIO源码，发现禁用`System.gc()`这不是有问题嘛，你Kafka大量使用Java的直接内存，直接内存靠一般的Gc是回收不掉的，只能靠Ful Gc顺带回收，JDK官方代码又是靠频繁调用`System.gc()`强制腾出直接内存空间的，你`System.gc()`禁用了不是“找死”么，于是赶紧解释了一波` Bits#reserveMemory`写的“垃圾代码”来证实自己的观点，然后建议启用`System.gc()`，并且为了提高Full Gc效率使用`-XX:+ExplicitGCInvokesConcurrent`。
- 官方发现这个问题赶紧修复了一版并且提交了issue。

### MaxInlineLevel

`java` 有一个参数 `-XX:MaxInlineLevel`(JDK14之前默认值为 9)，这个值在JDK14之后默认值改为15。这个值的修改可以参考JDK官方的声明 [https://bugs.openjdk.org/browse/JDK-8234863]( https://bugs.openjdk.org/browse/JDK-8234863)。

> 下面的图Oracle官方对于JDK14版本之后修改`MaxInlineLevel=15`的Push。

![调整MaxInlineLevel为15](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230223221451.png)

下面长篇大论源自网上收集的资料和个人理解，其实简单理解为现代硬件资源足以支持 `-XX:MaxInlineLevel`设置为15，更大的内联深度可以让JIT编译出更多的本地代码从而提高Java代码的运行效率即可。

> 如果你的服务器还是古旧的四五年前的机器，或者生产机器确实渣的可以，那么还是建议把这个参数 `-XX:MaxInlineLevel`改回 9 比较妥当。

长篇大论的部分：

链接[https://bugs.openjdk.org/browse/JDK-8234863]( https://bugs.openjdk.org/browse/JDK-8234863)的声明指出，15这个值在scala上的性能测试是被认为最优结果。这个值在现代处理器速度以及性能优化较好的今天最为合适，默认值9这个数字显得非常**过时**。

Kafka作为激进压榨机器性能的典范，也遵从JDK官方的改动默认所有版本的JDK统一使用15这个默认值。

> 这里额外插一嘴，个人认为实际上这个值Oracle官方在JDK11就可以修改为15。

MaxInlineLevel本身的判断逻辑似乎更引起广大程序员的关注，StackFlow上有一篇关于这个参数的讨论：[https://stackoverflow.com/questions/32503669/why-does-the-jvm-have-a-maximum-inline-depth](https://stackoverflow.com/questions/32503669/why-does-the-jvm-have-a-maximum-inline-depth ) 比较有意思。

在评论中有网友指出在比较低的JDK8版本当中，MaxRecursiveInlineLevel对**直接**和**间接**的递归调用都进行计数，编译后的代码应该在运行时保持对整个内联树的跟踪（以便能够解压和去优化）。紧接着是其他人的一些个人观点，没人接这个人话茬=-=，尴尬。

继续翻阅，下面的评论有一位大佬解释了为什么会出现MaxInlineLevel这个参数，简单易懂这里就直接贴过来了：

> One reason is also that the inlining itself in the HotSpot JVM is implemented with recursion. Every time inlining of a method is started a new context is created on the native stack. Allowing an unlimited depth would eventually make the JIT-compiler crash when it runs out of stack. 

**（旧版保守的限制方法内联深度），其中一个原因是HotSpot JVM的内联本身是用递归实现的。每次对一个方法进行内联时，都会在本地堆栈中创建一个新的上下文。如果允许无限的深度，最终会使JIT-编译器在堆栈耗尽时崩溃。**

在过去硬件资源紧张的情况下，过度的方法内联有可能会出现比较深的堆栈调用，十分消耗程序内存，但是现代内存动不动就是32，64，128G 的今天，加上处理器的核心数量上来了之后，扩大默认的方法内联深度参数值确实非常有必要。

>  方法内联是JVM比较底层的优化，可以通过周大神的《深入理解JVM虚拟机第三版》了解。

如果不懂方法内联直接无脑设置`MaxInlineLevel=15`即可，没有为什么，官方都已经在高版本JDK修改了默认值，JDK8忠实粉丝自然也可以这么干。

jdk14 hotspot 依赖的调整日志：[https://hg.openjdk.org/jdk8u/jdk8u/hotspot/](https://hg.openjdk.org/jdk8u/jdk8u/hotspot/)

### -Djava.awt.headless=true

这个参数比较奇怪，但是实际上在SpringBoot源码中也有同样的写法。

这算是一个不太被关注的优化参数，简单理解是`-Djava.awt.headless=true`可以屏蔽掉一些不必要的外置设备影响，告知程序当前没有外置设备，尽可能的让程序底层自己模拟，比如打印从图形显示变为控制台打印。

又是牵扯内容很多的一个点，具体解释可以看这篇文章：[[【Java】The Java Headless Mode]]，篇幅有限，这里就不多解释了。

## KAFKA_GC_LOG_OPTS

见名知意，就是JVM的日志参数配置，Kafka最终的日志格式为：`XXX-gc.log`，日志配置这一块和大部分以JAVA为底层的开源组件大差不差，简单的扫一眼差不多了。

```sh
# Log directory to use
# 获取log_dir，如果没配置就那 $base_dir 环境变量
if [ "x$LOG_DIR" = "x" ]; then
  # base_dir=$(dirname $0)/..
  LOG_DIR="$base_dir/logs"
fi


# GC options	
GC_FILE_SUFFIX='-gc.log'
GC_LOG_FILE_NAME=''
if [ "x$GC_LOG_ENABLED" = "xtrue" ]; then
  GC_LOG_FILE_NAME=$DAEMON_NAME$GC_FILE_SUFFIX

  # The first segment of the version number, which is '1' for releases before Java 9
  # it then becomes '9', '10', ...
  # Some examples of the first line of `java --version`:
  # 8 -> java version "1.8.0_152"
  # 9.0.4 -> java version "9.0.4"
  # 10 -> java version "10" 2018-03-20
  # 10.0.1 -> java version "10.0.1" 2018-04-17
  # We need to match to the end of the line to prevent sed from printing the characters that do not match
  JAVA_MAJOR_VERSION=$("$JAVA" -version 2>&1 | sed -E -n 's/.* version "([0-9]*).*$/\1/p')
  if [[ "$JAVA_MAJOR_VERSION" -ge "9" ]] ; then
    KAFKA_GC_LOG_OPTS="-Xlog:gc*:file=$LOG_DIR/$GC_LOG_FILE_NAME:time,tags:filecount=10,filesize=100M"
  else
    KAFKA_GC_LOG_OPTS="-Xloggc:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M"
  fi
fi
```

**JAVA_MAJOR_VERSION**就是通过正则去除JDK的主版本号。

- 如果主版本号大于或者等于JDK9，就使用JDK9新增的 `-xlog:gc*` 统一的日志参数作为启动参数，
- 如果是JDK8之前的版本，就需要用一大堆旧版的日志参数，学习和使用成本比较大：
  - GCLogFileSize=100M，限制GC日志文件大小为100M。
  - NumberOfGCLogFiles=10，允许存在的GC日志文件数量为10个。
  - UseGCLogFileRotation，让GC日志不断循环，如果最后一个GC日志写满，将会从第一个文件重新开始写入
  - `-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps` 是旧版本混乱的GC参数配置诞生的恶果，这些参数在JDK9之后被统统被`-xlog:gc*` 替代。
    - `-verbose:gc -XX:+PrintGCDetails`这两个参数经常在低版本JDK一起出现，最大的区别是前者是稳定版本，后者则是被认为是不稳定的日志启动参数（强制和其他GC参数配合出现显得不稳定）。
    - `-XX:+PrintGCDateStamps`：每行开头显示当前绝对的日期及时间，打印GC发生时的时间戳，搭配 -XX:+PrintGCDetails 使用，不可以独立使用。
    - `-XX:+PrintGCTimeStamps` **自从JVM启动**以来的时间。

`-XX:+PrintGCDateStamps`和`-XX:+PrintGCTimeStamps`可以直接看下面的例子对比：

```java
-XX:+PrintGCDateStamps
日志输出示例：
2014-01-03T12:08:38.102-0100: [GC 66048K->53077K(251392K), 0,0959470 secs]
2014-01-03T12:08:38.239-0100: [GC 119125K->114661K(317440K), 0,1421720 secs]

-XX:+PrintGCTimeStamps
日志输出示例：
0,185: [GC 66048K->53077K(251392K), 0,0977580 secs]
0,323: [GC 119125K->114661K(317440K), 0,1448850 secs]
```

> 因为`-XX:+PrintGCDetails`被标记为manageable，所以可以通过如下三种方式修改：
> 1、com.sun.management.HotSpotDiagnosticMXBean API
> 2、JConsole
> 3、jinfo -flag

最后再把英文注释部分简单翻译一下：

1. 第一个参数如果是1开头，代表是JDK9之后的版本。
2. `java --version`产生的结果如下：
   - 8 -> java version "1.8.0_152"
   - 9.0.4 -> java version "9.0.4"
   - 10 -> java version "10" 2018-03-20
   - 10.0.1 -> java version "10.0.1" 2018-04-17
3. 通过正则表达式匹配到行尾，以防止sed打印出不匹配的字符


## KAFKA_JMX_OPTS

JMX全称Java Management Extensions, 为Java应用提供管理扩展功能。在JDK 5的时候引入，Kafka设置启动参数让Kafka应用程序获得JMX远程调用的支持。

```sh
# JMX settings
if [ -z "$KAFKA_JMX_OPTS" ]; then
  KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false "
fi
```

KAFKA_JMX_OPTS对应的value的含义参考自：[https://www.jianshu.com/p/414647c1179e](https://www.jianshu.com/p/414647c1179e),此处列举一些有关JMX的相关参数：
| 参数名                                        | 类型 | 描述                                                         |
| :-------------------------------------------- | :--- | :----------------------------------------------------------- |
| -Dcom.sun.management.jmxremote                | 布尔 | 是否支持远程JMX访问，默认true                                |
| -Dcom.sun.management.jmxremote.port           | 数值 | 监听端口号，方便远程访问                                     |
| -Dcom.sun.management.jmxremote.authenticate   | 布尔 | 是否需要开启用户认证,默认开启                                |
| -Dcom.sun.management.jmxremote.ssl            | 布尔 | 是否对连接开启SSL加密，默认开启                              |
| -Dcom.sun.management.jmxremote.access.file    | 路径 | 对访问用户的权限授权的文件的路径，默认路径`JRE_HOME/lib/management/jmxremote.access` |
| -Dcom.sun.management.jmxremote. password.file | 路径 | 设置访问用户的用户名和密码，默认路径`JRE_HOME/lib/management/ jmxremote.password` |


## KAFKA_LOG4J_OPTS

log4j的日志配置地址。

```java
if [ "x$KAFKA_LOG4J_OPTS" = "x" ]; then  
    export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$base_dir/../config/log4j.properties"  
fi  
```

 配置含义不需要记忆，在阅读的时候查阅相关资料即可：[https://www.jianshu.com/p/ccafda45bcea](https://www.jianshu.com/p/ccafda45bcea)，这里直接贴过来作为注释部分供读者参考。

```java
# 根Log
# 默认日志等级为INFO级别
# NFO、WARN、ERROR和FATAL级别的日志信息都会输出
# 日志最终输出到kafkaAppender
log4j.rootLogger=INFO, stdout, kafkaAppender  
# 控制台配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
# 布局模式使用可以灵活模式
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
# 日志打印格式
# [%d] 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
# %m:：输出代码中指定的具体日志信息。
# %p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
# %c：输出日志信息所属的类目，通常就是所在类的全名。
# %n：输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"。
log4j.appender.stdout.layout.ConversionPattern=[%d] %p %m (%c)%n  

# DailyRollingFileAppender Kafka默认服务端日志
log4j.appender.kafkaAppender=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.kafkaAppender.DatePattern='.'yyyy-MM-dd-HH  
# server.log 存储位置
log4j.appender.kafkaAppender.File=${kafka.logs.dir}/server.log  
log4j.appender.kafkaAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.kafkaAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  

# DailyRollingFileAppender 状态机变更日志
log4j.appender.stateChangeAppender=org.apache.log4j.DailyRollingFileAppender  
# 按照每小时产生一个日志的方式
log4j.appender.stateChangeAppender.DatePattern='.'yyyy-MM-dd-HH   
log4j.appender.stateChangeAppender.File=${kafka.logs.dir}/state-change.log  
log4j.appender.stateChangeAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.stateChangeAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  

# 请求日志
log4j.appender.requestAppender=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.requestAppender.DatePattern='.'yyyy-MM-dd-HH  
log4j.appender.requestAppender.File=${kafka.logs.dir}/kafka-request.log  
log4j.appender.requestAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.requestAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  

# Log清理日志
log4j.appender.cleanerAppender=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.cleanerAppender.DatePattern='.'yyyy-MM-dd-HH  
log4j.appender.cleanerAppender.File=${kafka.logs.dir}/log-cleaner.log  
log4j.appender.cleanerAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.cleanerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  

# Controller 日志
log4j.appender.controllerAppender=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.controllerAppender.DatePattern='.'yyyy-MM-dd-HH  
log4j.appender.controllerAppender.File=${kafka.logs.dir}/controller.log  
log4j.appender.controllerAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.controllerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  

# 验证日志
log4j.appender.authorizerAppender=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.authorizerAppender.DatePattern='.'yyyy-MM-dd-HH  
log4j.appender.authorizerAppender.File=${kafka.logs.dir}/kafka-authorizer.log  
log4j.appender.authorizerAppender.layout=org.apache.log4j.PatternLayout  
log4j.appender.authorizerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n  
  
# Change the line below to adjust ZK client logging  
# 修改下面的日志控制ZK的日志输出
log4j.logger.org.apache.zookeeper=INFO  
  
# Change the two lines below to adjust the general broker logging level (output to server.log and stdout)  
# 更改下面两行以调整一般代理日志记录级别（输出到 server.log 和 stdout）
log4j.logger.kafka=INFO  
log4j.logger.org.apache.kafka=INFO  
  
# Change to DEBUG or TRACE to enable request logging  
# 修改日志级别为 DEBUG和TRACE获取请求日志
log4j.logger.kafka.request.logger=WARN, requestAppender  
log4j.additivity.kafka.request.logger=false  
  
# Uncomment the lines below and change log4j.logger.kafka.network.RequestChannel$ to TRACE for additional output  
# 取消注释下面的行并将 log4j.logger.kafka.network.RequestChannel$ 更改为 TRACE 以获得额外的输出
# related to the handling of requests  
# 与请求的处理相关
#log4j.logger.kafka.network.Processor=TRACE, requestAppender  
#log4j.logger.kafka.server.KafkaApis=TRACE, requestAppender  
#log4j.additivity.kafka.server.KafkaApis=false  

log4j.logger.kafka.network.RequestChannel$=WARN, requestAppender  
log4j.additivity.kafka.network.RequestChannel$=false  
  
log4j.logger.kafka.controller=TRACE, controllerAppender  
log4j.additivity.kafka.controller=false  
  
log4j.logger.kafka.log.LogCleaner=INFO, cleanerAppender  
log4j.additivity.kafka.log.LogCleaner=false  
  
log4j.logger.state.change.logger=INFO, stateChangeAppender  
log4j.additivity.state.change.logger=false  
  
# Access denials are logged at INFO level, change to DEBUG to also log allowed accesses 
# 拒绝访问记录在 INFO 级别，更改为 DEBUG 以记录允许的访问
log4j.logger.kafka.authorizer.logger=INFO, authorizerAppender  
log4j.additivity.kafka.authorizer.logger=false
```

## KAFKA_OPTS

KAFKA_OPTS 可以在这里设置自己的想要的通用配置：

```sh
# Generic jvm settings you want to add
if [ -z "$KAFKA_OPTS" ]; then
  KAFKA_OPTS=""
fi
```

## UPGRADE_KAFKA_STREAMS_TEST_VERSION

变量名称翻译过来是“升级kafka流的测试版本”，这里大致的意思是取出版本号进行一些判断之后设置到ClassPath当中。

说实话这部分内容看不太懂，但是不算是十分重要的东西，可以以后深入之后回来了解，这里直接忘记这个设置即可。

```sh
if [ -z "$UPGRADE_KAFKA_STREAMS_TEST_VERSION" ]; then
  for file in "$base_dir"/streams/examples/build/libs/kafka-streams-examples*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$CLASSPATH":"$file"
    fi
  done
else
  VERSION_NO_DOTS=`echo $UPGRADE_KAFKA_STREAMS_TEST_VERSION | sed 's/\.//g'`
  SHORT_VERSION_NO_DOTS=${VERSION_NO_DOTS:0:((${#VERSION_NO_DOTS} - 1))} # remove last char, ie, bug-fix number
  for file in "$base_dir"/streams/upgrade-system-tests-$SHORT_VERSION_NO_DOTS/build/libs/kafka-streams-upgrade-system-tests*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$file":"$CLASSPATH"
    fi
  done
  if [ "$SHORT_VERSION_NO_DOTS" = "0100" ]; then
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zkclient-0.8.jar":"$CLASSPATH"
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zookeeper-3.4.6.jar":"$CLASSPATH"
  fi
  if [ "$SHORT_VERSION_NO_DOTS" = "0101" ]; then
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zkclient-0.9.jar":"$CLASSPATH"
    CLASSPATH="/opt/kafka-$UPGRADE_KAFKA_STREAMS_TEST_VERSION/libs/zookeeper-3.4.8.jar":"$CLASSPATH"
  fi
fi
```

## CLASSPATH

运行 `./gradlew copyDependantLibs` 来获取本地目录下的所有依赖性jar。

注意这里划分了很多个子模块，所以使用了for循环加载到**CLASSPATH**当中，这会导致最终产生的命令会非常长。

```sh
# run ./gradlew copyDependantLibs to get all dependant jars in a local dir
shopt -s nullglob
if [ -z "$UPGRADE_KAFKA_STREAMS_TEST_VERSION" ]; then
  for dir in "$base_dir"/core/build/dependant-libs-${SCALA_VERSION}*;
  do
    CLASSPATH="$CLASSPATH:$dir/*"
  done
fi

for file in "$base_dir"/examples/build/libs/kafka-examples*.jar;
do
  if should_include_file "$file"; then
    CLASSPATH="$CLASSPATH":"$file"
  fi
done
```

个人尝试了一下注释介绍的`gradle copyDependantLibs`命令，本地执行结果如下，这个命令会在对应的模块构建依赖jar包：

```sh
$ gradle copyDependantLibs

> Configure project :
Building project 'core' with Scala version 2.13.3
Building project 'streams-scala' with Scala version 2.13.3

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.6.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 2s
55 actionable tasks: 3 executed, 52 up-to-date

```

个人是win11的电脑，通过wox查找`dependant-libs`结果如下：

![image-20230224133728509](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/image-20230224133728509.png)

对应的一堆依赖jar包

![image-20230224133744539](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/image-20230224133744539.png)


## CONSOLE_OUTPUT_FILE

日志的打印输出地址文件地址设置，注意不是GC的日志。

```sh
  case $COMMAND in
    -name)
      DAEMON_NAME=$2
      CONSOLE_OUTPUT_FILE=$LOG_DIR/$DAEMON_NAME.out
      shift 2
      ;;
```

这里翻阅了一些有关`shift`的资料：

> 关于Shift的作用可以参考：[https://ss64.com/bash/shift.html](https://ss64.com/bash/shift.html)。Linux中通过`help shift`查看使用手册，但是会发现写的比较潦草和抽象。

```sh
shift: shift [n]
    Shift positional parameters.
    
    Rename the positional parameters $N+1,$N+2 ... to $1,$2 ...  If N is
    not given, it is assumed to be 1.
    
    Exit Status:
    Returns success unless N is negative or greater than $#
```

比如下面的程序：

```sh
#! /bin/bash

echo $1 
echo $2

shift 1

echo $1
echo $2

# 输出结果
[zxd@localhost ~]$ ./test.sh 1 2 3 5 6
1
2

2
3
```

**shift 1 执行之后会弹出第一个参数**，之后的运行参数会往前“推动”，$1变为$2的值，$2变为$3的值，以此类推。

## & 后台启动和nohup挂起进程

末尾部分是设置ClaassPath，用户自己自定义参数以及把标准输入和输出重定向到同一个位置，最后就是以后台模式启动并且最终通过nohup挂起整个进程。

```sh
-cp "$CLASSPATH" $KAFKA_OPTS "$@" > "$CONSOLE_OUTPUT_FILE" 2>&1 < /dev/null &
```

这段脚本最前面把替代参数意义进行了替换。`-cp`是nohup命令的参数，接着是把输出的结果全部重定向到标准输出当中，这个地址对应**CONSOLE_OUTPUT_FILE**。

下面理解最后部分的`nohup和&`、`2>&1`和`/dev/null`这几个常见的服务端脚本启动参数的含义。

### nohup和&

nohup：nohup指令会忽略所有挂断（SIGHUP）信号不挂断的运行。注意nohup命令本身并没有后台运行的功能，需要配合`&`使用。它的实现原理是让命令不间断的运行实现挂机的效果。

& 是指在后台运行，但当用户退出（解除挂起）的时候，命令自动也跟着退出，`nohup`和`&`这两个指令通常会放到一起使用。

### /dev/null

这个空间属于Linux的一块特殊空间，UNIX系统中，它被称为空设备。以下内容摘自维基百科：

>  /dev/null（或称空设备）在类Unix系统中是一个特殊的设备文件，它丢弃一切写入其中的数据（但报告写入操作成功），读取它则会立即得到一个EOF[1]。
>
> 在程序员行话，尤其是Unix行话中，/dev/null被称为比特桶或者**黑洞**。

### 2>&1的问题

前面的第一个数字2通常对应下面几种含义：

- 0 – stdin (standard input) 标准输入
- 1 – stdout (standard output) 标准输出
- 2 – stderr (standard error) 标准错误输出

\> 是重定向符号，而数字2的含义是标准错误输出，`&1`指的就是标准输出，三个符号组合到一起就是把标准错误输出输入重定向到标准输出当中，这里可以理解为“合流”。

> 注意命令`2>&1` 和`2>1`是存在区别的，这里`&`不能丢，后者的1代表输出代表错误重定向到一个**文件1**，不代表标准输出，只有`&1`才代表标准输出。

如果想要丢弃所有的标准错误输出和标准输出结果，下面是一个不错的例子：

```sh
nohup python3 getfile.py > /dev/null 2>&1 &
```

如果想要写入到指定的位置，下面是又一个不错的例子：

```sh
nohup python3 getfile.py > test.log 2>&1 &
```

最后是实际一点的例子：

```sh
0 9 \* \* \* /usr/bin/python3 /opt/getFile.py > /opt/file.log 2>&1
```

> 上面的命令含义是放在crontab中的定时任务，每天9:00启动这个python的脚本，并把执行结果写入日志文件file.log中

## exec 运行

如果不是守护进程的执行，则是使用`exec`在当前的shell中进行正常模式启动，此时整个shell会挂起运行kafka服务端。

```sh
  exec "$JAVA" $KAFKA_HEAP_OPTS $KAFKA_JVM_PERFORMANCE_OPTS $KAFKA_GC_LOG_OPTS $KAFKA_JMX_OPTS $KAFKA_LOG4J_OPTS -cp "$CLASSPATH" $KAFKA_OPTS "$@"
```

# 其他内容

Lauch modes 使用到的变量设置包含了启动整个Kafka服务端的核心部分，下面再列觉其他的依赖配置以及“辅助”内容。

## Scala 版本选择

Kafka是使用Java和Scala混合编写的，根据不同的Kafka版本需要不同版本的Scala版本支持，这里官方做了一个版本选择强制判断选择出最合适的Scala。

```sh
if [ -z "$SCALA_VERSION" ]; then
  SCALA_VERSION=2.13.3
  if [[ -f "$base_dir/gradle.properties" ]]; then
    SCALA_VERSION=`grep "^scalaVersion=" "$base_dir/gradle.properties" | cut -d= -f 2`
  fi
fi
```

### gradle.properties

Kafka 项目是基于gradle构建的，gradle 个人平时基本没啥接触机会，这里做一个大致配置了解。

```properties
group=org.apache.kafka  
# NOTE: When you change this version number, you should also make sure to update  
# the version numbers in  
#  - docs/js/templateData.js  
#  - tests/kafkatest/__init__.py  
#  - tests/kafkatest/version.py (variable DEV_VERSION)  
#  - kafka-merge-pr.py  
version=2.7.2  
scalaVersion=2.13.3  
task=build  
org.gradle.jvmargs=-Xmx2g -Xss4m -XX:+UseParallelGC
```

gradle这里分配的是2g的堆内存，Xss4m每个线程的堆栈大小为4M，最后是使用`ParallelGC`垃圾收集器，也是JDK8的默认垃圾收集器。

## DEBUG模式

如果在启动参数里面设置了`KAFKA_DEBUG`，就可以开启DEBUG模式。

```sh
# Set Debug options if enabled
if [ "x$KAFKA_DEBUG" != "x" ]; then

    # Use default ports
    DEFAULT_JAVA_DEBUG_PORT="5005"

    if [ -z "$JAVA_DEBUG_PORT" ]; then
        JAVA_DEBUG_PORT="$DEFAULT_JAVA_DEBUG_PORT"
    fi

    # Use the defaults if JAVA_DEBUG_OPTS was not set
    DEFAULT_JAVA_DEBUG_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=${DEBUG_SUSPEND_FLAG:-n},address=$JAVA_DEBUG_PORT"
    if [ -z "$JAVA_DEBUG_OPTS" ]; then
        JAVA_DEBUG_OPTS="$DEFAULT_JAVA_DEBUG_OPTS"
    fi

    echo "Enabling Java debug options: $JAVA_DEBUG_OPTS"
    KAFKA_OPTS="$JAVA_DEBUG_OPTS $KAFKA_OPTS"
fi
```

我们需要了解的是 JAVA_DEBUG_OPTS 命令的含义。起初虽然不是很懂下面的参数含义，但是可以知道是JAVA调试应用程序用的。

```sh
-agentlib:jdwp=transport=dt_socket,server=y,suspend=${DEBUG_SUSPEND_FLAG:-n},address=$JAVA_DEBUG_PORT
```

我们调试程序更多是在IDE里面，下面的内容来自网络资料整合参考和理解：

[Debugging Java applications]([Debugging Java applications - IBM Documentation](https://www.ibm.com/docs/en/sdk-java-technology/8?topic=applications-debugging-java)) 这篇文章大概介绍了如何在JVM启动之后调试JAVA程序，以及如何在使用JDK调试应用程序。

若要调试 Java 进程，可以使用 Java 调试器 （JDB） 应用进程或其他调试器，这些调试器通过使用 SDK 为操作系统提供的 Java™ 平台调试器体系结构 （JPDA） 进行通信。

在Linux系统当中进行JAVA进程调试可以使用下面的命令。对于我们来说这些写法照着写就行，不需要过分追究具体的含义。

```sh
java -agentlib:jdwp=transport=dt_socket,server=y,address=_<port>_ <class>
```

调试远程服务器运行的JAVA应用程序，在Window中和Linux中调试方式如下：

-   On Windows systems:
    
```plaintext-ibm
jdb -connect com.sun.jdi.SocketAttach:hostname=<host>,port=<port>
```

-   On other systems:

```plaintext-ibm
jdb -attach <host>:<port>
```

此外Stack-Flow上还有一个写的更棒的帖子，这篇帖子的参数和Kafka的脚本部分基本一致了。

[debugging - What are Java command line options to set to allow JVM to be remotely debugged? - Stack Overflow](https://stackoverflow.com/questions/138511/what-are-java-command-line-options-to-set-to-allow-jvm-to-be-remotely-debugged)

Before Java 5.0, use `-Xdebug` and `-Xrunjdwp` arguments. These options will still work in later versions, but it will run in interpreted mode instead of JIT, which will be slower.

JDK5之前的版本这里可以直接忽略。（知道了也没啥用处）

From Java 5.0, it is better to use the `-agentlib:jdwp` single option:

JDK5之后使用下面的命令格式：

```java
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=1044
```

Options on `-Xrunjdwp` or `agentlib:jdwp` arguments are :

-   `transport=dt_socket` : means the way used to connect to JVM (socket is a good choice, it can be used to debug a distant computer)
-   `address=8000` : TCP/IP port exposed, to connect from the debugger,
-   `suspend=y` : if 'y', tell the JVM to wait until debugger is attached to begin execution, otherwise (if 'n'), starts execution right away.

- `transport=dt_socket` ：表示用于连接JVM的方式（socket是一个不错的选择，它可以用来调试远程计算机）
- `address=8000` ： TCP / IP端口公开，从调试器连接。
- `suspend=y` ：如果为“y”，则告诉 JVM 等到连接调试器后再开始执行，否则（如果为“n”），立即开始执行。

最后对比一下Kafka的参数，豁然开朗。

```sh
-agentlib:jdwp=transport=dt_socket,server=y,suspend=${DEBUG_SUSPEND_FLAG:-n},address=$JAVA_DEBUG_PORT
```

## Which java to use

如注释所言查找java命令在哪。

```sh
if [ -z "$JAVA_HOME" ]; then
  JAVA="java"
else
  JAVA="$JAVA_HOME/bin/java"
fi
```

##  Memory options

内存配置选项如下：

```sh
if [ -z "$KAFKA_HEAP_OPTS" ]; then
  KAFKA_HEAP_OPTS="-Xmx256M"
fi
```

> -Xmxn：指定内存分配池的最大大小（以字节为单位）。此值的倍数必须大于 2MB，1024 的倍数。

这里设置最大的HEAP大小为256M。

## cc_pkg

同样是jar包依赖的查找和引入到ClassPath当中，这里同样不知道干啥用的，简单理解是获取必要依赖项即可。

```sh
for cc_pkg in "api" "transforms" "runtime" "file" "mirror" "mirror-client" "json" "tools" "basic-auth-extension"
do
  for file in "$base_dir"/connect/${cc_pkg}/build/libs/connect-${cc_pkg}*.jar;
  do
    if should_include_file "$file"; then
      CLASSPATH="$CLASSPATH":"$file"
    fi
  done
  if [ -d "$base_dir/connect/${cc_pkg}/build/dependant-libs" ] ; then
    CLASSPATH="$CLASSPATH:$base_dir/connect/${cc_pkg}/build/dependant-libs/*"
  fi
done
```

## Exclude jars not necessary for running commands.

排除命令不需要的jar包，比如test和javadoc等。

```sh
regex="(-(test|test-sources|src|scaladoc|javadoc)\.jar|jar.asc)$"
should_include_file() {
  if [ "$INCLUDE_TEST_JARS" = true ]; then
    return 0
  fi
  file=$1
  if [ -z "$(echo "$file" | egrep "$regex")" ] ; then
    return 0
  else
    return 1
  fi
}
```

## INCLUDE_TEST_JARS

判断是否开启了包含测试的jar包。

```sh
if [ -z "$INCLUDE_TEST_JARS" ]; then
  INCLUDE_TEST_JARS=false
fi
```

# 写在最后

不得不感叹学无止境，知道的越多不知道的也就更多，一个脚本里面居然有这么多学问，本部分的核心毫无疑问是**JVM的启动参数**，其他的参数或者配置以及奇怪的脚本写法看不懂 也没啥关系，这里仅仅对于一些个人关注的核心部分进行介绍，对于一些细枝末节不做过多的追究和钻牛角尖，读者感兴趣可以对比参考资料做更多了解。