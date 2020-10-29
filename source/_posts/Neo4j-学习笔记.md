---
title: Neo4j 学习笔记
subtitle: Neo4j 学习
author: lazytime
url_suffix: note10
tags:
  - Neo4j
categories:
  - Neo4j
keywords: Neo4j 学习
description: Neo4j 学习
copyright: true
date: 2020-07-26 22:05:18
---

# Neo4j 学习

# 目录

## 1. 入门

### 1.1 安装neo4j

直接安装 neo4j desktop 版本

如果是community 社区版本，则需要到安装目录的bin 下面执行如下命令

<!-- more -->

## 2. [Cypher简介](https://neo4j.com/docs/getting-started/current/cypher-intro/)

```
()
(matrix)
(:Movie)
(matrix:Movie)
(matrix:Movie {title: "The Matrix"})
(matrix:Movie {title: "The Matrix", released: 1997})
```

### 2.1 关系语法

```
-->
-[role]->
-[:ACTED_IN]->
-[role:ACTED_IN]->
-[role:ACTED_IN {roles: ["Neo"]}]->
```

### 2.2 模式语法

```
(keanu:Person:Actor {name:  "Keanu Reeves"} )
-[role:ACTED_IN     {roles: ["Neo"] } ]->
(matrix:Movie       {title: "The Matrix"} )
```

## 3. Cypher 基本语法

### 3.1 创建数据

```
CREATE (:MyMovie{title:"The Matrix", released:1997})
```

### 3.2 创建并且返回数据

```
CREATE (p:MyPeron{ name:"test", relase:"test"}) RETURN p
```

### 3.3 创建多个元素

```
CREATE (a:Person { name:"Tom Hanks",
  born:1956 })-[r:ACTED_IN { roles: ["Forrest"]}]->(m:Movie { title:"Forrest Gump",released:1994 })
                                                               
CREATE (d:Person { name:"Robert Zemeckis", born:1951 })-[:DIRECTED]->(m)
RETURN a,d,r,m
```

### 3.4 匹配模式

```
MATCH (p:Person { name:"Tom Hanks" })-[r:ACTED_IN]->(m:Movie)
RETURN m.title, r.roles
```

### 3.5 连接结构

```
MATCH (p:Person { name:"Tom Hanks" })
CREATE (m:Movie { title:"Cloud Atlas",released:2012 })
CREATE (p)-[r:ACTED_IN { roles: ['Zachry']}]->(m)
RETURN p,r,m
```

### 3.6 完成模式

```
MERGE (m:Movie { title:"Cloud Atlas" })
ON CREATE SET m.released = 2012
RETURN m
```

使用合并关联多个

```
MATCH (m:Movie { title:"Cloud Atlas" })
MATCH (p:Person { name:"Tom Hanks" })
MERGE (p)-[r:ACTED_IN]->(m)
ON CREATE SET r.roles =['Zachry']
RETURN p,r,m
```

处理多重关联

```
CREATE (y:Year { year:2014 })
MERGE (y)<-[:IN_YEAR]-(m10:Month { month:10 })
MERGE (y)<-[:IN_YEAR]-(m11:Month { month:11 })
RETURN y,m10,m11
```

## 4. Cypher 筛选结果正确的结果

### 4.1 测试数据

```
CREATE (xiaohong:Person {name: "小红", born: 1954})
CREATE (xiaolv:Person {name: "小绿S", born: 1964})
CREATE (xiaohuang:Person {name: "小黄", born: 1948})
CREATE (xiaolan:Person {name: "小蓝", born: 1999})
CREATE (xiyouji:Person {name: "西游记", date: "1999-10-25"})
CREATE (lanjingling:Person {name: "蓝精灵", date: "1969-5-16"})
CREATE (xiaolv)-[:ACTED_IN { roles: ["想看"]}]->(xiyouji)
CREATE (xiaohuang)-[:ACTED_IN { roles: ["想看"]}]->(lanjingling)
```

### 4.2 根据where 条件查找

```
MATCH (xiyouji:Person) WHERE xiyouji.name="西游记" RETURN xiyouji
```

等同于如下查询

```
MATCH (xiyouji:Person {name:"西游记"}) RETURN xiyouji
```

使用正则匹配如下

```
MATCH (xiaolv:Person)-[r:ACTED_IN]->(xiyouji) WHERE xiaolv.name =~ "小+" OR "Neo" IN r.roles RETURN xiaolv,r,xiyouji
```

### 4.3 内置函数使用

#### 使用NOT 排除关系

```
MATCH (p:Person)-[:ACTED_IN]->(m)
WHERE NOT (p)-[:DIRECTED]->()
RETURN p,m
```

#### 内置函数的

```
MATCH (p:Person) RETURN p,p.name AS name, toUpper(p.name), coalesce(p.name, "n/a") AS nickname, { name: p.name, label: head(labels(p))} AS person
```

### 4.4 汇总信息

#### 使用count

```
MATCH (:Person)
RETURN count(*) AS people
```

#### 使用

```
MATCH (actor:Person)-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(director:Person) RETURN actor, director, count(*) AS collacbotration
```

### 4.5 排序和分页

#### 使用 limit 以及 skip 进行分页

```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie) RETURN a, count(*) AS appearances ORDER BY appearances DESC LIMIT 10
MATCH (a:Person)-[:ACTED_IN]->(m:Movie) RETURN a, count(*) AS appearances ORDER BY appearances DESC SKIP 10
```

### 4.6 收集汇总

```
MATCH (m:Movie)<-[:ACTED_IN]-(a:Person) RETURN m.title AS movie, collect(a.name) AS cast, count(*) AS actors
```

### 4.7 使用UNION链接多个内容

```
MATCH (actor:Person)-[r:ACTED_IN]->(movie:Movie)
RETURN actor.name AS name, type(r) AS type, movie.title AS title
UNION
MATCH (director:Person)-[r:DIRECTED]->(movie:Movie)
RETURN director.name AS name, type(r) AS type, movie.title AS title
```

#### 使用WITH

```
MATCH (person:Person)-[:ACTED_IN]->(m:Movie) WITH person, count(*) AS appear, collect(m.title) AS movies WHERE appear>1 RETURN person.name, appear, movies
```

### 4.8 使用约束

```
CREATE CONSTRAINT ON (movie:Movie) ASSERT movie.title IS UNIQUE
CALL db.constraints
```

### 4.9 使用import 导入数据

https://neo4j.com/docs/cypher-manual/3.5/clauses/load-csv/

#### 从CSV文件导入数据

要使用的CSV文件`LOAD CSV`必须具有以下特征：

- 字符编码为UTF-8；
- 终端线终端取决于系统，例如，它`\n`在UNIX上还是`\r\n`在Windows上；
- 默认的字段终止符是`,`;
- 可以使用命令中`FIELDTERMINATOR`可用的选项来更改字段终止符`LOAD CSV`；
- CSV文件中允许带引号的字符串，并且在读取数据时将引号删除；
- 字符串引号的字符为双引号`"`;
- if `dbms.import.csv.legacy_quote_escaping`设置为默认值`true`，`\`用作转义字符；
- 双引号必须在带引号的字符串中并使用转义符或第二个双引号进行转义。

下面为示例：

article.cvs

```
1,ABBA,1992
2,Roxette,1986
3,Europe,1979
4,The Cardigans,1992
LOAD CSV FROM 'file:///article.cvs' AS line
CREATE (:Artist { name: line[1], year: toInteger(line[2])})
```

注意此处会踩坑，因为neo4j 默认配置中将import 设置到安装根目录的`import` 目录下面

```
dbms.security.allow_csv_import_from_file_urls=true

dbms.directories.import=import
```

注意第一个配置默认为false， 设置为true将允许neo4j访问系统文件，这样做是**不建议**的

注意linux 系统的用户访问权限的问题

#### 使用header 导入 csv文件

article_with_head.csv

```
Id,Name,Year
1,ABBA,1992
2,Roxette,1986
3,Europe,1979
4,The Cardigans,1992
```

注意语法有稍微的改变

```
LOAD CSV WITH HEADERS FROM 'file:///article_with_head.csv' AS line
CREATE (:Artist { name: line.Name, year: toInteger(line.Year)})
```

#### 使用自定义字段定界符从CSV文件导入数据

```
LOAD CSV FROM 'file:///article_fieldterminator.csv' AS line FIELDTERMINATOR ';'
CREATE (:Artist { name: line[1], year: toInteger(line[2])})
```

#### 导入大量数据

可以指定读取多少行之后提交，防止数据过大，减少内存开销

```
USING PERIODIC COMMIT LOAD CSV FROM 'file:///artists.csv' AS line CREATE (:Artist { name: line[1], year: toInteger(line[2])})
```

#### 设置定期提交的速率

```
USING PERIODIC COMMIT 500 LOAD CSV FROM 'file:///artists.csv' AS line CREATE (:Artist { name: line[1], year: toInteger(line[2])})
```