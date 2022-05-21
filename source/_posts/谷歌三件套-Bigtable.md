---
title: 谷歌三件套 - Bigtable
subtitle: 三件套中可能是影响最大的一个组件
author: 阿东
url_suffix: google-bigtable
tags:
  - LevelDB
  - 谷歌
categories:
  - 谷歌三件套
keywords: 谷歌三件套
description: bigtable数据结构解析
copyright: true
date: 2022-05-21 19:16:54
---

# 谷歌三件套 - Bigtable
# 引言
如标题所言，这一篇文章简单介绍BigTable，其实个人更建议看LevelDB这款开源数据库，因为这数据库也是Bigtable的作者 **JeffreyDean** 设计的，很多内容不能说像简直就是一模一样。

值得注意的是，看Bigtable的内容**千万不要带着关系型数据库的思维**，建议看之前看看《数据密集型应用系统设计》的第三章，里面提到了LSM-Tree以及大数据系统设计思想，或者看看个人之前写的文章 [[《数据密集型型系统设计》LSM-Tree VS BTree]]

# 三件套论文资料
Bigtable 原始在线论文： **[Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/bigtable-osdi06.pdf)**

MapReduct 原始在线论文：[MapReduce: Simplified Data Processing on Large Clusters](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf)

GFS 原始在线论文：[The Google File System](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf)

如果看不懂英文或者想要线下阅读，个人从某全是广告的技术网站买了一份中文翻译以及原始英文论文PDF，合并到一起免费分享给大家（虽然掏钱买也没几个钱）：

链接: [Google-GFS,Bigtable,Mapreduce三大论文英文原版+中文翻译](https://pan.baidu.com/s/15Z24aq62u25Y_LrHSaNCCw) 提取码: 82ok 
> （如果链接失效可以关注公众号“懒时小窝” 回复“谷歌三件套”或者“谷歌”获取这些内容）

<!-- more -->


# 简单介绍
下面Bigtable介绍内容可以跳过，论文巴拉巴拉吹了一大堆，其实关键也就是关注这几个点：
- 大数据、分布式存储、异地多活容灾（侧面反应）。
- GFS和BigTable的关系。
- Chubby。
- LSM-Tree 数据结构。
- SSTable（LSM-Tree）。

Bigtable 是一个**分布式存储系统**，用于管理旨在扩展到非常大的结构化数据
大小：数千种商品的 PB 级数据服务器。 Google 的许多项目都将数据存储在 Bigtable 中，包括网络索引、谷歌地球和谷歌财经。 这些应用提出了非常不同的要求

在 Bigtable 上，无论是在数据大小方面（从 URL 到网页到卫星图像）和延迟要求
（从后端批量处理到实时数据服务）。尽管有这些不同的需求，Bigtable 还是成功地为所有用户提供了灵活、高性能的解决方案这些 Google 产品。 在本文中，我们描述了 Bigtable 提供的简单数据模型，它为客户提供对数据布局和格式的动态控制，我们描述了 Bigtable 的设计和实现。

Bigtable看起来像一个数据库，采用了很多数据库的实现策略。但是Bigtable并不支持完整的关系型数据模型；而是为客户端提供了一种简单的数据模型，客户端可以动态地控制数据的布局和格式，并且利用底层数据存储的局部性特征。Bigtable将数据统统看成无意义的字节串，客户端需要将结构化和非结构化数据串行化再存入Bigtable。

前面提到相当多的google应用使用了BigTable，比如Google Earth和Google Analytics，这里建议有条件高级上网的同学推荐看一下**Google Earth** 找找你家位置，你会发现在这个世界上你没有啥秘密可言（地理位置上），也能最直观的明白现代导弹为什么可以精准无误的打击，挺恐怖的事情。

题外话就扯到这，由于网上有很多介绍的文章，这里也同样结合原始论文和理解摘录自己感兴趣的部分，因为个人是看完一整个LevelDB的源代码之后再回来看的，很多东西都省略了，没看过的更多内容可以参考下面这篇文章：[BigTable解读](https://blog.csdn.net/OpenNaive/article/details/7532589)

我们不需要关注谷歌吹逼自己的高性能，高负载介绍，毕竟都会这么对外宣传，我们只要了解Bigtable干了啥和怎么实现即可。

# 数据模型
首先介绍最为重要的数据组织结构也就是数据模型，论文第二节开头对于SSTable做了定义：

 A Bigtable is a sparse（稀疏）, distributed（分布式）, persistent（持久化） multidimensional（多维度） sorted map（排序哈希表）。

简单的数据模型意味着灵活和很强的扩展性，SSTable 使用 `row`、`column` 和 `timestamp` 三个字段作为这个哈希的键，值是字节数组，其实也就是字符串。

一条数据最小单位可以抽象理解为这样的存储形式：

`(row:string, column:string, time:int64) → string`

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202205201448701.png)

再次强调这里的数据格式**不能按照关系型数据库理解**，原因是他本质上是**Key/Value**的存储格式，这三个值不能按照单纯行或者列存储格式理解，而是使用**混合存储**+多维数据的存储方式，所以这三个值抽象理解为**行键（row）、列键（column）、时间戳（timestamp）**，最终由这三个参数构成三维参数。

**行键（row）**

行键是原子操作，行数据可以是任意的字符串，最大可以扩展到64个KB，当然多数情况为10 - 100个字节，每一个读和写操作都是一个独立原子性的。

行的范围是可以动态划分的，行的数据切分称为切片，通过切片用户只需要和更加少量的数据通信，通过分片也可以更好的获取更加准确和可控的数据范围。

切片在行键中被称为 tablet，切片支持负载均衡，随着表的扩展片也会自动进行分裂，最终一个分片控制在100 MB - 200MB 当中。

**列键（column）**

列的存储格式涉及到一个被称之为 **列族** 的概念，通过列族的方式把相似的值组合到一起，一个列族里的列一般存储相同类型的数据，所以通常情况下列族的数据变动比较小，但是列族是可以随意添加和删除的，并且通过谷歌特定的格式进行命名，列族

这里补充列族的概念，指的是**把一行中的所有列和行主键保存到一起**，并且不使用列压缩的形式存储。其实这种用行转列基本就可以实现，所以列族严格意义上依然是行存储的变体，和真正的列存储还是存在差异的。

由于列族的存在，使得SSTable实现一个key的多维度映射，所以多维的概念就是在列族上出现的，同时可以把列族看做是二级索引。

**时间戳**

时间戳负责标记每一个行列索引的版本号，每个单元格可以包含多个版本，版本通过时间戳管理，BigTable的时间戳是64位整数，通常情况为微秒级别的单位，可以使用客户端进行指定单位。

时间戳显然就是三级别索引了，读取的时候通过最新的时间戳可以认为是数据的最新版本。另外在查询时如果 只给出行列，那么返回的是**最新版本的数据**；如果给出了行列时间戳，那么返回的是时间**小于或等于时间戳的数据**。

> 这也是现在大数据框架的存储格式特点，比如目前前景不错的**Tidb**，支持OLTP也支持OLAP。

# 支撑组件
BigTable除开SSTable之外，还存在其他的支持组件：
- 用GFS来存储日志和数据文件.
- 按SSTable文件格式存储数据.
- 用Chubby管理元数据.

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202205201416964.png)

**GFS（Google File System）**

从上面的内容可以看到，GFS先于memtable操作，显然充当了整个系统的日志部分，这一部分牵扯到谷歌三件套的另一个系统设计GFS，不是本文讨论的重点，这里只需要知道它干了所有和Log和数据存储位置有关的事情即可。

既然是日志和数据的存储，那么GFS自然也知道数据的具体位置，因为属于SSTable的前置组件，所以 SSTable 的具体位置需要GFS提前记录。

另外memtable相当于SSTable的缓存，当memtable成长到一定规模会被冻结，Bigtable随之创建一个新的memtable，并且将冻结的memtable转换为SSTable格式写入GFS，这个操作称为`minor compaction`。

> 在 LevelDB中体现的是Level0的SSTable 压缩合并。

**Chubby**

Bigtable 依赖于一个高可用和持久的分布式锁服务称为 **Chubby**，它由五个活动副本组成，其中一个是选举为主节点的Master，节点正常的时候可以进行互相通信，Chubby 使用Paxos算法保持一致。

Chubby提供了命名空间，内部通过小文件和目录组成，目录或者文件可以配置单独的锁，使得读和写操作都是原子性的，Chubby 客户端提供一致性的文件缓存，每一个Chubby 都必须和另一个 Chubby 保持会话，如果客户端会话过期会丢失全部的锁。

**SSTable**

终于要进入重点部分了，可惜的是原始论文并没有详细的介绍SSTable的内部数据结构，仅仅在论文第六个小节中介绍了SSTable的作用。

首先看看BigTable和GFS 是什么关系呢？在论文中我们可以看到一个类似树的结构，其中根节点为主服务器，主服务器负责接受请求，通过管理分片服务器将请求分片到不同的片服务器中，所以从外层看最终干活的是**片服务器**。

然而片服务器实际上本身也只是负责管理自己分片的SSTable，它也通过特殊索引知道数据在那个SSTable分片中，然后从GFS中读取SSTable文件的数据，而GFS则可能要从多个Chuncker server里面搜索数据。

而图中的metatable原数据表可以看作是和SSTable绑定的类似**索引**的关系，元数据表的数据是**不能被外界访问**的，外界访问的是元数据对应的SSTable分片。

# Bigtable集群

BigTable集群通过三个层级配套组件完成工作。

第一层是**主服务器（master server**）也就是我们上面提到的Chubby，本身也通过集群的方式保证root tablet正常访问，也可以直接看作我们广为使用的中间件节点集群。

第二层是**分片服务器**，也称 tablet，其中root tablet是元数据表（METADATA table）的第一个分片，

第三层是**元数据**的部分，和 root tablet 组成元数据映射表，元数据包含很多的用户数据分片。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202205211600915.png)

关于三个层级内部的组成这里不用过多猜测到底长啥样，还是那句话去看LevelDB吧。

# 总结
这篇文章简单介绍了一下BigTable 中一些核心部分，有很大部分内容都被忽略了，对于大数据方向的同学来说这三篇论文基本是必看的资料，因为说白了这三驾马车放到现在基本也可以通用，提到的很多理念对现在的中间件有很深的影响。

# 写在最后
关于**Bigtable**部分就介绍到这里了，虽然论文还要很多理论的部分，但是个人看下来之后基本在LevelDB都有体现，所以想要了解使用的直接看LevelDB源代码理解起来更快。

# 推荐阅读
[[《数据密集型型系统设计》LSM-Tree VS BTree]](https://segmentfault.com/a/1190000041652003)

[[LSM-Tree - LevelDb了解和实现]](https://segmentfault.com/a/1190000041720205)

[[LSM-Tree - LevelDb 源码解析]](https://segmentfault.com/a/1190000041864579)

[[LSM-Tree - LevelDb Skiplist跳表]](https://segmentfault.com/a/1190000041869586)

[[LSM-Tree - LevelDb 布隆过滤器]](https://segmentfault.com/a/1190000041873400)
