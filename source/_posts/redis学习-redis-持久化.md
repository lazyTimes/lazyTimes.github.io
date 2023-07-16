---
title: redis学习 - redis 持久化
subtitle: 关于redis持久化的一些学习了解，以及一些拓展知识的了解
url_suffix: redis-chijiu
author: lazytime
tags:
  - Redis
categories:
  - Redis
keywords: ['Redis','持久化']
description: 关于redis持久化的一些学习了解，以及一些拓展知识的了解
copyright: true
date: 2020-11-21 22:24:47
---

无论面试和工作，持久化都是重点。

一般情况下,redis占用内存超过20GB以上的时候，必须考虑主从多redis实例进行数据同步和备份保证可用性。

rbd保存的文件都是 `dump.rdb`，都是配置文件当中的快照配置进行生成的。一般业务情况只需要用rdb即可。

aof默认是不开启的，因为aof非常容易产生大文件，虽然官方提供重写但是在文件体积过大的时候还是容易造成阻塞，谨慎考虑使用

rbd和aof在大数据量分别有各种不同情况的系统性能影响，具体使用何种解决策略需要根据系统资源以及业务的实际情况决定。

<!-- more -->

## 数据设计影响持久化：

https://szthanatos.github.io/topic/redis/improve-01/

## 为什么要持久化？

1. 重用数据
2. 防止系统故障备份重要数据

### 持久化的方式

1. RDB 快照：将某一个时刻的所有数据写入到磁盘
2. AOF（append-only file）：将所有的命令写入到此判断。

默认情况：**RDB**，AOF需要手动开启

## redis.conf持久化配置说明

在`redis.conf`文件当中，存在如下的选项：

`redis.conf`当中RDB的相关配置

```properties
#是否开启rdb压缩 默认开启
rdbcompression yes
#代表900秒内有一次写入操作，就记录到rdb
save 900 1
# rdb的备份文件名称
dbfilename dump.rdb
# 表示备份文件存放位置
dir ./
```

`redis.conf`当中AOF的相关配置

```properties
# 是否开启aof，默认是关闭的
appendonly no
#aof的文件名称
appendfilename "appendonly.aof"
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
appendfsync everysec
# 在进行rewrite的时候不开启fsync，即不写入缓冲区，直接写入磁盘，这样会造成IO阻塞，但是最为安全，如果为yes表示写入缓冲区，写入的适合redis宕机会造成数据持久化问题(在linux的操作系统的默认设置下，最多会丢失30s的数据)
no-appendfsync-on-rewrite no
# 下面两个参数要配合使用，代表当redis内容大于64m同时扩容超过100%的时候会执行bgrewrite，进行持久化
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

```

## RDB

### 创建rdb快照的几种方式：

1. 客户端向redis发送bgsave的命令（注意windows不支持bgsave），此时reids调用 **fork** 创建子进程，父进程继续处理，子进程将快照写入磁盘，父进程继续处理请求。
2. 客户端发送save命令创建快照。注意这种方式会**阻塞**整个父进程。很少使用，特殊情况才使用。
3. redis通过shutdown命令关闭服务器请求的时候，此时redis会停下所有工作执行一次save，阻塞所有客户端不再执行任何命令并且进行磁盘写入，写入完成关闭服务器。
4. redis集群的时候，会发送sync 命令进行一次复制操作，如果主服务器**没有执行**或者**刚刚执行完**bgsave，则会进行bgsave。
5. 执行**flushall** 命令

### RDB快照的一些注意点:

1. **只使用rdb的时候**，如果创建快照的时候redis崩溃，redis会留存上一次备份快照，但是具体丢失多少数据由备份时间查看
2. 只适用一些可以容忍一定数据丢失的系统，否则需要考虑aof持久化
3. 在大数据量的场景下，特别是内存达到20GB以上的适合，一次同步大约要4-6秒
   1. 一种方式是用手动同步，在凌晨的适合进行手动阻塞同步，比BGSAVE快一些

> 一种解决方法：
>
> 通过日志记录来恢复中断的日志，来进行数据的恢复

如何通过修改配置来获得想要的持久化？

1. 修改save参数，尽量在开发环境模拟线上环境设置save，过于频繁造成资源浪费，过于稀少有可能丢失大量数据
2. 日志进行聚合计算，按照save进行计算最多会丢失多少时间的数据，判断容忍性，比如一小时可以设置 `save 3600 1`

### RDB的优缺点对比：

#### 优点：

1. 适合大规模的数据恢复
2. 如果数据不小心误删，可以及时恢复
3. 恢复速度一般情况下快于aof

#### 缺点：

1. 需要一定的时间间隔，如果redis意外宕机，最后一次修改的数据就没有了，具体丢失多少数据需要看持久化策略
2. fork进程的时候，会占用一定的内存空间，如果fork的内存过于庞大，可能导致秒级别的恢复时间
3. 数据文件经过redis压缩，可读性较差

## AOF（append only fail）

其实就是把我们的命令一条条记录下来，类似linux的`history`

默认是不开启的，需要手动开启，开启之后需要重启

如果aof文件错位了，可以用`redis-check-aof` 进行文件修复

> 文件同步：写入文件的时候，会发生三件事：
>
> 1. file.write() 方法将文件存储到缓冲区
> 2. file.flush() 将缓冲区的内容写入到硬盘
> 3. sync 文件同步，阻塞直到写入硬盘为止

### AOC的同步策略

| 选项     | 同步频率                               |
| -------- | -------------------------------------- |
| always   | 每次命令都写入磁盘，严重降低redis速度  |
| everysec | 每秒执行一次，显示将多个命令写入到磁盘 |
| no       | 操作系统决定，佛系                     |

分析：

1. 第一种对于固态的硬盘的伤害比较大，我们都知道固态的擦写次数的寿命是远远小于机械硬盘的，频繁的io是容易对固态造成欺骗认为一次擦写，导致本就寿命不长的固态变得更命短，**基本不用**，特殊情况下有可能用得到
2. 第二种是默认的方式，也是推荐以及比较实用的方式，最多只会丢失一秒的数据，这种方式比较好的保证数据的备份可用，**推荐使用**
3. 第三种对于CPU的压力是最小的，因为由系统决定，但是需要考虑能不能接受不定量的数据丢失，还有一个原因是硬盘将缓冲区刷新到硬盘不定时，所以**不建议使用**



### 重写和压缩AOF文件：

由于1秒一次同步在不断写入之后造成文件内容越来越大，同时同步速度也会变慢，为了解决这个问题，redis引入了`bgrewriteaof`命令来进行压缩，和`bgsave`创建快照类似，同样会有子进程拖垮的问题，同时会有大文件在重写的时候带来巨大的文件系统删除的压力，导致系统阻塞。

命令如下

`bgrewriteaof`

示例如下：

> 127.0.0.1:16379> BGREWRITEAOF
> Background append only file rewriting started

> 参数控制：
>
> auto-aof-rewrite-percentage：**100**
>
> auto-aof-rewrite-min-size ：**64MB**
>
> 这里案例配置代表当AOF大于64并且扩大了100%将处罚**bgrewrite**命令

#### redis aof的rewrite做了那些事？

1. 对于一些冗余的命令进行清除
2. 检测存在错误的命令，将错误命令下面的所有命令都进行清理，一般情况是末尾由于宕机没有执行完的一些命令清理。

### aof的优缺点对比

#### 优点：

1. 从不同步，效率高
2. 每秒同步一次，可能丢失一秒数据
3. 每次修改都同步，文件完整性好

#### 缺点：

1. 相对于数据文件来说，aof远远大于rdb。修复速度慢一些
2. 存在未知的bug，比如如果重写aof文件的时候突然中断，会有很多奇怪的现象

## 如何检查redis的性能瓶颈：

1. redis-benchmark 官方推荐的性能测试工具，非常强大，具体的地址为：https://www.runoob.com/redis/redis-benchmarks.html
2. Redis-cli中调用`slowlog get`，作用是返回执行时间**超过redis.conf**中定义的持续时间的命令列表，注意这个时间仅仅是请求的处理时间，不包含网络通信的时间，**默认值是一秒**，

> redis.conf 当中对于慢日志的解释:
>
> The following time is expressed in microseconds, so 1000000 is equivalent to one second. Note that a negative number disables the slow log, while a value of zero forces the logging of every command.
>
> 接下来的时间以微秒为单位，因此1000000等于一秒。 请注意，负数将禁用慢速日志记录，而零值将强制记录每个命令。**（以微秒为单位）**
>
> **slowlog-log-slower-than 10000**
>
> There is no limit to this length. Just be aware that it will consume memory. You can reclaim memory used by the slow log with SLOWLOG RESET.
>
> 该长度没有限制。 请注意，它将消耗内存。 您可以使用SLOWLOG RESET回收慢速日志使用的内存。**（意思就是说超过128条之后的命令会被自动移除）**
>
> **slowlog-max-len 128**

> 可以用命令 SLOWLOG RESET 清楚慢日志占用的内存
>
> 127.0.0.1:16379> SLOWLOG reset
> OK

==慢日志是存储在内存当中的，切记==

## 持久化性能建议

> - 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。

> - 如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。

> - 如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。**新浪微博**就选用了这种架构。

其他性能优化指南（强烈推荐）：

https://szthanatos.github.io/topic/redis/improve-02/

## 总结对比rdb和aof：

|              | RDB                                                          | AOF                                                          |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **存储内容** | 数据                                                         | 写操作日志                                                   |
| **性能影响** | 小                                                           | 大                                                           |
| **恢复速度** | 高                                                           | 低                                                           |
| **存储空间** | 小                                                           | 大                                                           |
| **可读性**   | 低                                                           | 高                                                           |
| **安全程度** | 较低，保存频率低                                             | 较高，保存频率高                                             |
| **默认开启** | 是                                                           | 否                                                           |
| **存储策略** | `save 900 1`：九百秒内一次修改即保存 `save 300 10`：三百秒内十次修改即保存 `save 60 10000`：六十秒内一万次修改即保存 允许自定义 | `always`：逐条保存 or `everysec`：每秒保存 or `no`：系统自己决定什么时候保存 |

## 其他拓展知识：

### 关于linux内核开启`transparent_hugepage`会带来的阻塞问题：

个人对于Linux学艺不精，就直接引用文章了，侵权请联系删除

[Linux 关于Transparent Hugepages的介绍](https://www.cnblogs.com/kerrycode/p/4670931.html)

[简单说说THP——记一次数据库服务器阻塞的问题解决](https://blog.51cto.com/1152313/1767927)

### 官方解决aof和rdb对于性能问题的折中处理方式

1. redis4.0之后有一个参数叫做:`aof-use-rdb-preamble yes`

参数解释如下：

```
# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading, Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, then continues loading the AOF
# tail.
＃重写AOF文件时，Redis可以在
＃AOF文件可加快重写和恢复速度。 启用此选项时
重写的AOF文件上的＃由两个不同的节组成：
＃
＃[RDB文件] [AOF尾巴]
＃
＃加载时，Redis会识别AOF文件以“ REDIS”开头
＃字符串并加载带前缀的RDB文件，然后继续加载AOF
＃ 尾巴。

```

大致的内容就是说redis会将较早的部分内容转为RDB文件进行恢复，同时加入近期的数据为AOF文件

加载的时候先执行rdb文件的恢复，然后再加载aof命令

### 如何进行内存清理

在**redis4.0**之后，可以通过将配置里的`activedefrag`设置为`yes`开启自动清理，或者通过`memory purge`命令手动清理。

