---
title: Redis学习 - 复制以及三种部署模式
subtitle: redis复制以及集群部署的方式
url_suffix: redis_copy
author: lazytime
tags:
  - redis
categories:
  - redis
keywords: reids,redis复制
description: redis复制以及集群部署的方式
copyright: true
date: 2020-11-29 19:59:01
---

# Redis学习 - 复制以及三种部署模式

## 什么是复制

单机的redis通常情况是无法满足项目需求的，一般都建议使用集群部署的方式进行数据的多机备份和部署，这样既可以保证数据安全，同时在redis宕机的时候，复制也可以对于数据进行快速的修复。



<!-- more -->

## 采取的方式

1. 单机部署（忽略）
2. 主从链
3. 一主多从
4. 哨兵模式
5. 集群模式

## 复制的前提

1. 需要保证`redis.conf`里面的配置是正确的，比如：

```
dir ./
dbfilename "dump.rdb"
```

2. 需要保证指定的路径对于redis来说是**可写**的，意味着如果当前目录没有写权限同样会失败

## 从服务器连接主服务器的几种方式

1. 在从服务器的配置文件里面配置连接那个主服务器：

连接的具体配置如下：

> 在5.0版本中使用了`replicaof`代替了`slaveof`（[github.com/antirez/red…](https://github.com/antirez/redis/issues/5335)），`slaveof`还可以继续使用，不过建议使用`replicaof`

下面是个人的配置

```properties
# replicaof <masterip> <masterport> 
replicaof 127.0.0.1 16379

```

> **警告**：此小节只说明了这一个配置的更改，进行主从配置的时候还有其他几个参数需要更改，这里只作为部分内容参考

2. 在启动的适合，在**redis从服务器**的redis-cli当中敲击如下的命令：

```shell
127.0.0.1:16380> slaveof 127.0.0.1 16379
OK Already connected to specified master
```

这样就可以在从服务器动态的指定要连接哪个主服务器了，但是这种配置是**当前运行时有效**，下次再次进入的时候，会根据配置文件进行配置或者按照默认的规则当前实例就是**master**3. 

3. 在从服务器执行`slaveof no one`，当前实例脱离控制自动成为`master`

## redis 复制启动的过程==（重点）==

| 主服务器操作                                                 | 从服务器操作                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. （等待命令）                                              | 1. 连接（重新连接）主服务器，发送**sync**命令                |
| 2. 开始执行bgsave，使用缓冲区记录bgsave之后执行所有写命令    | 2. 根据配置选项是使用现有的数据（存在）处理客户端请求，还是向请求的客户端返回错误信息 |
| 3. bgsave执行完毕，向从服务器发送**快照文件**，同时异步执行缓冲区记录的写命令 | 3. **丢弃所有的旧数据**，载入主服务器的快照文件              |
| 4.  快照文件发送完毕，开始向着从服务器发送存储在缓冲区的写命令 | 4. 完成对于快照的解释操作，恢复日常的请求操作                |
| 5. 缓冲区写命令发送完成，同时现在每执行一个写命令就像从服务器发送相同写命令 | 5. 执行主服务器发来的所有存储在缓冲区的写命令，并且从现在开始接受主服务器的每一个命令 |

> 建议：由于**bgsave**需要开启进行子线程的创建写入缓冲区的创建，所以最好在系统中预留30% - 45% 内存用于redis的bgsave操作

> 特别注意：当从服务器连接主服务器的那一刻，执行到第三步会**清空**当前redis里面的所有数据。

### 配置方式和命令方式的区别：

redis.conf 配置slaveof 的方式：不会马上进行主服务器同步，而是**先载入当前本地存在的rdb或者aof**到redis中进行数据恢复，然后才开始同步复制

命令slaveof方式：会**立即**连接主服务器进行同步操作

### 关于redis的主主复制:

如果我们尝试让两台服务器互相slaveof 那么会出现上面情况呢？

从上面的复制过程可以看到，当一个服务器slaveof另一个服务器，产生的结果只会是两边相互覆盖，也就是从服务器会去同步主服务器的数据，如果此时按照主主的配置，两边互相同步对方的数据，这样产生的数据可能会不一致，或者数据干脆就是不完整的。不仅如此，这种操作还会大量占用资源区让两台服务器互相知道对方

### 当一台服务器连接另一台服务器的时候会发生什么？

| 当有新服务器连接的时候    | 主服务器操作                                                 |
| ------------------------- | ------------------------------------------------------------ |
| 步骤3还没有执行           | 所有从服务器都会收到相同的快照文件和相同缓冲区写命令         |
| 步骤3正在执行或者已经执行 | 完成了之前同步的五个操作之后，会跟新服务器重新执行一次新的五个步骤 |



## 系统故障处理

复制和持久化虽然已经基本可以保证系统的数据安全，但是总有意外的情况，比如突然断电断网，系统磁盘故障，服务器宕机等一系列情况，那么会出现各种莫名奇妙的问题，下面针对这些情况说明一下解决方式：

### 验证快照文件以及aof文件

在redis的`bin`目录下面，存在如下的两个sh

```
-rwxr-xr-x 1 root root 9722168 Nov 15 20:53 redis-check-aof
-rwxr-xr-x 1 root root 9722168 Nov 15 20:53 redis-check-rdb
```

他们的命令作用和内容如下：

```
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ ./redis-check-aof 
Usage: ./redis-check-aof [--fix] <file.aof>
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ ./redis-check-rdb 
Usage: ./redis-check-rdb <rdb-file-name>

```

redis-check-aof：如果加入`--fix`选项，那么命令会尝试修复aof文件，会将内容里面出现错误的命令以及下面的所有命令清空，一般情况下回清空尾部的一些未完成命令。

redis-check-rdb：遗憾的是目前这种修复收效甚微。建议在修复rdb的时候，用SHA1和SHA256验证文件是否完整。

### 校验和与散列值：

redis2.6 之后加入了校验和与散列值进行验证。

快照文件增加CRC64校验和

> 什么是crc**循环冗余校验**？
>
> https://zh.wikipedia.org/wiki/%E5%BE%AA%E7%92%B0%E5%86%97%E9%A4%98%E6%A0%A1%E9%A9%97

### 更换故障主服务器：

1. 假设A故障，存在BC两台机器，B为从服务，C为将要替换的主服务器
2. 向机器B发送save命令，同时创建一个新的快照文件，同步完成之后，发送给C
3. 机器C上面启动redis,让C成为B的主服务器

### Redis sentienel 哨兵

可以监视指定主服务器以及属下的从服务器

也就是我们常用的**哨兵模式**

但是随着时代进步，目前使用redis基本还是以`cluster模式`为主

## redis主从复制模式（redis6.0版本）：

### 前提说明：

有条件的可以弄三台虚拟机查看效果，这样模拟出来的效果算是比较真实的。

### 三台从服务器以及一台主服务器的配置

个人的办法是copy一个公用的配置，然后进行修改（这里只列举区别以及改动较多的地方，其他地方根据需要配置）：

第一台机器的配置：

```properties
pidfile /var/run/redis_16379.pid
port 16379
dbfilename dump16379.rdb
appendfilename "appendonly16379.aof"
logfile "log16379"
```

第二台机器的配置：

```properties
pidfile /var/run/redis_16380.pid
port 16380
dbfilename dump16380.rdb
appendfilename "appendonly16380.aof"
logfile "log16380"
```

第三台机器的配置：

```properties
pidfile /var/run/redis_16381.pid
port 16381
dbfilename dump16381.rdb
appendfilename "appendonly16381.aof"
logfile "log16381"
```

这时候要配置一台主服务器

```properties
pidfile /var/run/redis_10000.pid
port 10000
dbfilename dump10000.rdb
appendfilename "appendonly10000.aof"
logfile "log10000"
```

### 启动redis一主多从：

配置很简单，可以用**手动**进行主从复制，也可以使用**redis.conf**提前配置，具体区别上文已经进行过介绍，这里不再赘述。

从服务器可以通过命令：`slaveof 127.0.0.1 10000` 实现主从复制拷贝

可以通过命令`info replication` 查看主从配置的信息。

主服务器启动日志：

```
127.0.0.1:10000> info replication
# Replication
role:master
connected_slaves:0
master_replid:e2a92d8c59fbdde3b162da12f4d74ff28bab4fbb
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:10000> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=127.0.0.1,port=16381,state=online,offset=14,lag=1
slave1:ip=127.0.0.1,port=16380,state=online,offset=14,lag=1
slave2:ip=127.0.0.1,port=16379,state=online,offset=14,lag=1
master_replid:029e455ee6f8fdc0e255b6d5c4f63136d933fb24
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
```

可以看到进行主从配置之后，当前的目录下面多出了对应备份文件

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201123232149.png)

当进行主从配置之后，从服务就无法进行写入了，主服务器才可以写入：

```
127.0.0.1:16379> set key 1
(error) READONLY You can't write against a read only replica.
```

#### 测试一主多从复制：

主服务器敲入如下命令：

```
127.0.0.1:10000> hset key1 name1 value1
(integer) 1
127.0.0.1:10000> keys *
1) "key1"
```

从服务器：

```
127.0.0.1:16379> hget key1 name1
"value1"
127.0.0.1:16380> hget key1 name1
"value1"
127.0.0.1:16381> hget key1 name1
"value1"
```

### 主从链

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201123134010.png)

配置方式：

和主从配置一样，只不过主节点换为从节点。

> 注意：主从链的配置依然只能是master节点可以写数据，同时中间的节点也是slave



### 扩展

如何检测磁盘是否写入数据？

1. 主从服务器通过一个虚标值（unique dummy value）来验证从服务器是否真的把数据写入到自己的磁盘。
2. 通过命令：`info`检查结果当中的 `aof_appending_bio_fsync`的值是否为0:

> \# 5.0 版本之后改为如下形式验证
>
> repl_backlog_active:0







## redis主从哨兵模式（Redis sentienel）（redis6.0版本）

### 哨兵模式有什么作用：

Redis的哨兵模式就是对redis系统进行实时的监控，其主要功能有下面两点

1.**监测**主数据库和从数据库是否正常运行。

2.当我们的主数据库出现故障的时候，可以自动将从数据库转换为主数据库，实现**自动的切换**。

### 为什么要使用哨兵模式：

1. 主从复制在主节点宕机的情况下，需要人工干预恢复redis，无法实现高可用。
2. 主节点宕机的情况下需要备份数据到新的从节点，然后其他节点将主节点设置为新的redis，需要一次全量复制同步数据的过程

### 哨兵模式原理

主节点故障的时候，由redis sentinel自动完成故障发现和转移

### 如何部署哨兵模式：

1. 首先按照上一节配置，已经设置了一个主节点三个从节点的配置

> 下面的配置如下：
>
> 主节点：10000
>
> 从节点1：16379
>
> 从节点2：16380
>
> 从节点3：16381

```
[xd@iZwz99gyct1a1rh6iblyucZ ~]$ ps -ef | grep redis
xd        2964  2910  0 18:02 pts/0    00:00:00 grep --color=auto redis
root     26412     1  0 Nov23 ?        00:06:07 ./redis-server 127.0.0.1:10000
root     26421     1  0 Nov23 ?        00:05:37 ./redis-server 127.0.0.1:16379
root     26428     1  0 Nov23 ?        00:05:37 ./redis-server 127.0.0.1:16380
root     26435     1  0 Nov23 ?        00:05:37 ./redis-server 127.0.0.1:16381
```

2. `sentinel.conf` 配置文件在安装redis的源码包里面有，所以如果误删了可以下回来然后把文件弄到手，其实可以配置一个常用的或者通用的配置放到自己的本地有需要直接替换
3. **配置5个sentienl.conf文件（建议奇数个哨兵，方便宕机选举产生新的节点）**

```shell
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ sudo cp sentinel.conf sentinel_26379.conf
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ sudo cp sentinel.conf sentinel_26380.conf
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ sudo cp sentinel.conf sentinel_26381.conf
[xd@iZwz99gyct1a1rh6iblyucZ bin]$ sudo cp sentinel.conf sentinel_10000.conf
```

4. 四个配置文件的改动依次如下：

所有的`sentinel.conf` 配置如下：

```shell
# 指定哨兵端口
port 20000
# 监听主节点10000
sentinel monitor mymaster 127.0.0.1 10000 2
# 连接主节点时的密码，如果redis配置了密码需要填写
sentinel auth-pass mymaster 12345678
# 故障转移时最多可以有2从节点同时对新主节点进行数据同步
sentinel config-epoch mymaster 2
# 故障转移超时时间180s，
sentinel failover-timeout mymasterA 180000 
# sentinel节点定期向主节点ping命令，当超过了300S时间后没有回复，可能就认定为此主节点出现故障了……
sentinel down-after-milliseconds mymasterA 300000
# 故障转移后，1代表每个从节点按顺序排队一个一个复制主节点数据，如果为3，指3个从节点同时并发复制主节点数据，不会影响阻塞，但存在网络和IO开销
sentinel parallel-syncs mymasterA 1
# 设置后台启动
daemonize yes
# 进程的pid文件，保险起见设置不一样的，特别是设置后台启动的时候
pidfile /var/run/redis-sentinel.pid
```

> 扩展：如何判定转移失败:
>
> a - 如果转移超时失败，下次转移时时间为之前的2倍；
>
> b - 从节点变主节点时，从节点执行slaveof no one命令一直失败的话，当时间超过**180S**时，则故障转移失败
>
> c - 从节点复制新主节点时间超过**180S**转移失败

下面为配好五个之后的配置：

```shell
-rw-r--r-- 1 root root   10772 Nov 28 21:00 sentienl_26382.conf
-rw-r--r-- 1 root root   10767 Nov 28 20:43 sentinel_10000.conf
-rw-r--r-- 1 root root   10772 Nov 28 21:03 sentinel_26379.conf
-rw-r--r-- 1 root root   10766 Nov 28 20:46 sentinel_26380.conf
-rw-r--r-- 1 root root   10772 Nov 28 20:59 sentinel_26381.conf
-rw-r--r-- 1 root root   10772 Nov 28 21:03 sentinel_26382.conf
-rw-r--r-- 1 root root   10744 Nov 28 18:06 sentinel.conf
```

5. 上一节已经启动过，这里不再介绍

5. **启动sentinel服务**

启动五个哨兵：

```shell
./redis-sentinel ./sentinel_10000.conf 
./redis-sentinel ./sentinel_26379.conf 
./redis-sentinel ./sentinel_263780.conf 
./redis-sentinel ./sentinel_263781.conf 
./redis-sentinel ./sentinel_263782.conf 
```

使用`ps`命令查看所有的服务：

```shell
root      3267     1  0 21:14 ?        00:00:01 ./redis-sentinel *:20000 [sentinel]
root      3280     1  0 21:15 ?        00:00:01 ./redis-sentinel *:26379 [sentinel]
root      3296     1  0 21:20 ?        00:00:00 ./redis-sentinel *:26380 [sentinel]
root      3303     1  0 21:21 ?        00:00:00 ./redis-sentinel *:26381 [sentinel]
root      3316  3254  0 21:28 pts/0    00:00:00 grep --color=auto redis
root     26412     1  0 Nov23 ?        00:06:17 ./redis-server 127.0.0.1:10000
root     26421     1  0 Nov23 ?        00:05:47 ./redis-server 127.0.0.1:16379
root     26428     1  0 Nov23 ?        00:05:47 ./redis-server 127.0.0.1:16380
root     26435     1  0 Nov23 ?        00:05:47 ./redis-server 127.0.0.1:16381
```

7. 验证一下哨兵是否管用

10000是主节点，他的`info`信息如下：

```
# Keyspace
db0:keys=1,expires=0,avg_ttl=0
127.0.0.1:10000> info replication
# Replication
role:master
connected_slaves:3

```

使用`kill -9 master节点进程端口号`之后，我们已经干掉了额主进程，验证一下从节点是否启动

进入到6379端口的`redis-cli`当中，可以看到从节点6379的实例被选举为新的的节点

```
127.0.0.1:16379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=16380,state=online,offset=857706,lag=1
slave1:ip=127.0.0.1,port=16381,state=online,offset=858242,lag=1

```

> **挂掉的主节点恢复之后，能不能进行恢复为主节点？**
>
> 尝试重启挂掉的master之后，可以发现他变成了从节点
>
> ```shell
> 127.0.0.1:10000> info replication
> # Replication
> role:slave
> master_host:127.0.0.1
> master_port:16379
> master_link_status:up
> master_last_io_seconds_ago:2
> 
> ```

**注意：生产环境建议让redis Sentinel部署到不同的物理机上**

如果不喜欢上面的启动哨兵模式，也可以使用下面的命令开启：

```
[root@dev-server-1 sentinel]# redis-server sentinel1.conf --sentinel
[root@dev-server-1 sentinel]# redis-server sentinel2.conf --sentinel
[root@dev-server-1 sentinel]# redis-server sentinel3.conf --sentinel
```



### 哨兵模式部署建议

a，sentinel节点应部署在**多台**物理机（**线上**环境）

b，至少三个且**奇数**个sentinel节点

c，通过以上我们知道，**3个sentinel**可同时监控一个主节点或多个主节点

  监听N个主节点较多时，如果sentinel出现异常，会对多个主节点有影响，同时还会造成sentinel节点产生过多的网络连接，

  一般线上建议还是， **3个sentinel**监听一个主节点

也可以按照下面的方式在启动哨兵的时候启动：

### 哨兵模式的优缺点：

优点：

1. 哨兵模式基于主从复制模式，所以主从复制模式有的优点，哨兵模式也有
2. 哨兵模式下，master挂掉可以自动进行切换，系统可用性更高

缺点：

1. 同样也继承了主从模式难以在线扩容的缺点，Redis的容量受限于单机配置
2. 需要额外的资源来启动sentinel进程，实现相对复杂一点，同时slave节点作为备份节点不提供服务

## redis集群模式（redis6.0版本）

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201129145217.png)

随着应用的扩展，虽然主从模式和哨兵模式的加入解决了高可用的问题，但是现代的应用基本都是要求可以动态扩展了，为了支持动态扩展，redis在后续的版本当中加入了哨兵的模式

集群模式主要解决的问题是：

Cluster模式实现了Redis的分布式存储，即每台节点存储不同的内容，来解决在线扩容的问题

### redis结构设计：

使用的是无中心的结构，每一个节点和节点之间相互连接

1. redis 使用彼此互联的(ping-pong)的方式，进行互相关联，内部使用二进制协议优化速度
2. 客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
3. 节点的fail是通过集群中超过**半数**的节点检测失效时才生效

### redis集群的工作机制

1. 在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383，redis会根据接节点的数量分配槽的位置来进行判定发送给哪一个cluster节点

2. 当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作

3. 为了保证高可用，Cluster模式也引入**主从复制模式**，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点

4. 当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么**该集群就无法再提供服务了**


   

### 配置集群（重点）：

为了不产生干扰，先把上一节所有的redis进程干掉，包括哨兵的配置

使用`kil -9 进程端口号`直接抹掉整个应用

配置如下：

1. 集群至少需要三主三从，同时需要奇数的节点配置。
2. 我们可以将之前的主从配置的一主三从**增加两个主节点**，目前的配置如下：

```
-rw-r--r-- 1 root root   84993 Nov 28 21:41 redis10000.conf
-rw-r--r-- 1 root root   84936 Nov 28 21:35 redis16379.conf
-rw-r--r-- 1 root root   84962 Nov 28 21:35 redis16380.conf
-rw-r--r-- 1 root root   84962 Nov 28 21:35 redis16381.conf

# 增加两个主要节点
-rw-r--r-- 1 root root   84962 Nov 28 21:35 redis16382.conf
-rw-r--r-- 1 root root   84962 Nov 28 21:35 redis16383.conf
```

主节点的配置主要如下：

```
port 7100 # 本示例6个节点端口分别为7100,7200,7300,7400,7500,7600 
daemonize yes # r后台运行 
pidfile /var/run/redis_7100.pid # pidfile文件对应7100,7200,7300,7400,7500,7600 
cluster-enabled yes # 开启集群模式 
masterauth passw0rd # 如果设置了密码，需要指定master密码
cluster-config-file nodes_7100.conf # 集群的配置文件，同样对应7100,7200等六个节点
cluster-node-timeout 15000 # 请求超时 默认15秒，可自行设置 

```

启动如下：

```shell
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17000_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17100_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17200_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17300_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17400_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-server ./cluster/redis17500_cluster.conf
[root@iZwz99gyct1a1rh6iblyucZ bin]# ps -ef | grep redis
root      4761     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17000 [cluster]
root      4767     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17100 [cluster]
root      4773     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17200 [cluster]
root      4779     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17300 [cluster]
root      4785     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17400 [cluster]
root      4791     1  0 15:55 ?        00:00:00 ./redis-server 127.0.0.1:17500 [cluster]
root      4797  4669  0 15:55 pts/0    00:00:00 grep --color=auto redis

```

启动了上面六个节点之后，使用下面的命令并且敲入`yes`让他们变为集群：

```
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-cli --cluster create 127.0.0.1:17000 127.0.0.1:17100 127.0.0.1:17200 127.0.0.1:17300 127.0.0.1:17400 127.0.0.1:17500 --cluster-replicas 1

>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:17400 to 127.0.0.1:17000
Adding replica 127.0.0.1:17500 to 127.0.0.1:17100
Adding replica 127.0.0.1:17300 to 127.0.0.1:17200
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 1179bb5f47e7f8221ba7917b5852f8064778e0db 127.0.0.1:17000
   slots:[0-5460] (5461 slots) master
M: 153afa1b9b14194de441fffa791f8d9001badc66 127.0.0.1:17100
   slots:[5461-10922] (5462 slots) master
M: 4029aeeb6b80e843279738d6d35eee7a1adcd2ff 127.0.0.1:17200
   slots:[10923-16383] (5461 slots) master
S: 3ceb11fe492f98432f124fd1dcb7b2bb1e769a96 127.0.0.1:17300
   replicates 1179bb5f47e7f8221ba7917b5852f8064778e0db
S: 66eaea82ccf69ef96dbc16aac39fd6f6ed3d0691 127.0.0.1:17400
   replicates 153afa1b9b14194de441fffa791f8d9001badc66
S: c34aeb59c8bedc11b4aeb720b70b0019e7389093 127.0.0.1:17500
   replicates 4029aeeb6b80e843279738d6d35eee7a1adcd2ff

```

#### 验证集群：

1. 输入`redis-cli`进入任意的一个主节点，注意是主节点，从节点不能做写入操作

`Redirected to slot [9189] located at 127.0.0.1:17100`根据Hash的算法，算出连接那个节点槽，然后提示slot[9189] 落到了17100上面，所以集群会自动跳转进行Key的加入

```shell
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-cli -p 17000
127.0.0.1:17000> set key1 1
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-cli -p 17000
127.0.0.1:17000> set key1 1
(error) MOVED 9189 127.0.0.1:17100
[root@iZwz99gyct1a1rh6iblyucZ bin]# ./redis-cli -p 17000 -c
127.0.0.1:17000> set key1 ke
-> Redirected to slot [9189] located at 127.0.0.1:17100
OK
```

> 小贴士：集群之后不能使用传统的连接方式，因为每一个key都要经过一次hash的操作找到对应的槽 -》节点之后才能做后续的操作
>
> 使用如下命令进入后正常
>
> ./redis-cli -p 17000 **-c**
>
> -c 代表以集群的方式连接

2. 可以使用如下命令验证集群的信息:

```shell
127.0.0.1:17000> cluster nodes
66eaea82ccf69ef96dbc16aac39fd6f6ed3d0691 127.0.0.1:17400@27400 slave 153afa1b9b14194de441fffa791f8d9001badc66 0 1606639411000 2 connected
4029aeeb6b80e843279738d6d35eee7a1adcd2ff 127.0.0.1:17200@27200 master - 0 1606639411000 3 connected 10923-16383
3ceb11fe492f98432f124fd1dcb7b2bb1e769a96 127.0.0.1:17300@27300 slave 1179bb5f47e7f8221ba7917b5852f8064778e0db 0 1606639410000 1 connected
1179bb5f47e7f8221ba7917b5852f8064778e0db 127.0.0.1:17000@27000 myself,master - 0 1606639410000 1 connected 0-5460
153afa1b9b14194de441fffa791f8d9001badc66 127.0.0.1:17100@27100 master - 0 1606639412002 2 connected 5461-10922
c34aeb59c8bedc11b4aeb720b70b0019e7389093 127.0.0.1:17500@27500 slave 4029aeeb6b80e843279738d6d35eee7a1adcd2ff 0 1606639413005 3 connected


```

3. 接下来我们验证一下当一个主节点挂掉会发生什么情况：

还是和主从复制的验证一样，直接Kill 进程：

kill掉 17000 之后，我们可以发现 17300 被升级为主节点

```shell
127.0.0.1:17300> info replication
# Replication
role:master
connected_slaves:0

```

此时的节点情况如下：

```shell
127.0.0.1:17100> cluster nodes
4029aeeb6b80e843279738d6d35eee7a1adcd2ff 127.0.0.1:17200@27200 master - 0 1606640582000 3 connected 10923-16383
153afa1b9b14194de441fffa791f8d9001badc66 127.0.0.1:17100@27100 myself,master - 0 1606640581000 2 connected 5461-10922
66eaea82ccf69ef96dbc16aac39fd6f6ed3d0691 127.0.0.1:17400@27400 slave 153afa1b9b14194de441fffa791f8d9001badc66 0 1606640581000 2 connected
c34aeb59c8bedc11b4aeb720b70b0019e7389093 127.0.0.1:17500@27500 slave 4029aeeb6b80e843279738d6d35eee7a1adcd2ff 0 1606640582624 3 connected
3ceb11fe492f98432f124fd1dcb7b2bb1e769a96 127.0.0.1:17300@27300 master - 0 1606640580619 7 connected 0-5460
1179bb5f47e7f8221ba7917b5852f8064778e0db 127.0.0.1:17000@27000 master,fail - 1606640370074 1606640367068 1 disconnected

```

4. 如果这时候主节点恢复呢？

和哨兵的模式一样，恢复之后也变为`slave`了。



### 集群模式优缺点：

优点：

1. 无中心架构，数据按照slot分布在多个节点。
2. 集群中的每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。
3. 可线性扩展到1000多个节点，节点可动态添加或删除
4. 能够实现自动故障转移，节点之间通过gossip协议交换状态信息，用投票机制完成slave到master的角色转换

缺点：

1. 客户端实现复杂，驱动要求实现Smart Client，缓存slots mapping信息并及时更新，提高了开发难度。目前仅JedisCluster相对成熟，异常处理还不完善，比如常见的“max redirect exception”
2. 节点会因为某些原因发生阻塞（阻塞时间大于 cluster-node-timeout）被判断下线，这种failover是没有必要的
3. 数据通过异步复制，不保证数据的强一致性
4. slave充当“冷备”，不能缓解读压力
5. 批量操作限制，目前只支持具有相同slot值的key执行批量操作，对mset、mget、sunion等操作支持不友好
6. key事务操作支持有线，只支持多key在同一节点的事务操作，多key分布不同节点时无法使用事务功能
7. 不支持多数据库空间，单机redis可以支持16个db，集群模式下只能使用一个，即db 0

>  Redis Cluster模式不建议使用pipeline和multi-keys操作，减少max redirect产生的场景。

### cluster的相关疑问

#### 为什么redis的槽要用 `16384`？

![img](https://img2018.cnblogs.com/blog/725429/201908/725429-20190829164650720-1058321793.jpg)

值得高兴的是：这个问题作者出门回答了：

能理解作者意思的可以不用看下面的内容

地址：https://github.com/redis/redis/issues/2576

```
The reason is:

Normal heartbeat packets carry the full configuration of a node, that can be replaced in an idempotent way with the old in order to update an old config. This means they contain the slots configuration for a node, in raw form, that uses 2k of space with16k slots, but would use a prohibitive 8k of space using 65k slots.
At the same time it is unlikely that Redis Cluster would scale to more than 1000 mater nodes because of other design tradeoffs.
So 16k was in the right range to ensure enough slots per master with a max of 1000 maters, but a small enough number to propagate the slot configuration as a raw bitmap easily. Note that in small clusters the bitmap would be hard to compress because when N is small the bitmap would have slots/N bits set that is a large percentage of bits set.
```

1. 首先我们查看一下结构体，关于cluster的源代码：`cluster.h`

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201129184329.png)

代码如下：

```c
typedef struct {
    char sig[4];        /* Signature "RCmb" (Redis Cluster message bus). */
    uint32_t totlen;    /* Total length of this message */
    uint16_t ver;       /* Protocol version, currently set to 1. */
    uint16_t port;      /* TCP base port number. */
    uint16_t type;      /* Message type */
    uint16_t count;     /* Only used for some kind of messages. */
    uint64_t currentEpoch;  /* The epoch accordingly to the sending node. */
    uint64_t configEpoch;   /* The config epoch if it's a master, or the last
                               epoch advertised by its master if it is a
                               slave. */
    uint64_t offset;    /* Master replication offset if node is a master or
                           processed replication offset if node is a slave. */
    char sender[CLUSTER_NAMELEN]; /* Name of the sender node */
    unsigned char myslots[CLUSTER_SLOTS/8];
    char slaveof[CLUSTER_NAMELEN];
    char myip[NET_IP_STR_LEN];    /* Sender IP, if not all zeroed. */
    char notused1[34];  /* 34 bytes reserved for future usage. */
    uint16_t cport;      /* Sender TCP cluster bus port */
    uint16_t flags;      /* Sender node flags */
    unsigned char state; /* Cluster state from the POV of the sender */
    unsigned char mflags[3]; /* Message flags: CLUSTERMSG_FLAG[012]_... */
    union clusterMsgData data;
} clusterMsg;

```

集群节点之间的通信内容无非就是IP信息，请求头，请求内容，以及一些参数信息，这里着重看一下参数`myslots[CLUSTER_SLOTS/8]`

> #define CLUSTER_SLOTS 16384  这里就是16384的来源

在redis节点发送心跳包时需要把所有的槽放到这个心跳包里，以便让节点知道当前集群信息，16384=16k，在发送心跳包时使用`char`进行bitmap压缩后是2k（`2 * 8 (8 bit) * 1024(1k) = 2K`），也就是说使用2k的空间创建了16k的槽数。

虽然使用CRC16算法最多可以分配65535（2^16-1）个槽位，65535=65k，压缩后就是8k（`8 * 8 (8 bit) * 1024(1k) = 8K`），也就是说需要需要8k的心跳包，作者认为这样做不太值得；并且一般情况下一个redis集群不会有超过1000个master节点，所以16k的槽位是个比较合适的选择。



## 参考资料：

https://juejin.cn/post/6844904097116585991#heading-9

https://juejin.cn/post/6844904178154897415#heading-25

[为什么Redis集群有16384个槽](https://www.cnblogs.com/rjzheng/p/11430592.html)

