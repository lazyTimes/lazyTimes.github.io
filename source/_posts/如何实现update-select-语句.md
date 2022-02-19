---
title: 如何实现update select 语句
subtitle: 一个数据库的使用经验技巧
url_suffix: updateselect
author: lazytime
tags:
  - 数据库
categories:
  - 数据库-其他
keywords: updateselect
description: 一个数据库的使用经验技巧
copyright: true
date: 2021-03-28 15:01:30
---

# 如何实现update select 语句

## 前言：

​	有些时候我们会遇到如下情况，我们需要依赖一张表的查询结果来更新另一张表，比如我们存在一张主表和一张关联表，我们需要把关联表的部分字段数据同步到主表的里面。

​	这次的文章出现也是因为这样一个类似的需求，个人需要把一个**30万行**（后续会发文介绍常见的处理手段）的数据文件入库，同时需要将部分字段迁移到另一张表，两个表之间通过**两个字段**进行and匹配。下面画一下结构图：

​	![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210315133417.png)

​	处理方式也比较简单，直接使用sql就可以完成，这篇文章针对这个小需求，总结一下update select 的几种实现方式。

<!-- more -->

# 文章目的：

1. 实现update select 的几种常见方法
   1. join
   2. merge
   3. 子查询
2. merge的踩坑和问题

# 准备数据

​	为了更好的进行实际操作，这里构建两张简单的表来模拟场景。直接复制下面的db即可，由于不同数据库sql不同，这里使用的是**postgreSql** 数据库。

## 旧表

```sql
CREATE TABLE "public"."olddb" (
  "id" int4 NOT NULL,
  "relevance1" varchar(255) COLLATE "pg_catalog"."default",
  "relevance2" varchar(255) COLLATE "pg_catalog"."default",
  "new_field" varchar(255) COLLATE "pg_catalog"."default",
  CONSTRAINT "olddb_pkey" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."olddb" 
  OWNER TO "postgres";

COMMENT ON COLUMN "public"."olddb"."id" IS '主键';

COMMENT ON COLUMN "public"."olddb"."relevance1" IS '关联字段1';

COMMENT ON COLUMN "public"."olddb"."relevance2" IS '关联字段2';

COMMENT ON COLUMN "public"."olddb"."new_field" IS '新字段，需要由关联表同步';
```

表成功创建之后，在内部加入一些简单的数据：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210315134419.png)

## 新表

```sql
CREATE TABLE "public"."newdb" (
  "id" int4 NOT NULL,
  "relevance1" varchar(255) COLLATE "pg_catalog"."default",
  "relevance2" varchar(255) COLLATE "pg_catalog"."default",
  "new_field" varchar(255) COLLATE "pg_catalog"."default",
  CONSTRAINT "newdb_pkey" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."newdb" 
  OWNER TO "postgres";

COMMENT ON COLUMN "public"."newdb"."id" IS '主键';

COMMENT ON COLUMN "public"."newdb"."relevance1" IS '关联字段1';

COMMENT ON COLUMN "public"."newdb"."relevance2" IS '关联字段2';

COMMENT ON COLUMN "public"."newdb"."new_field" IS '新字段，需要同步到旧表';
```

> 提醒：注意数据库是**postgresql**，其他数据库可能存在字段等差别而无法成功 

表成功创建之后，在内部加入一些简单的数据：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210315134505.png)



# 实现方式汇总

## join

​	第一种的连接方式使用的是连接表的`join`方法，我们通过关联字段查出对应的关联记录，同时在关联之后将关联新字段的数据更新到旧表，这样就实现了每关联一条记录就更新一条记录数据：

​	实现方式也比较简单，只需要使用

```plsql
UPDATE olddb aa
SET new_field = bb.new_field
FROM
	 newdb bb where aa.relevance1 = bb.relevance1 
	AND aa.relevance2 = bb.relevance2
```

他的执行结果如下：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210315143721.png)

> 下面的下方是错的，这时候sql会抛出一个错误。
>
> ```sql
> UPDATE olddb ALIAS 
> SET ( new_field ) = (
> 	SELECT
> 		( bb.new_field ) 
> 	FROM
> 		olddb aa
> 		JOIN newdb bb ON aa.relevance2 = bb.relevance2 
> 		AND aa.relevance1 = bb.relevance1 
> 	)
> ```

## Merge（未验证）

第二种方式可能比较陌生，因为`merge`算是对于insert以及update的一个统合，粗略了解了一下发现能干不少事情，下面说下。

注意下面的方法在postgresql **报错**，原因是是我的postgresql**版本太低**，但是个人在升级过后还不能支持使用merge方法 ，所以这里保存了sql，可以改动后尝试到其它的数据库语言进行使用。

> ERROR:  syntax error at or near "MERGE" 
>
> 很头疼，在stackflow也没用找到答案。

```sql
merge into olddb as olds
using newdb news on olds.new_field = news.new_field
when matched
update set 
olds.new_field = news.new_field
```

需要注意的是不同的数据库对于merge的特性是不一致的，建议查看当前安装数据库的版本以及文档进行确认比较稳妥。

下面是 **postgresql** 的`merge`使用案例，注意一般建议版本为`11`以上再使用`merge`。

```plsql
MERGE INTO wines 
USING (VALUES('Chateau Lafite 2003', '24')) v
ON v.column1 = w.winename
WHEN NOT MATCHED 
  INSERT VALUES(v.column1, v.column2)
WHEN MATCHED
  UPDATE SET stock = stock + v.column2;
```

## 子查询

​	子查询是最简单也是最容易想到的一种方式，不过子查询有一个明显的缺点就是数据量较大的情况下通常性能都比较差， 这种操作通常适合数据量比较小的情况，下面是对应的案例语法：

```sql
UPDATE olddb 
SET new_field = ( SELECT newdb.new_field FROM newdb WHERE olddb.relevance1 = newdb.relevance1 AND olddb.relevance2 = newdb.relevance2 )
```

下面是子查询需要注意的点：

+ 如果子查询无法找到任何匹配的行，则更新后的值将被更改为NULL
+ 如果子查询找到多个匹配的行，update查询将返回一个错误。

> 错误的信息如下：
>
> `> ERROR:  more than one row returned by a subquery used as an expression`
>
> (\>错误:作为表达式使用的子查询返回多行)

+ 多数情况下子查询的性能较差，尽量避免使用

# 总结：

​	由于merge个人使用经验不足，花了较多时间依然没有解决，所以文章标题进行了标记，后续使用了其他的方式避开问题。

​	update select的实现实际情况复杂多变，这里只列举了最简单的使用情况。