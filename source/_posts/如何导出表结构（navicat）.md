---
title: 如何导出表结构（navicat）
subtitle: 如何导出表结构（navicat）
author: lazytime
url_suffix: note18
tags:
  - 数据库
categories:
  - 数据库-其他
keywords: 如何导出表结构（navicat）
description: 如何导出表结构（navicat）
copyright: true
date: 2020-08-29 22:02:28
---

# 如何导出表结构（navicat）

## 应用场景

我需要将表复制到 markdow 或者 word里面的时候

<!-- more -->

## 具体使用方法：

1. 复制sql 复制到 navicat的查询里面
2. 需要设置数据库的名称：`table_schema`
3. 需要设置导出的表名：`table_name`

```sql
SELECT
	COLUMN_NAME 列名,
	COLUMN_TYPE 数据类型,
	DATA_TYPE 字段类型,
	CHARACTER_MAXIMUM_LENGTH 长度,
	IS_NULLABLE 是否为空,
	COLUMN_DEFAULT 默认值,
	COLUMN_COMMENT 备注 
FROM
	INFORMATION_SCHEMA.COLUMNS 
	WHERE-- developerclub为数据库名称，到时候只需要修改成你要导出表结构的数据库即可
	table_schema = 'galaxy_education' 
	AND -- article为表名，到时候换成你要导出的表的名称
-- 如果不写的话，默认会查询出所有表中的数据，这样可能就分不清到底哪些字段是哪张表中的了，所以还是建议写上要导出的名名称
	table_name = '表名称'
```

table_name = '表名称'

# 怎么样导出数据库所有的表结构和详细信息呢?

博客地址：
https://blog.csdn.net/cai454692590/article/details/82799638
https://www.cnblogs.com/LeeYongze/archive/2012/07/19/2599338.html

```sql
SELECT
     表名       = Case When A.colorder=1 Then D.name Else '' End,
     表说明     = Case When A.colorder=1 Then isnull(F.value,'') Else '' End,
     字段序号   = A.colorder,
     字段名     = A.name,
     字段说明   = isnull(G.[value],''),
     标识       = Case When COLUMNPROPERTY( A.id,A.name,'IsIdentity')=1 Then '√'Else '' End,
     主键       = Case When exists(SELECT 1 FROM sysobjects Where xtype='PK' and parent_obj=A.id and name in (
                      SELECT name FROM sysindexes WHERE indid in( SELECT indid FROM sysindexkeys WHERE id = A.id AND colid=A.colid))) then '√' else '' end,
     类型       = B.name,
     占用字节数 = A.Length,
     长度       = COLUMNPROPERTY(A.id,A.name,'PRECISION'),
     小数位数   = isnull(COLUMNPROPERTY(A.id,A.name,'Scale'),0),
     允许空     = Case When A.isnullable=1 Then '√'Else '' End,
     默认值     = isnull(E.Text,'')
 FROM
     syscolumns A
 Left Join
     systypes B
 On
     A.xusertype=B.xusertype
 Inner Join
     sysobjects D
 On
     A.id=D.id  and D.xtype='U' and  D.name<>'dtproperties'
 Left Join
     syscomments E
 on
     A.cdefault=E.id
 Left Join
 sys.extended_properties  G
 on
     A.id=G.major_id and A.colid=G.minor_id
 Left Join

 sys.extended_properties F
 On
     D.id=F.major_id and F.minor_id=0
     --where d.name='OrderInfo'    --如果只查询指定表,加上此条件
 Order By
     A.id,A.colorder
```

# 使用方法：

以mysql为例，根据sql的内容。

根据示例的内容

```mysql
SELECT
	COLUMN_NAME 列名,
	COLUMN_TYPE 数据类型,
	DATA_TYPE 字段类型,
	CHARACTER_MAXIMUM_LENGTH 长度,
	IS_NULLABLE 是否为空,
	COLUMN_DEFAULT 默认值,
	COLUMN_COMMENT 备注 
FROM
	INFORMATION_SCHEMA.COLUMNS 
	WHERE-- developerclub为数据库名称，到时候只需要修改成你要导出表结构的数据库即可
	table_schema = 'hotel' 
	AND -- article为表名，到时候换成你要导出的表的名称
-- 如果不写的话，默认会查询出所有表中的数据，这样可能就分不清到底哪些字段是哪张表中的了，所以还是建议写上要导出的名名称
	table_name = 'orders'
```



![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200829215314.png)

点击`导出向导`

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200829215354.png)

选择需要导出的内容，根据所需导出，即可，我这里选择导出`excel`

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20200829215901.png)

我们可以将导出的内容贴到md 里面



| 列名        | 数据类型 | 字段类型 | 长度 | 是否为空 | 默认值 | 备注 |
| ----------- | -------- | -------- | ---- | -------- | ------ | ---- |
| id          | int(11)  | int      |      | NO       |        |      |
| table_id    | int(11)  | int      |      | YES      |        |      |
| orderDate   | datetime | datetime |      | YES      |        |      |
| totalPrice  | double   | double   |      | YES      |        |      |
| orderStatus | int(11)  | int      |      | YES      | 0      |      |