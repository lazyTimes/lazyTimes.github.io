---
title: ElasticSearch6.2.4 踩坑
subtitle: Elasticsearch 踩坑 - 6.2.4 版本
author: lazytime
url_suffix: note9
tags:
  - Elasticsearch
categories:
  - Elasticsearch
keywords: Elasticsearch
description: Elasticsearch 踩坑 - 6.2.4 版本
copyright: true
date: 2020-07-26 21:32:58
---

# Elasticsearch 踩坑 - 6.2.4 版本

# 目标

## 自定义评分器

**可以根据自己的规则去调整搜索结果的评分，修改权重**

1. 需要根据用户标签以及网站标签，收集用户最关注的标签内容，进行标签标记
2. 收集数据生成可视化报告
3. 关键字查询，基于用户的网站标签内容建设
4. 形成综合信息关联库。以核心用户标签和核心业务标签为参考，对重点资源进行归纳合并，形成重点主题资源关联关系对应表。
5. 设计一个标签关联模块，可以由运维进行标签关联的配置,即给用户分配权重

<!-- more -->

## 智能推荐：

搜索优化查询

# 目录



- [Elasticsearch 踩坑 - 6.2.4 版本](#elasticsearch-踩坑-624-版本)
- 目标
  - [自定义评分器](#自定义评分器)
  - [智能推荐：](#智能推荐)
- [目录](#目录)
- 官方改变1
  - [RestFul请求需要加入Header](#restful请求需要加入header)
- 官方改变2：
  - [分词器 N元语法ngram](#分词器-n元语法ngram)
- \1. 基本使用
  - [1.1 创建索引](#11-创建索引)
  - [1.2 查看索引信息:](#12-查看索引信息)
  - 1.3 创建类型
    - [1.3.2 指定id的方式：](#132-指定id的方式)
    - [1.3.1 不指定id的方式](#131-不指定id的方式)
  - [1.4 创建一个文档](#14-创建一个文档)
- \2. 基础搜索
  - 2.1 使用query 进行搜索
    - [原始方式](#原始方式)
    - kibana方式
      - [2.1.1 如果只想查找一个字段，可以使用term](#211-如果只想查找一个字段可以使用term)
  - 2.2 过滤器
    - [2.2.1 ES6以下：](#221-es6以下)
    - [2.2.2 ES6及以上：](#222-es6及以上)
  - 2.3 应用聚集
    - [2.3.1 ES6以下：](#231-es6以下)
    - [2.3.2 ES6及以上：](#232-es6及以上)
  - [2.4 通过Id获取文档](#24-通过id获取文档)
- \3. elasticsearch配置
  - [3.1 指定集群名称：](#31-指定集群名称)
  - [3.2 指定详细的日志记录](#32-指定详细的日志记录)
  - 3.3 调整虚拟机大小
    - [3.3.1 linux 使用如下命令：](#331-linux-使用如下命令)
    - [3.3.2 使用elasticsearch.yml修改启动堆内存大小](#332-使用elasticsearchyml修改启动堆内存大小)
    - [3.3.3 使用修改bin/elasticsearch.in.sh 修改](#333-使用修改binelasticsearchinsh-修改)
  - 3.4 集群中加入节点
    - [3.4.1 查看节点状态](#341-查看节点状态)
- \4. 映射_mapping
  - 4.1 检索和自定义映射
    - [4.1.1 获取目前的映射](#411-获取目前的映射)
    - [4.1.2 获取自动生成的映射](#412-获取自动生成的映射)
    - 4.1.3 定义新的映射
      - [es6以下版本](#es6以下版本)
      - [es6以上版本](#es6以上版本)
    - [4.1.4 改变现有字段类型的方法：](#414-改变现有字段类型的方法)
  - 4.2 用于定义文档字段的核心类型
    - [4.2.1 定义映射当中的index选项](#421-定义映射当中的index选项)
    - [4.2.2 如何创建嵌套类型：](#422-如何创建嵌套类型)
    - [4.2.3 定义多字段和数组类型](#423-定义多字段和数组类型)
  - 4.3 使用预定义字段
    - [4.3.1 控制如何存储和搜索文档](#431-控制如何存储和搜索文档)
  - 4.4 更新现有文档
    - [4.4.1 文档更新包含如下步骤：](#441-文档更新包含如下步骤)
  - 4.5 删除文档
    - [4.5.1 删除文档的方式](#451-删除文档的方式)
    - [4.5.2 删除文档](#452-删除文档)
    - [4.5.3 删除单个文档](#453-删除单个文档)
    - [4.5.4 删除映射类型和删除查询匹配的文档](#454-删除映射类型和删除查询匹配的文档)
    - [4.5.5 删除索引](#455-删除索引)
    - [4.5.6 关闭和开发索引](#456-关闭和开发索引)
- \5. 版本控制
  - 5.1 乐观锁
    - [5.1.1 内部版本控制](#511-内部版本控制)
    - [5.1.2 外部版本控制](#512-外部版本控制)
  - [5.2 自动重试](#52-自动重试)
- \6. 搜索与高级搜索分析
  - [6.1 搜索请求的基本模块](#61-搜索请求的基本模块)
  - 6.2 基于url的搜索请求
    - [6.2.1 基于from 和szize的参数来实现结果分页](#621-基于from-和szize的参数来实现结果分页)
    - [6.2.2 改变结果的顺序](#622-改变结果的顺序)
    - [6.2.3 搜索结果当中限制_source 字段](#623-搜索结果当中限制_source-字段)
    - [6.2.4 匹配查询](#624-匹配查询)
  - 6.3 基于请求主体的搜索请求
    - [6.3.1 使用from 和 size，参数结果分页](#631-使用from-和-size参数结果分页)
    - 6.3.2 过滤返回_source 内容
      - [在source字段当中返回通配符](#在source字段当中返回通配符)
      - [通过include以及exclude 过滤返回_source的内容](#通过include以及exclude-过滤返回_source的内容)
    - [6.3.3 基于结果的排序](#633-基于结果的排序)
  - 6.4 实践中的基础模块
    - [6.4.1 理解回复的内容](#641-理解回复的内容)
  - 6.5 查询和过滤器DSL （重点）
    - 6.5.1 match 查询和term过滤器
      - [过滤器和查询器有什么不同：](#过滤器和查询器有什么不同)
      - [es6以下 以及旧版本使用的方式](#es6以下-以及旧版本使用的方式)
      - [这里需要注意下面使用的是es6以及以上版本](#font-colorred这里需要注意下面使用的是es6以及以上版本font)
  - 6.6 常用的基础查询和过滤器
    - [6.6.1 match_all 的使用场景](#661-match_all-的使用场景)
    - 6.6.2 query_string 查询
      - [使用query_string 进行复杂的查询](#使用query_string-进行复杂的查询)
    - 6.6.3 term 查询和 term过滤器
      - [使用term 过滤器的得分，和term查询器的得分对比](#使用term-过滤器的得分和term查询器的得分对比)
    - [6.6.4 terms 查询](#664-terms-查询)
    - 6.5.5 match查询和 term过滤器
      - [布尔查询行为](#布尔查询行为)
      - 词组（phrase ）查询的方式
        - [es5版本以及以下的语法：](#es5版本以及以下的语法)
        - [es6以及以上要使用以下语法](#es6以及以上要使用以下语法)
    - 6.5.6 phrase_prefix 查询
      - [es5.x 版本以及以下的语法：](#es5x-版本以及以下的语法)
      - [es6.x 需要改成如下写法](#es6x-需要改成如下写法)
      - [使用multi_match 匹配多个字段](#使用multi_match-匹配多个字段)
  - 6.7 组合查询和复合查询
    - [6.7.1 bool查询](#671-bool查询)
    - [6.7.2 bool 的过滤器](#672-bool-的过滤器)
  - 6.8 超越match 和 过滤器的查询
    - 6.8.1 range 查询和过滤器
      - [模拟数据](#模拟数据)
      - [现在需要查找出 11 年到 16年的数据](#现在需要查找出-11-年到-16年的数据)
    - 6.8.2 prefix 查询和过滤器
      - [在过滤器当中使用](#在过滤器当中使用)
    - [6.8.3 wildcard 查询](#683-wildcard-查询)
  - 6.9 使用过滤器查询字段的存在性
    - [6.9.1 exists 过滤器](#691-exists-过滤器)
    - 6.9.2 missing 过滤器（6.x以上版本已经删除）
      - [作为测试,先插入一条增加一个字段的数据](#作为测试先插入一条增加一个字段的数据)
      - [接下来，查找字段里没有的值,这样写是错误的，没有正确写法（6.x）](#接下来查找字段里没有的值这样写是错误的没有正确写法6x)
    - 6.9.3 将任何查询转为过滤器
      - [缓存过滤器](#缓存过滤器)
    - [6.9.4 为任务选择最好的查询](#694-为任务选择最好的查询)
- \7. 分析数据
  - 7.1 什么是分析数据
    - [7.1.1 字符过滤](#711-字符过滤)
    - [7.1.2 切分为分词](#712-切分为分词)
    - [7.1.3 分词过滤器](#713-分词过滤器)
    - [7.1.4 分词索引](#714-分词索引)
  - 7.2 为文档使用分析器
    - 7.2.1 创建索引的过程中，添加定制分析器
      - [旧版本的使用方式](#旧版本的使用方式)
      - [网上的案例：](#网上的案例)
      - [将书上的案例改为6.x支持的形式，格式如下:](#将书上的案例改为6x支持的形式格式如下)
    - [7.2.2 使用elasticsearch.yml 的配置文件进行配置](#722-使用elasticsearchyml-的配置文件进行配置)
    - 7.2.3 在映射中指定某个字段的分析器
      - [书本的方式已经不适用于6.x版本了，根据官方文档的内容要进行如下定义](#书本的方式已经不适用于6x版本了根据官方文档的内容要进行如下定义)
  - 7.3 使用API来分析文本
    - [7.3.1 使用_analyzer 内置API来进行分析分析器解析步骤](#731-使用_analyzer-内置api来进行分析分析器解析步骤)
    - [7.3.2 分析中指定自定义分析器](#732-分析中指定自定义分析器)
    - 7.3.3 使用组合即时创建分析器
      - [使用curl方式](#使用curl方式)
      - [使用kibana方式](#使用kibana方式)
    - [7.3.4 使用基于某个字段的映射分析](#734-使用基于某个字段的映射分析)
    - [7.3.5 使用词条向量API来学习索引词条](#735-使用词条向量api来学习索引词条)
  - 7.4 分析器/分词器和分词过滤器
    - [7.4.0 分析器的概览](#740-分析器的概览)
    - 7.4.1 内置分析器
      - [标准分析器](#标准分析器)
      - [**简单分析器**](#简单分析器)
      - [**空白分析器**](#空白分析器)
      - [**停止分析器**](#停止分析器)
      - [**关键字分析器**](#关键字分析器)
      - [**模式分析器**](#模式分析器)
      - [**语言分析器**](#语言分析器)
      - [**指纹分析器**](#指纹分析器)
      - [雪球分析器](#雪球分析器)
    - 7.4.2 分词器
      - [**标准分词器**](#标准分词器)
      - [**字母分词器**](#字母分词器)
      - [**小写分词器**](#小写分词器)
      - [**空格分词器**](#空格分词器)
      - [UAX URL电子邮件令牌生成器](#uax-url电子邮件令牌生成器)
      - [经典分词器](#经典分词器)
      - [泰语分词器](#泰语分词器)
    - 7.4.3 分词过滤器
      - [标准分词过滤器](#标准分词过滤器)
      - [ASCII Folding Token Filter（ASCII折叠过滤器）](#ascii-folding-token-filterascii折叠过滤器)
      - [Flatten Graph Token Filter（图形分词过滤器）](#flatten-graph-token-filter图形分词过滤器)
      - [Length Token Filter（长度分词过滤器）](#length-token-filter长度分词过滤器)
      - [Lowercase token filter（小写分词过滤器）](#lowercase-token-filter小写分词过滤器)
      - [Uppercase Token Filter（大写分词过滤器）](#uppercase-token-filter大写分词过滤器)
      - [NGram Token Filter（N元语法过滤器）](#ngram-token-filtern元语法过滤器)
      - [Edge NGram Token Filter（侧边N元语法过滤器）](#edge-ngram-token-filter侧边n元语法过滤器)
    - 7.4.4 N元语法的使用场景以及使用案例
      - [使用案例:](#使用案例)
      - [Shingle Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/analysis-shingle-tokenfilter.html)（滑动窗口分词过滤器）
  - 7.5 提取词干
    - 7.5.1 算分提取词干
      - [选取kstem 作为案例测试](#选取kstem-作为案例测试)
    - [7.5.2 使用字典提取词干](#752-使用字典提取词干)
- \8. 使用相关性进行搜索
  - 8.1 es 的打分机制
    - [8.1.1 文档是如何运作的：](#811-文档是如何运作的)
    - [8.1.2 词频](#812-词频)
    - [8.1.3 逆文档词频](#813-逆文档词频)
    - [8.1.4 Lucene 评分公式](#814-lucene-评分公式)
  - 8.2 其他打分方式
    - [如何指定索引的打分方式？](#如何指定索引的打分方式)
  - [8.3 索引期间Boosting](#83-索引期间boosting)
  - 8.4 查询期间的Boosting
    - [8.4.1 跨越多个字段的查询](#841-跨越多个字段的查询)
    - [8.4.2 使用multi_match 查询，指定整个Boost](#842-使用multi_match-查询指定整个boost)
    - [8.4.3 使用特殊符号^对于name 进行boost](#843-使用特殊符号对于name-进行boost)
    - [8.4.4 query_string 中对于针对单个的词条进行boost](#844-query_string-中对于针对单个的词条进行boost)
  - 8.5 使用"解释(explain)"来理解文档评分
    - [8.5.1 解释一篇文档不匹配的原因](#851-解释一篇文档不匹配的原因)
  - 8.6 使用查询再打分来减小评分操作影响
    - [8.6.1 官方文档介绍 6.2版本](#861-官方文档介绍-62版本)
    - [8.6.2 使用rescore 特性，对于匹配文档子集再评分](#862-使用rescore-特性对于匹配文档子集再评分)
  - 8.7 function_score 来定制得分
    - [8.7.1 function_score 的基本结构](#871-function_score-的基本结构)
    - 8.7.2 weight 函数
      - [可以指定多个weight函数](#可以指定多个weight函数)
    - [8.7.3 合并得分](#873-合并得分)
    - [8.7.4 field_value_factor 函数](#874-field_value_factor-函数)
    - [8.7.5 在function_score 查询中使用field_value_factor](#875-在function_score-查询中使用field_value_factor)
  - 8.8 painless脚本(5.0后go语言脚本支持已删除)
    - [8.7.5 random_score 函数](#875-random_score-函数)
    - 8.7.6 衰减函数
      - [官方文档](#官方文档)
      - [配置选项](#配置选项)
  - [8.9 使用脚本排序](#89-使用脚本排序)
- \9. Elasticsearch REST API 的学习使用
  - [9.0 API官网](#90-api官网)
  - 9.1 REST 低级 Api
    - [9.1.1 起步](#911-起步)
    - [9.1.2 初始化](#912-初始化)
    - [9.1.3 执行请求](#913-执行请求)
    - [9.1.4 接受响应体的处理](#914-接受响应体的处理)
  - 9.2 REST 高级 API
    - [9.2.1 机器翻译介绍](#921-机器翻译介绍)
    - [9.2.2 ](#922-兼容性)[兼容性](https://github.com/elastic/elasticsearch/edit/6.2/docs/java-rest/high-level/getting-started.asciidoc)
    - [9.2.3 起步](#923-起步)
    - [9.2.4 创建一个rest 连接](#924-创建一个rest-连接)
    - 9.2.5 【6.2.4】版本 支持的相关API （重点）
      - [创建索引](#创建索引)
      - [删除索引](#删除索引)
      - [开放索引](#开放索引)
      - [关闭索引](#关闭索引)
    - 9.2.6 索引本身的API
      - [直接根据索引创建一个文档需要如下参数](#直接根据索引创建一个文档需要如下参数)
      - [提供文档来源的多种方式](#提供文档来源的多种方式)
      - [可选参数](#可选参数)
      - [获取响应内容](#获取响应内容)
      - [根据上述内容整合](#根据上述内容整合)
- \10. Elasticsearch JAVA API 的学习使用
  - 10.1 起步：
    - [10.1.1 Maven 依赖](#1011-maven-依赖)
- \11. Es painless 脚本插件使用（Elasticsearch script plugin）
  - 11.1 前置条件：windows安装gradle
    - [11.1.1 官方下载地址](#1111-官方下载地址)
    - [11.1.2 将gradle放入合适位置，将gradle加入path环境变量](#1112-将gradle放入合适位置将gradle加入path环境变量)
    - [11.1.3 配置阿里云的镜像](#1113-配置阿里云的镜像)
- N kibana 学习es的所有内容
  - [2019.10.14](#20191014)
  - [2019.10.15](#20191015)



# 官方改变1

## RestFul请求需要加入Header

> -H "Content-Type: application/json"

```
curl -H "Content-Type: application/json" -XPUT 'localhost:9200/get-together/group/1?pretty' -d '{"name":"elastice denver","organizer":"Lee"}'
```

# 官方改变2：

## 分词器 N元语法ngram

分词器 N元语法ngram 由于在实际使用发生过巨大差异事故，在6.2.4 提示是被弃用的，但是后面社区提议在实际需求中使用发现没有效果，与开发交流过后，官方在7.0之后将弃用删除，继续使用ngram

> 所以在这个6.2.4 版本中使用ngram 分词器 的参数 min_gram 和 max_gram 无法作用于N元语法

# 1. 基本使用

## 1.1 创建索引

```
PUT /lib/

{

  "settings":{

      "index":{

        "number_of_shards": 5,

        "number_of_replicas": 1

        }

      }

}
```

## 1.2 查看索引信息:

```
GET /lib/_settings
curl -XGET '192.168.92.180:9200/lib/_settings?pretty'
{
  "lib" : {
    "settings" : {
      "index" : {
        "creation_date" : "1570895449310",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "KF8Kfs3SSBKPhZX4pX5iVw",
        "version" : {
          "created" : "6020499"
        },
        "provided_name" : "lib"
      }
    }
  }
}
GET _all/_settings
curl -XGET '192.168.92.180:9200/_all/_settings?pretty'
{
  "new-index" : {
    "settings" : {
      "index" : {
        "creation_date" : "1570877785070",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "0HE2lh_gS96UPRCIWMqaZA",
        "version" : {
          "created" : "6020499"
        },
        "provided_name" : "new-index"
      }
    }
  },
  "get-together" : {
    "settings" : {
      "index" : {
        "creation_date" : "1570878100116",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "t1a2Kwx-QvCjF-pSuTGntQ",
        "version" : {
          "created" : "6020499"
        },
        "provided_name" : "get-together"
      }
    }
  },
  "lib" : {
    "settings" : {
      "index" : {
        "creation_date" : "1570895449310",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "KF8Kfs3SSBKPhZX4pX5iVw",
        "version" : {
          "created" : "6020499"
        },
        "provided_name" : "lib"
      }
    }
  }
}
```

## 1.3 创建类型

### 1.3.2 指定id的方式：

```
PUT /lib/user/2
{
  "first_name": "book",
  "last_name": "lin lin",
  "age": 20,
  "birthday": "2019-4-28",
  "interesting": [
    "football"
  ]
}
```

### 1.3.1 不指定id的方式

```
POST /lib/user/
{
    "first_name": "book",
  "last_name": "lin lin",
  "age": 20,
  "birthday": "20191122",
  "interesting": [ "football"]
}

POST /lib/user/
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         23,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

## 1.4 创建一个文档

当使用curl 的时候， 他看上去是这样的

```
curl -H "Content-Type: application/json" -XPUT '192.168.92.180:9200/lib/user/1' -d \
> '{
> "first_name":"Jane",
> "last_name":"Smith",
> "age":32,
> "about":"I like to collect rock albums",
> "interests": ["music"]
> }'
```

结果

```
{
	"_index": "lib",
	"_type": "user",
	"_id": "1",
	"_version": 1,
	"result": "created",
	"_shards": {
		"total": 2,
		"successful": 1,
		"failed": 0
	},
	"_seq_no": 0,
	"_primary_term": 1
}
```

# 2. 基础搜索

## 2.1 使用query 进行搜索

### 原始方式

curl -H "Content-Type: application/json"

```
curl -H "Content-Type: application/json" -XGET '119.23.219.183:9200/lib/user/_search?pretty' -d '{"query":{“defualt”: "field_name", "query_string": {"query": "jone"}}}'
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "lib",
        "_type" : "user",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "first_name" : "Jone",
          "last_name" : "pig",
          "age" : 11,
          "birthday" : "2019-11-18",
          "interesting" : [
            "basketball"
          ]
        }
      }
    ]
  }
}
```

### kibana方式

```
GET /lib/user/_search?pretty 
{
  "query":{
    "query_string": {
      “defualt”: "field_name"
      "query": "jone"
    }
  }
}
```

#### 2.1.1 如果只想查找一个字段，可以使用term

```
GET /lib/user/_search?pretty 
{
  "query":{
    "term": {
      "first_name": "jone"
    }
  }
}
```

## 2.2 过滤器

### 2.2.1 ES6以下：

旧版本使用如下方式 **es6版本以下**

```
GET /lib/user/_search
{
  "query":{
    "filtered":{
      "filter": {
        "term":{
          "first_name": "jone"
        }
      }
    }
  }
}
```

### 2.2.2 ES6及以上：

新版本的过滤器要使用如下方式

```
GET /lib/user/_search
{
  "query":{
    "bool": {
      "filter": {
        "term": {
          "first_name": "jone"
        }
      }
    }
  }
}
```

## 2.3 应用聚集

### 2.3.1 ES6以下：

旧版本使用如下方式 **es6版本以下**

```
GET /lib/user/_search
{
	"aggregations":{
		"organizers":{
			"terms": { "field" : "organizer" }
		}
	}
}
```

### 2.3.2 ES6及以上：

需求是根据年龄进行划分聚合

```
GET /lib/user/_search
{
  "size": 0, 
  "aggs": {
    
    "organizers":{
      "terms": {
        "field": "age",
        "size": 10
      }
    }
  }
}
```

**（1）size**：查询条数，这里设置为0，因为不关心搜索到的结果，只关心聚合结果，提供效率；

**（2）aggs**：声明这是一个聚合查询，是aggregations的缩写；

结果

```
{
  "took": 47,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "organizers": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": 11,
          "doc_count": 1
        },
        {
          "key": 20,
          "doc_count": 1
        },
        {
          "key": 23,
          "doc_count": 1
        }
      ]
    }
  }
}
```

## 2.4 通过Id获取文档

```
GET /lib/user/1
```

# 3. elasticsearch配置

## 3.1 指定集群名称：

在**elasticsearch.yml** 文件当中指定

```
cluster.name: xxxx
```

## 3.2 指定详细的日志记录

1. 主要日志

   基础的日志

2. 慢搜索日志

   多于半秒的查询都会被记录（默认）

3. 慢索引日志

   多于半秒的查询都会被记录（默认）

在**elasticsearch.yml** 文件当中指定,查看所有的日志

```
rootLogger: TRACE, console, file
```

## 3.3 调整虚拟机大小

### 3.3.1 linux 使用如下命令：

```
export ES_HEAP_SIZE=500m bin/elasticsearch
```

### 3.3.2 使用elasticsearch.yml修改启动堆内存大小

### 3.3.3 使用修改bin/elasticsearch.in.sh 修改

\#!/bin/sh 后面加入ES_HEAP_SIZE=500m

## 3.4 集群中加入节点

### 3.4.1 查看节点状态

```
GET _cat/shards?v
```

# 4. 映射_mapping

使用映射来定义文档：

注意同一个索引存在多个类型，所有映射也可以多个类型

但是多个类型可能存在文档里面相同名称的字段

## 4.1 检索和自定义映射

### 4.1.1 获取目前的映射

```
GET /lib/user/_mapping?pretty
```

结果

```
{
  "lib": {
    "mappings": {
      "user": {
        "properties": {
          "about": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "age": {
            "type": "long"
          },
          "birthday": {
            "type": "date"
          },
          "first_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "interesting": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "interests": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "last_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

### 4.1.2 获取自动生成的映射

使用下面的实例：

第一步：创建一个新索引

```
PUT /get-together/new-event/1
{
  "name":"late night with elastic",
  "date":"2013-10-25T19:00"
}
```

第二步： 获取当前映射

```
GET /get-together/_mapping/new-event?pretty
```

结果

```
{
  "get-together": {
    "mappings": {
      "new-event": {
        "properties": {
          "date": {
            "type": "date"
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

### 4.1.3 定义新的映射

#### es6以下版本

```
PUT /get-together/_mapping/new-event
{
  "new-event":{
    "properties":{
      "host": {
        "type": "string"
      }
    }
  }
}
```

#### es6以上版本

```
PUT /get-together/_mapping/new-event
{
  "properties": {
    "host":{
      "type": "keyword"
    }
  }
}
```

此时获取映射会发现新增了一个映射内容

### 4.1.4 改变现有字段类型的方法：

1. 删除 new-evenets 类型的所有数据
2. 设置新的映射类型以及字段类型
3. 再次索引所有数据

## 4.2 用于定义文档字段的核心类型

### 4.2.1 定义映射当中的index选项

1. anaLyzed(默认) 将大写转为小写，字符串分解为单词
2. not_analyzed 分析过程略过，变成单独的词条索引
3. no 索引略过，没有词条产生
   1. 节省存储空间，无须在该字段搜索的时候使用

### 4.2.2 如何创建嵌套类型：

```
PUT /get-together/_mapping/new-event
{
  "properties": {
    "test": {
      "type": "text"
    },
    "user":{
      "type": "nested",
      "properties": {
        "myname":{
          "type":"text"
        }
      }
    }
  }
}
```

### 4.2.3 定义多字段和数组类型

## 4.3 使用预定义字段

### 4.3.1 控制如何存储和搜索文档

1. _source 存储原有内容

   1. 按照原有格式存储原有文档
   2. enable可以设置true,false
   3. 返回某些特定字段在url指定使用fields=xxx,如果需要多个字段用逗号隔开
   4. 使用"store": "yes" 存储特定的字段

2. _all 索引一切

   1. 不指定字段名称默认是_all查找
   2. 默认隐藏属性为"**include_in_all**" ， 可以设置为false不被包含

3. _uid 识别文档

   1. _type和_id的结合体
   2. es内部使用_uid 来确定唯一文档的身份

4. _index 在文档中存储索引名称

   1. 默认情况下是关闭的
   2. 如果需要了解其内容，需要实现使用put 请求将enabled 设置为true

5. _update 更新现有文档

   1. 默认使用go语言进行脚本的更新操作

   ```
   POST /lib/user/1/_update
   {
     "doc": {
       "first_name":"sss"
     }
   }
   ```

6. upsert 创建不存在的文档（注意此时5不存在）

   ```
   POST /lib/user/5/_update
   {
     "doc":{
       "age":5
     },
     "upsert": {
       "myname": "elastic",
       "age": 4
     }
   }
   ```

7. _open 以及 _close 分别表示开放索引和关闭索引

## 4.4 更新现有文档

### 4.4.1 文档更新包含如下步骤：

1. 检索文档
2. 处理文档
3. 重新索引文档
4. 先前文档被删除
5. 完成

## 4.5 删除文档

### 4.5.1 删除文档的方式

1. 删除单个文档或者一组文档
   1. 标记为删除
   2. 延迟用异步方式排出
2. 删除整个索引（删除多组文档的特例）
   1. 移除和索引所有相关的文件
   2. 效率高
3. 关闭索引
   1. 不允许读写操作，数据也不进内存
   2. 但是索引还是在磁盘内
   3. 可以再次打开关闭的索引

### 4.5.2 删除文档

1. 通过id删除单个文档
2. 单个请求删除多个文档
   1. 比单独删除效率高
3. 删除映射类型，包括其中文档
4. 删除匹配某个查询的所有文档

### 4.5.3 删除单个文档

```
DELETE /lib/user/5
```

为了防止外部的并发更新与删除的操作：

es在一段时间内保留这篇文档的版本，拒绝比删除文档更低的更新操作

默认时间为60秒

yml或者索引配置的 **index.gc_deletes** 设置保留时间

### 4.5.4 删除映射类型和删除查询匹配的文档

```
DELETE /get-together/_query?q=elasticsearch
```

注意点：

1. 类型名称只是文档的另一个字段
2. 索引的所有文档，无论属于哪个映射类型，都存在同一个分片
3. **针对删除类型和删除完整索引两者的性能进行比较的时候，删除类型比删除索引要更长的时间和更多的资源**

### 4.5.5 删除索引

```
DELETE /myindex[,xxx,xx] 删除一个或者多个索引
DELETE /_all 删除所有的索引
```

> _all 删除所有文档是否过于危险？
>
> elasticsearch.yml 文件中
>
> action.destructive_requires_name: true
>
> 预防这种情况发生， 可以**拒绝**delete _all 操作
>
> 以及索引名称当中的**通配符**

注意这里的删除也是标记删除：在分段与合并中进行合并之后才会真实删除操作

> 分段与合并是什么？
>
> 1. 一个分段是建立索引的时候创建的一块Lucene 索引
> 2. 当es在分片上查询的时候，Luncene需要查询所有的分段
> 3. 合并文档意味着分段的合并以及I/O和CPU读写操作（合并操作是异步进行的）
> 4. 索引操作会产生许多小分段，需要定期进行合并

### 4.5.6 关闭和开发索引

使用POST 加上 _open 以及 _close 对于索引进行开放和关闭

1. 一旦索引被关闭，内存唯一痕迹是元数据，如名字以及分片的位置
2. 如果条件允许，关闭索引比删除索引要好

# 5. 版本控制

## 5.1 乐观锁

### 5.1.1 内部版本控制

PUT /lib/user/4?version=3

### 5.1.2 外部版本控制

version_type=external + 版本号

## 5.2 自动重试

retry_on_conflict=3(版本号) 自动重试

# 6. 搜索与高级搜索分析

```
curl '/_search'
curl '/xxx/_search' 指定索引
curl '/xxx/xxx/_search' 搜索类型
curl '/_all/xxx/_search' 所有索引中搜索时事件类型
curl '/*/xxx/_search' 通配符搜索所有
curl '/xxxx,xxxx/xxx,xxxx/_search' 多个类型搜索
curl '/+get-toge*,-get-together/_search' 搜索以get-toge 开头的索引，但是不包括get-together
```

## 6.1 搜索请求的基本模块

1. query：搜索请求最重要的部分，配置了基于评分返回最佳的文档
2. size：代表返回文档数量
3. from：和size一起使用，用于分页操作（注意：为了知道第二页的10项结果，必须先知道计算前20的结果，也就是说，**越往后的翻页搜索代价越大**）
4. _source：指定字段如何返回
5. sort：默认排序基于文档得分

## 6.2 基于url的搜索请求

### 6.2.1 基于from 和szize的参数来实现结果分页

```
GET /lib/user/_search?from=2&size=2
```

from=2 表示从第三条开始

size=2 表示读取2条记录

### 6.2.2 改变结果的顺序

```
GET /lib/_search?sort=age:_id
```

### 6.2.3 搜索结果当中限制_source 字段

```
GET /lib/_search?sort=age:_id&_source=first_name,age
```

### 6.2.4 匹配查询

```
GET /lib/_search?sort=age:_id&_source=first_name,age
```

## 6.3 基于请求主体的搜索请求

### 6.3.1 使用from 和 size，参数结果分页

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 2
}
```

### 6.3.2 过滤返回_source 内容

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["first_name", "age"]
}
```

#### 在source字段当中返回通配符

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["*_name", "a*"]
}
```

如果是数组，可以指定: _source: ["name.*", "address.*"]

#### 通过include以及exclude 过滤返回_source的内容

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": {
    "includes": ["*_name", "age"],
    "excludes": ["first_name"]
  }
}
```

include: 包含匹配

excludes: 排除

### 6.3.3 基于结果的排序

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
 
    "_score"
  ]
}
```

结果

```
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 15,
    "max_score": null,
    "hits": [
      {
        "_index": "lib",
        "_type": "user",
        "_id": "BrimyW0B5sayAd6P9LGl",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "CbimyW0B5sayAd6P-LHO",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "D7imyW0B5sayAd6P_LH0",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "ELimyW0B5sayAd6P_bGM",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "BbimyW0B5sayAd6P57Hs",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "B7imyW0B5sayAd6P9rG3",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "EbimyW0B5sayAd6P_rEo",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "1",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "CLimyW0B5sayAd6P97Hz",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      },
      {
        "_index": "lib",
        "_type": "user",
        "_id": "CrimyW0B5sayAd6P-bGO",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        },
        "sort": [
          32,
          1
        ]
      }
    ]
  }
}
```

## 6.4 实践中的基础模块

结合上面的几个点，总结出了下面一个查询内容

```
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2,
  "_source": ["*_name", "age"],
  "sort": [
    {
      "age":"desc"  
    }
    
  ]
}
```

### 6.4.1 理解回复的内容

注意点： 如果没有存储文档的_source 和 fields, 无法从es 获取数值

## 6.5 查询和过滤器DSL （重点）

### 6.5.1 match 查询和term过滤器

```
GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": "Jane"
    }
  }
}
```

注意：此处 **jane** 是否大写对于查询结果没有影响

#### 过滤器和查询器有什么不同：

查询器：为特定的词条计算得分

过滤器：

1. “文档是否匹配这个查询” 返回简单的是或者否
2. 限制了需要计算得分的文档数量
3. 根据过滤器的种类，可以再“位集合”中缓存结果
4. 允许手动指定一个过滤器是否应该被缓存

> 基于如上原因，过滤器会比查询器要快上很多，并且查询结果可以被缓存

#### es6以下 以及旧版本使用的方式

```
{
    "query":{
        "filtedred":{ -- 表示指定一个附上过滤器的查询
            "query":{ -- 指定查询器
                "match": "xxx"
            },
            "filter":{ -- 指定过滤器
                "term":{
                    ....
                }
            }
        }
    }
}
```

仅做了解即可，新版更为简单易用

#### 这里需要注意下面使用的是es6以及以上版本

> es6 之后使用
>
> bool:{
>
> must 表示匹配操作
>
> 可以执行match 操作
>
> filter:
>
> 对于结果进行过滤,
>
> 这要比在全部搜索中去匹配要快的多
>
> }

```
GET /lib/user/_search
{
  
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "about": "hide"
          }
        }
      ], 
      "filter": {
        "term": {
          "first_name": "wang"
        }
      }
    }
  }
}
```

结果

```
{
  "took": 16,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.2039728,
    "hits": [
      {
        "_index": "lib",
        "_type": "user",
        "_id": "E7joyW0B5sayAd6PEbFA",
        "_score": 1.2039728,
        "_source": {
          "first_name": "wang",
          "last_name": "laji",
          "age": 333,
          "about": "I hide to collect rock albums",
          "interests": [
            "basket"
          ]
        }
      }
    ]
  }
}
```

## 6.6 常用的基础查询和过滤器

### 6.6.1 match_all 的使用场景

1. 希望使用过滤器的时候
2. 希望返回被搜索的索引和类型的全部文档

### 6.6.2 query_string 查询

> 注意事项:
>
> AND 必须大写， 即查询条件的连接符， 否则系统会当做查询字符串的内容进行识别
>
> 查询条件连接符两边必须加上一个空格区分

```
GET /lib/_search?q=wang

GET /lib/_search
{
  "query": {
    "query_string": {
      "default_field": "first_name",
      "query": "first_name:wang AND age:333"
    }
  }
}
```

#### 使用query_string 进行复杂的查询

```
{ tags:search OR tags:lucene} AND create_on:[1999-01-01 TO 2011-01-01]
```

这是一个示例查询

下面为个人实验案例

```
GET /lib/_search
{
  "query": {
    "query_string": {
      "default_field": "first_name",
      "query": "first_name:wang OR first_name:jane AND age:[0 TO 999]"
    }
  }
}
```

> 由于query_string 过于强大：建议的替代方案为使用 term, terms, match 或者 multi_match
>
> 允许你在一个或者多个文档搜索
>
> 或者使用 simple_query_string
>
> 使用 +、-、AND 和 OR 更容易使用的查询语法

### 6.6.3 term 查询和 term过滤器

1. 最简单的几个可执行查询之一
2. 指定需要搜索的词条和字段
3. 了解term(词条)查询

> 注意：被搜索的词条没有经过分析，文档的词条要经过精确匹配才能作为结果返回

#### 使用term 过滤器的得分，和term查询器的得分对比

使用过滤器

```
GET /lib/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "_id": 1
        }        
      }
    }
  }
}
```

结果

```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0,
    "hits": [
      {
        "_index": "lib",
        "_type": "user",
        "_id": "1",
        "_score": 0,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}
```

使用查询器

```
GET /lib/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "first_name": "wang"
          }
        }
      ]
    }
  }
}
```

结果

```
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.2039728,
    "hits": [
      {
        "_index": "lib",
        "_type": "user",
        "_id": "E7joyW0B5sayAd6PEbFA",
        "_score": 1.2039728,
        "_source": {
          "first_name": "wang",
          "last_name": "laji",
          "age": 333,
          "about": "I hide to collect rock albums",
          "interests": [
            "basket"
          ]
        }
      }
    ]
  }
}
```

### 6.6.4 terms 查询

使用多词条查询搜索多个词条

```
GET /lib/_search
{
  "query": {
    "terms": {
      "about": [
        "i",
        "hide"
      ]
    }
  }
}
```

可以强制规定每篇文档中匹配词条的最小数量

mininum_should_match 参数

> 注意：在新的版本当中只有过滤器才能制定该参数

### 6.5.5 match查询和 term过滤器

match查询可以有多重行为方式，最常见就是**布尔(bool)\**和\**词组(phrase)**

#### 布尔查询行为

默认情况match查询使用布尔行为和OR操作符号， 如 "es den" 默认是"es OR den"

```
GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "query": "Jane wang",
        "operator": "or"
      }
    }
  }
}
```

**match 查询的第二个重要行为是作为phrase 查询**

#### 词组（phrase ）查询的方式

> 在6.x已经不支持在math里面使用type， 可以修改为以下语法：

可以设置slop 用来实现自动提示中的跨单词匹配的功能

##### es5版本以及以下的语法：

```
GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "type": "phrase",
        "query": "hide"
      }
    }
  }
}
```

会出现如下报错

```
{
  "error": {
    "root_cause": [
      {
        "type": "parsing_exception",
        "reason": "[match] query does not support [type]",
        "line": 5,
        "col": 17
      }
    ],
    "type": "parsing_exception",
    "reason": "[match] query does not support [type]",
    "line": 5,
    "col": 17
  },
  "status": 400
}
```

##### es6以及以上要使用以下语法

```
GET /lib/user/_search
{
  "query": {
    "match_phrase": {
      "about": {
        "query": "to rock",
        "slop":1
      }
    }
  }
}
```

注意：slop 为0 代表中间不进行跳行匹配,默认为0

### 6.5.6 phrase_prefix 查询

依据词组的最后一个词条进行前缀匹配

对于搜索框自动完成来说，是一个十分重要的功能

> 注意：这种方式最好设置 max_expansions 设置最大的前缀扩展数量

#### es5.x 版本以及以下的语法：

```
GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "type": "phrase_prefix",
        "query": "hide"
      }
    }
  }
}
```

#### es6.x 需要改成如下写法

```
GET /lib/user/_search
{
  "query": {
    "match_phrase_prefix": {
      "first_name": {
        "query": "wa",
        "max_expansions": 5
      }
    }
  }
}
```

结果

```
{
  "took": 24,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.2039728,
    "hits": [
      {
        "_index": "lib",
        "_type": "user",
        "_id": "E7joyW0B5sayAd6PEbFA",
        "_score": 1.2039728,
        "_source": {
          "first_name": "wang",
          "last_name": "laji",
          "age": 333,
          "about": "I hide to collect rock albums",
          "interests": [
            "basket"
          ]
        }
      }
    ]
  }
}
```

bool查询和phrase 查询 对于接受用户输入是很好的选择

#### 使用multi_match 匹配多个字段

和match 多个字段有细微的区别：

允许搜索多个字段中的值

```
POST /lib/user/_search
{
  "query": {
    "multi_match": {
        "query": "wang smith",
        "fields": ["first_name", "last_name"]
    }
  }
}
```

**multi_match 也可以转化为 phrase(词组) 查询 或者 phrase_prefix 查询**

match 查询是核心的查询类型

## 6.7 组合查询和复合查询

### 6.7.1 bool查询

有以下类别：

1. 必须(must)
2. 应该(should)
3. 不能(must_not)

- 如果指定了bool查询的部分是must匹配， 只有匹配上结果才能返回
- 如果指定了bool查询的部分是should匹配， 只有匹配上**指定数量子句**的文档才会被返回
- 如果没有must匹配，至少要匹配一个should子句才能返回
- must_not 子句会使得文档被移除结果集合

下面使用一个综合的例子进行试验

```
POST /lib/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase":  {
            "about": {
              "query": "i to",
              "slop": 1
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "first_name": {
              "value": "wang"
            }
          }
        }
      ],
      "must_not": [
        {
          "query_string": {
            "default_field": "age",
            "query": "first_name:jane"
          }
        }
      ],
      "minimum_should_match": 1 --指定至少shold子句匹配数，满足才能返回结果
    }
  }
}
```

### 6.7.2 bool 的过滤器

ES6 经过调整，已经将bool的查询器移除，只做过滤功能

es存在如下写法:

```
query
	filtered:
        filter:
            bool:
                must:
                should:
                must_not:
```

> 小贴士：
>
> mininum_should_match 默认有一些隐藏特性
>
> 如果指定了must 子句
>
> **默认值就会变为0**,
>
> 如果没有Must子句
>
> **默认就是1**

## 6.8 超越match 和 过滤器的查询

### 6.8.1 range 查询和过滤器

#### 模拟数据

```
POST /myindex/test/5
{
  "name": "CHENLONG_looo",
  "age": 66,
  "birthday": "2016-10-10"
}

POST /myindex/test/5
{
  "name": "CHENLONG",
  "age": 15,
  "birthday": "2011-02-14"
}

POST /myindex/test/
{
  "name": "lixiaolong",
  "age": 5,
  "birthday": "2019-05-04"
}
```

#### 现在需要查找出 11 年到 16年的数据

```
GET /myindex/test/_search
{
  "query": {
    "range": {
      "birthday": {
        "gte": "2011-06-11",
        "lte": "2020-01-01"
      }
    }
  }
}
```

结果

```
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "myindex",
        "_type": "test",
        "_id": "5",
        "_score": 1,
        "_source": {
          "name": "CHENLONG_looo",
          "age": 66,
          "birthday": "2016-10-10"
        }
      },
      {
        "_index": "myindex",
        "_type": "test",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "lixiaolong",
          "age": 55,
          "birthday": "2019-05-04"
        }
      }
    ]
  }
}
```

range支持字符串的范围匹配，如 "li" 到 "e" 之间的分组

```
GET /myindex/test/_search
{
  "query": {
    "range": {
      "name": {
        "gte": "li",
        "lte": "o"
      }
    }
  }
}
```

range 应该用于过滤而不是查询

### 6.8.2 prefix 查询和过滤器

prefix 查询和过滤器允许给定的前缀来搜索词条,**这里的前缀在搜索之前没有经过分析**

```
GET /myindex/test/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "li"
      }
    }
  }
}
```

匹配 li 开头的词条

#### 在过滤器当中使用

```
GET /myindex/test/_search
{
  "query": {
    "bool": {
      "filter": {
        "prefix": {
          "name": "li"
        }
      }
    }
  }
}
```

注意过滤器会跳过得分计算，所以效率要高于查询

> 注意： 由于搜索前缀不会被分析，所以大小写是进行区分的
>
> 原因：es分析文档和查询的方式引起的

### 6.8.3 wildcard 查询

正则表达式的搜索方式：

更像是 shell 通配符 globbing 的工作方式：

ls *foo?ar

```
GET /myindex/test/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "lix?ao*"
      }
    }
  }
}
```

要注意的是：这种正则匹配并不像match 那样轻量级

如果需要用于系统，要进行数据分析以及测试

## 6.9 使用过滤器查询字段的存在性

### 6.9.1 exists 过滤器

只查找那些特定字段有值的文档

```
GET /myindex/test/_search
{
  "query": {
    "bool": {
      "filter": {
        "exists": {
          "field": "name"
        }
      }
    }
  }
}
```

### 6.9.2 missing 过滤器（6.x以上版本已经删除）

可以搜索字段里没有的值，或者映射的时候默认指定的默认值的文档（映射里的 null_value）

#### 作为测试,先插入一条增加一个字段的数据

```
POST /myindex/test/
{
  "name": "lixiaolong",
  "age": 5,
  "birthday": "2019-05-04",
  "test_field": "zzzzzz"
}
```

#### 接下来，查找字段里没有的值,这样写是错误的，没有正确写法（6.x）

```
GET /myindex/_search
{
  "query": {
    "bool": {
      "filter": {
        "missing": {
          "field": "test_field",
          "existence": true,
          "null_value": true
        }
      }
    }
  }
}
```

### 6.9.3 将任何查询转为过滤器

```
GET /myindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "query_string": {
            "default_field": "name",
            "query": "che*"
          }
        }
      ]
    }
  }
}
```

#### 缓存过滤器

加入 _cache: true 配置

### 6.9.4 为任务选择最好的查询

# 7. 分析数据

## 7.1 什么是分析数据

1. 字符过滤：使用字符过滤器转变字符
2. 文本切分为分词：将文本切分为单个或者多个分词
3. 分词过滤：使用分词过滤器转变为每个分词
4. 分词索引：将这些分词存储到索引中

### 7.1.1 字符过滤

主要是排除一些HTML标签以及特殊符号转为可以搜索是是别的词条

### 7.1.2 切分为分词

标准分词器：根据空格，换行，破折号进行划分单词

### 7.1.3 分词过滤器

最有用和最常用的是**小写分词过滤器**（外国）

中国建议使用 ik 中文分词器

书本的分词过滤为下：

1. 分词转小写
2. 删除"停用词"
3. 同义词添加

### 7.1.4 分词索引

经过分词过滤器过滤之后，将发送到Lucene进行文档索引。最终组成倒排索引

所有不同部分，组成一个分析器

> 搜索时候的分析：
>
> 如果搜索没有达到预期效果，牢记一条
>
> **分析方式和预想的不一致**

## 7.2 为文档使用分析器

以下两种方式指定字段使用的分析器

- 创建索引的时候，为特定索引进行设置
- es配置文件中，配置全局分析器

### 7.2.1 创建索引的过程中，添加定制分析器

参考：

https://blog.csdn.net/wsyw126/article/details/71080285

#### 旧版本的使用方式

```
PUT /mymyindex/
{
  "settings":{
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index": {
      "analysis": {
        "myCustomAnalyzer":{
          "type": "custom",
          "tokenizer": "myCustomTokenizer",
          "filter": ["myCustomFilter1", "myCustomFilter2"],
          "char_filter": ["myCustomCharFilter"]
        }
      }
    },
    "tokenizer": {
      "myCustomTokenizer": {
        "type": "letter"
      }
    },
    "filter": {
      "myCustomFilter1": {
        "type": "lowercase"
      },
      "myCustomFilter2":{
        "type": "kstem"
      }
    },
    "char_filter":{
      "myCustomCharFilter":{
        "type": "mapping",
        "mappings": ["ph=>f", "u=>you"]
      }
    }
  },
  "mappings":{
    
  }
}
```

#### 网上的案例：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "& => and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
        } 
    }
}
```

上面的网上案例使用 _settings 查看结果是这样的

```
{
  "my_index": {
    "settings": {
      "index": {
        "number_of_shards": "5",
        "provided_name": "my_index",
        "creation_date": "1571079347660",
        "analysis": {
          "filter": {
            "my_stopwords": {
              "type": "stop",
              "stopwords": [
                "the",
                "a"
              ]
            }
          },
          "analyzer": {
            "my_analyzer": {
              "filter": [
                "lowercase",
                "my_stopwords"
              ],
              "char_filter": [
                "html_strip",
                "&_to_and"
              ],
              "type": "custom",
              "tokenizer": "standard"
            }
          },
          "char_filter": {
            "&_to_and": {
              "type": "mapping",
              "mappings": [
                "& => and "
              ]
            }
          }
        },
        "number_of_replicas": "1",
        "uuid": "gYyxkcIRRWqLtWhjMd24CQ",
        "version": {
          "created": "6020499"
        }
      }
    }
  }
}
```

#### 将书上的案例改为6.x支持的形式，格式如下:

```
PUT /mymyindex/
{
	"settings": {
		"number_of_shards": 2,
		"number_of_replicas": 1,
		"analysis": {
			"myCustomAnalyzer": {
				"type": "custom",
				"tokenizer": "myCustomTokenizer",
				"filter": ["myCustomFilter1", "myCustomFilter2"],
				"char_filter": ["myCustomCharFilter"]
			},
			"tokenizer": {
				"myCustomTokenizer": {
					"type": "letter"
				}
			},
			"filter": {
				"myCustomFilter1": {
					"type": "lowercase"
				},
				"myCustomFilter2": {
					"type": "kstem"
				}
			},
			"char_filter": {
				"myCustomCharFilter": {
					"type": "mapping",
					"mappings": ["ph=>f", "u=>you"]
				}
			}
		}

	},
	"mappings": {

	}
}
```

### 7.2.2 使用elasticsearch.yml 的配置文件进行配置

目前没有理解如何配置，需要到官方文档去摸索

### 7.2.3 在映射中指定某个字段的分析器

#### 书本的方式已经不适用于6.x版本了，根据官方文档的内容要进行如下定义

注意下文有使用到该索引

```
PUT /mymyindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": { 
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        "my_text": {
          "type": "text",
          "analyzer": "std_folded" 
        }
      }
    }
  }
}
```

## 7.3 使用API来分析文本

### 7.3.1 使用_analyzer 内置API来进行分析分析器解析步骤

下面是官方的测试用例：使用的官方内置的 **standard（标准）**分析器

```
curl -X POST "192.168.92.180:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

结果如下：

```
{
  "tokens" : [
    {
      "token" : "the",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "2",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<NUM>",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 12,
      "end_offset" : 17,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "foxes",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "jumped",
      "start_offset" : 24,
      "end_offset" : 30,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 31,
      "end_offset" : 35,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "the",
      "start_offset" : 36,
      "end_offset" : 39,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "lazy",
      "start_offset" : 40,
      "end_offset" : 44,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "dog's",
      "start_offset" : 45,
      "end_offset" : 50,
      "type" : "<ALPHANUM>",
      "position" : 9
    },
    {
      "token" : "bone",
      "start_offset" : 51,
      "end_offset" : 55,
      "type" : "<ALPHANUM>",
      "position" : 10
    }
  ]
}
```

> POST my_index/_analyze?analyzer=standard 注意：传参数这样使用时 错误的
>
> 需要使用如下方式
>
> POST my_index/_analyze { "analyzer": "standard", "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone." }

### 7.3.2 分析中指定自定义分析器

这里在建立索引的时候用到了上面官方改动案例的索引

```
POST /mymyindex/_analyze
{
  "analyzer": "std_folded",
  "text": "this is test word I AM LAZY BOY"
}
```

### 7.3.3 使用组合即时创建分析器

#### 使用curl方式

6.x 语法有变，必须放在大括号里面

```
curl -X POST "192.168.92.180:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
{
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone.",

 "filter": ["lowercase", "reverse"],

"tokenizer": "whitespace"

}
'
```

此时会发现结果以倒序的方式进行分析

#### 使用kibana方式

```
POST /my_index/_analyze
{
  "tokenizer": "whitespace", 
  "filter": [
    "reverse"
  ],
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
```

大同小异，如果结果没有反转，可能是解析的过程中有其他干扰因素

### 7.3.4 使用基于某个字段的映射分析

```
POST /mymyindex/_analyze
{
  "tokenizer": "whitespace", 
  "filter": [
    "reverse"
  ],
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone.",
  "field": "my_text"
}
```

前提条件是必须是存在索引的字段才能进行映射分析

### 7.3.5 使用词条向量API来学习索引词条

使用_termvector 的端点来获取词条的更多信息

```
GET /get-together/group/3q8nz20BXNiyabSjyH-c/_termvectors
```

虽然可以使用，但是内容已经被官方改动，无法查看词条索引

## 7.4 分析器/分词器和分词过滤器

### 7.4.0 分析器的概览

- 输入文本
- 分析器
  - 字符过滤器
  - 分词器
  - 分词过滤器
- 输出分词

### 7.4.1 内置分析器

**Standard Analyzer**

The `standard` analyzer divides text into terms on word boundaries, as defined by the Unicode Text Segmentation algorithm. It removes most punctuation, lowercases terms, and supports removing stop words.

**Simple Analyzer**

The `simple` analyzer divides text into terms whenever it encounters a character which is not a letter. It lowercases all terms.

**Whitespace Analyzer**

The `whitespace` analyzer divides text into terms whenever it encounters any whitespace character. It does not lowercase terms.

**Stop Analyzer**

The `stop` analyzer is like the `simple` analyzer, but also supports removal of stop words.

**Keyword Analyzer**

The `keyword` analyzer is a “noop” analyzer that accepts whatever text it is given and outputs the exact same text as a single term.

**Pattern Analyzer**

The `pattern` analyzer uses a regular expression to split the text into terms. It supports lower-casing and stop words.

**Language Analyzers**

Elasticsearch provides many language-specific analyzers like `english` or `french`.

**Fingerprint Analyzer**

The `fingerprint` analyzer is a specialist analyzer which creates a fingerprint which can be used for duplicate detection.

**谷歌翻译之后**

#### 标准分析器

“标准”分析器根据Unicode文本分段算法的定义，将文本划分为单词边界上的各个术语。它删除大多数标点符号，小写术语，并支持删除停用词。

#### **简单分析器**

“简单”分析器在遇到非字母字符时会将文本划分为多个词。它小写所有术语。

只使用小写转化为分词器，注意不适用于亚洲的语言，在欧美这种单词划分的语言适用

#### **空白分析器**

每当遇到任何空白字符时，“空白”分析器都会将文本划分为多个词。它不小写。

#### **停止分析器**

“停止”分析器类似于“简单”分析器，但也支持删除停用词。

#### **关键字分析器**

“关键字”分析器是一个“空”分析器，它接受给出的任何文本，并输出与单个术语完全相同的文本。

#### **模式分析器**

“模式”分析器使用正则表达式将文本分割为多个词。它支持小写字母和停用词。

#### **语言分析器**

Elasticsearch提供了许多特定于语言的分析器，例如“ english”或“ french”。

#### **指纹分析器**

“指纹”分析仪是一种专业分析仪，可创建可用于重复检测的指纹。

#### 雪球分析器

除了使用标准分词器和标准分词过滤器，也使用了小写分词过滤器以及停用词过滤器，还使用雪球词干进行词干提取

### 7.4.2 分词器

**Standard Tokenizer**

The `standard` tokenizer divides text into terms on word boundaries, as defined by the Unicode Text Segmentation algorithm. It removes most punctuation symbols. It is the best choice for most languages.

**Letter Tokenizer**

The `letter` tokenizer divides text into terms whenever it encounters a character which is not a letter.

**Lowercase Tokenizer**

The `lowercase` tokenizer, like the `letter` tokenizer, divides text into terms whenever it encounters a character which is not a letter, but it also lowercases all terms.

**Whitespace Tokenizer**

The `whitespace` tokenizer divides text into terms whenever it encounters any whitespace character.

**UAX URL Email Tokenizer**

The `uax_url_email` tokenizer is like the `standard` tokenizer except that it recognises URLs and email addresses as single tokens.

**Classic Tokenizer**

The `classic` tokenizer is a grammar based tokenizer for the English Language.

**Thai Tokenizer**

The `thai` tokenizer segments Thai text into words.

**谷歌翻译之后**

#### **标准分词器**

标准分词器将文本划分为单词边界上的术语，这由Unicode文本分段算法定义。 它删除大多数标点符号。 这是大多数语言的最佳选择。 注意标点符号回进行过滤

```
POST /lib/_analyze
{
  "tokenizer": "standard",
  "text": "I have, photos"
}
```

配置：

| `max_token_length` | The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`. |
| ------------------ | ------------------------------------------------------------ |
| 最大分词数         | 允许设置最大分次数，如果分词长度超过最大分词数，将会进行切割作为下一个分词。默认为255 |

#### **字母分词器**

字母分词器在遇到非字母字符时会将文本分为多个术语

```
POST /lib/_analyze
{
  "tokenizer": "letter",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone"
}
```

结果

```
[ The, QUICK, Brown, Foxes, jumped, over, the, lazy, dog, s, bone ]
```

该`letter`分词器是不可配置

#### **小写分词器**

小写分词器与字母分词器一样，在遇到非字母的字符时，会将文本分为多个词，但所有词都小写。

```
POST _analyze
{
  "tokenizer": "lowercase",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

该`letter`分词器是不可配置

#### **空格分词器**

每当遇到任何空白字符时，空白令牌生成器都会将文本划分为多个术语。(但是不会对大小写进行转化，只判断空白字符)

配置：

| `max_token_length` | The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`. |
| ------------------ | ------------------------------------------------------------ |
| 最大分词数         | 允许设置最大分次数，如果分词长度超过最大分词数，将会进行切割作为下一个分词。默认为255 |

#### UAX URL电子邮件令牌生成器

**uax_url_email** 分词器类似于标准令牌生成器，不同之处在于它将URL和电子邮件地址识别为单个分词。

```
POST _analyze
{
  "tokenizer": "uax_url_email",
  "text": "Email me at john.smith@global-international.com"
}
```

注意分词出来的email类型为

```
{
    "token": "john.smith@global-international.com",
    "start_offset": 12,
    "end_offset": 47,
    "type": "<EMAIL>",
    "position": 3
}
```

配置

| `max_token_length` | The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`. |
| ------------------ | ------------------------------------------------------------ |
| 最大分词数         | 允许设置最大分次数，如果分词长度超过最大分词数，将会进行切割作为下一个分词。默认为255 |

#### 经典分词器

经典的分词器是用于英语的基于语法的分词器。

> 可以说是为英语而生的分词器. 这个分词器对于英文的首字符缩写、 公司名字、 email 、 大部分网站域名.都能很好的解决。 但是, 对于除了英语之外的其他语言，都不是很好使。

```
POST _analyze
{
  "tokenizer": "classic",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]
```

下面是官方文档谷歌翻译

```
该`classic`标记生成器是一种基于语法标记生成器是好的英语语言文档。该令牌生成器具有试探法，可对首字母缩写词，公司名称，电子邮件地址和互联网主机名进行特殊处理。但是，这些规则并不总是有效，并且分词器不适用于英语以外的大多数其他语言：

- 它最多将标点符号拆分为单词，删除标点符号。但是，不带空格的点被认为是令牌的一部分。
- 除非令牌中没有数字，否则它将使用连字符对单词进行拆分，在这种情况下，整个令牌将被解释为产品编号，并且不会拆分。
- 它将电子邮件地址和互联网主机名识别为一个令牌。
```

配置：

| `max_token_length` | The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`. |
| ------------------ | ------------------------------------------------------------ |
| 最大分词数         | 允许设置最大分次数，如果分词长度超过最大分词数，将会进行切割作为下一个分词。默认为255 |

#### 泰语分词器

泰语分词器将泰语文本分成单词。

### 7.4.3 分词过滤器

官方定义了大概20多种分词过滤，涵盖内容较多，以官方6.2.4文档为准

https://www.elastic.co/guide/en/elasticsearch/reference/6.4/analysis-tokenfilters.html

#### 标准分词过滤器

不要以为有多牛逼，其实什么事情都没有干。在以后的版本中有可能扩展功能。只是单纯占位符

#### ASCII Folding Token Filter（ASCII折叠过滤器）

官网翻译介绍：

```
类型为asciifolding的分词过滤器，用于将前127个ASCII字符（“基本拉丁” Unicode块）中不存在的字母，数字和符号Unicode字符转换为等效的ASCII字符。 例：
PUT /asciifold_example
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "default" : {
                    "tokenizer" : "standard",
                    "filter" : ["standard", "asciifolding"]
                }
            }
        }
    }
}
```

允许进行如下配置：

```
PUT /asciifold_example
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "default" : {
                    "tokenizer" : "standard",
                    "filter" : ["standard", "my_ascii_folding"]
                }
            },
            "filter" : {
                "my_ascii_folding" : {
                    "type" : "asciifolding",
                    "preserve_original" : true --默认为false
                }
            }
        }
    }
}
```

翻译版

```
preserve_original： 接受默认值为false的preserve_original设置，但如果为true，则将保留原始令牌并发出折叠后的令牌
```

#### Flatten Graph Token Filter（图形分词过滤器）

> 注意：此功能在Lucene中被标记为实验性的

```
flatten_graph令牌过滤器接受任意图形令牌流，例如“同义词图形令牌过滤器”生成的图形令牌流，并将其展平为适合索引的单个线性令牌链。

这是一个有损耗的过程，因为单独的边路径相互挤压，但是在索引过程中使用图形令牌流时是必需的，因为Lucene索引当前无法表示图形。 因此，最好仅在搜索时应用图形分析器，因为这样可以保留完整的图形结构并为接近查询提供正确的匹配项。

有关此主题及其复杂性的更多信息，请阅读Lucene的TokenStreams实际上是图博客文章。
```

#### Length Token Filter（长度分词过滤器）

将超出最短和最长限制范围的单词过滤掉，如min=2，max=8，不在此范围的单词都会进行过滤

配置项如下

| `min`   | The minimum number. Defaults to `0`.                         |
| ------- | ------------------------------------------------------------ |
| **max** | **The maximum number. Defaults to `Integer.MAX_VALUE`, which is `2^31-1` or 2147483647.** |

```
PUT /length-filter
{
  "settings": {
    "analysis": {
      "filter": {
        "myfilter":{
          "type": "length",
          "max": 8,
          "min": 2
        }
      }, 
      "analyzer": {
        "defulat": {
          "tokenizer": "standard",
          "filter": ["standard", "myfilter"]
        }
      }
    }
  }
}
```

结果如下:

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "length-filter"
}
```

#### Lowercase token filter（小写分词过滤器）

可以讲任何经过的分词变成小写

```
PUT /lowercase_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "standard_lowercase_example": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase"]
        },
        "greek_lowercase_example": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["greek_lowercase"]
        }
      },
      "filter": {
        "greek_lowercase": {
          "type": "lowercase",
          "language": "greek"
        }
      }
    }
  }
}
```

#### Uppercase Token Filter（大写分词过滤器）

讲所有经过的分词转大写

#### NGram Token Filter（N元语法过滤器）

| Setting    | Description      |
| ---------- | ---------------- |
| `min_gram` | Defaults to `1`. |
| `max_gram` | Defaults to `2`. |

索引级别设置index.max_ngram_diff控制max_gram和min_gram之间的最大允许差异。

N元语法过滤器即：

控制被切分单词的数量

举例：

> 1. **1-grams**:一元语法过滤器
>
> 以：＂spaghetti＂为例：
>
> ｓ　ｐ　ａ　ｇ　ｈ　ｅ　ｔ　ｔ　ｉ
>
> 1. 二元语法过滤器
>
>    将会拆分为: sp，pa，ag，gh，he，et，tt，ti
>
> 2. 三元语法过滤器
>
>    将会拆分为：spa，pag，agh，ghe，het，ett，tti

#### Edge NGram Token Filter（侧边N元语法过滤器）

| Setting    | Description                                                |
| ---------- | ---------------------------------------------------------- |
| `min_gram` | Defaults to `1`.                                           |
| `max_gram` | Defaults to `2`.                                           |
| `side`     | deprecated. Either `front` or `back`. Defaults to `front`. |

为N元语法的变体：

仅仅从边缘构建N元语法

如:min = 2 ,max = 6

"spaghetti" 将会切分为： sp spa spag spagh spaghe

### 7.4.4 N元语法的使用场景以及使用案例

N元语法过滤器主要是可以用于处理因为的错误单词拼写，将单词进行切分组合，匹配相同的部分，这能保证即使错误的书写依然可以进行正确的匹配，但是中文的拼写修改有待进一步学习

当不知道是什么语言的时候，N元语法是一种很好的文本分析方式：

可以分析单词之间没有空格的单词！！！

指定依然靠min和max这两个重要参数

#### 使用案例:

```
PUT /n-gram-filter
{
  "settings": {
    "analysis": {
      "filter": {
        "myfilter":{
          "type": "ngram",
          "min_gram": 2,
          "max_gram": 6
        }
      }, 
      "analyzer": {
        "defulat": {
          "tokenizer": "standard",
          "filter": ["standard", "myfilter"]
        }
      }
    }
  }
}
```

结果：会出现一个报错提示官方已经弃用这两个参数

```
Deprecation: Deprecated big difference between max_gram and min_gram in NGram Tokenizer,expected difference must be less than or equal to: [1]
```

有关该问题的社区讨论

https://discuss.elastic.co/t/deprecation-deprecated-big-difference-between-max-gram-and-min-gram-in-ngram/122969

#### [Shingle Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/analysis-shingle-tokenfilter.html)（滑动窗口分词过滤器）

## 7.5 提取词干

### 7.5.1 算分提取词干

es提供三种算法词干：

1. snowball 过滤器
2. porter stem 过滤器
3. kstem 过滤器

#### 选取kstem 作为案例测试

### 7.5.2 使用字典提取词干

```
DELETE /kstem-filter

PUT /kstem-filter
{
  "settings": {
    "analysis": {
      "filter": {
        "myfilter":{
          "type": "kstem"
        }
      }, 
      "analyzer": {
        "defulat": {
          "tokenizer": "standard",
          "filter": ["lowercase", "myfilter"]
        }
      }
    }
  }
}

GET kstem-filter/_settings
```

# 8. 使用相关性进行搜索

## 8.1 es 的打分机制

### 8.1.1 文档是如何运作的：

默认使用的是**词频-逆文档**（TF-IDF）词频

TF：词频

IDF：逆文档频率

### 8.1.2 词频

词条在文本当中出现的次数

### 8.1.3 逆文档词频

如果一个分词在索引的不同文档中出现的次数越多， 就越**不重要**

> - 文档频率的逆源自得分乘以 1/DF , DF是文档频率。意味着越高频率得分就会越低
> - 逆文档频率只关心“是否”

### 8.1.4 Lucene 评分公式

- 词频
- 逆文档频率
- 调和因子
  - 搜索过多少文档以及发现多少词条
- 查询标准化
  - 试图让不同的查询结果具有可比性

## 8.2 其他打分方式

- Okapi BM25
- 随机性分歧，DFR相似度
- 基于信息，IB相似度
- LM Dirichlet 相似度
- LM Jelinek Mercer 相似度

### 如何指定索引的打分方式？

- 修改字段映射的similarity 参数

  ```
  POST /take_score
  {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    }, 
    "mappings": {
      "mytype":{
        "properties": {
          "title":{
            "type": "text",
            "similarity": "defualt"
          }
        }
      }
    }
  }
  
  或者
  
  PUT /like
  {
    "mappings": {
      "get-together":{
        "properties": {
          "tilte": {
            "type": "text",
            "similarity": "BM25"
          }
        }
      }
    }
  }
  ```

- 在settings 中扩展，为某个索引设置打分算法

  ```
  PUT /dislike
  {
  "settings": {
      "similarity": {
        "my_custom": {
          "type": "BM25",
          "k1": 1.2,
          "b": 0.75,
          "discount_overlaps": false
        }
      }
    }
  }
  GET /dislike/_settings
  ```

  > 重要改变：6.0以后官方已不提倡使用TF-IDF，因为该评分方式已经不适用与当今时代的实际需要，官方推荐使用BM25，并且建议7.0 以后的版本移除TF-IDF算法的所有内容，所以TF-IDF这种打分机制只做了解即可

## 8.3 索引期间Boosting

不推荐在创建索引的时候使用Boosting, 而是在查询的时候,原因如下

- 缺少灵活性
- 以低精度的数值在Lucene 的内部索引结构当中，只有一个字节存储，可能丢失精度
- boost 是运用于一个词条，配上多个词条，就意味着多次的boost， 进一步增加权重

```
PUT /my-gettogether
{
  "mappings": {
    "groups": {
      "properties": {
        "name":{
          "type": "text",
          "boost": 2.0
        }
      }
    }
  }
}
GET /my-gettogether/_mapping
```

## 8.4 查询期间的Boosting

- match、multi_match、simple_query_string、query_string
- funcation_score (更为精确)

使用match 查询进行boosting

```
POST /lib/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "about": {
              "query": "i hide",
              "boost": 2.5
            }
          }
        },
        {
          "match": {
            "last_name": {
              "query": "Simith"
            }
          }
        }
      ]
    }
  }
}
```

第一个match 将会比第二个match 查询有更大的影响力

### 8.4.1 跨越多个字段的查询

### 8.4.2 使用multi_match 查询，指定整个Boost

```
POST /lib/_search
{
  "query": {
    "multi_match": {
      "query": "smit wang",
      "fields": ["first_name", "last_name"],
      "boost": 2.5
    }
  }
}
```

### 8.4.3 使用特殊符号^对于name 进行boost

使用3倍boost 修改某个字段查询的Boost

```
POST /myindex/_search
{
  "query": {
    "multi_match": {
      "query": "long",
      "fields": ["name^3"]
    }
  }
}
```

### 8.4.4 query_string 中对于针对单个的词条进行boost

```
POST myindex/_search
{
  "query": {
    "query_string": {
      "query": "lixiao* OR *long^3"
    }
  }
}
```

一个字段被boost 4 倍，并不是意味着得分会乘以4，所以得分不是按照严格的乘法

## 8.5 使用"解释(explain)"来理解文档评分

- 请求体 expliane = true 告诉es 运行解释操作
- Url指定_explain

下面为请求体重设置

```
POST myindex/_search
{
  "query": {
    "query_string": {
      "query": "lixiao* OR *long^3"
    }
  },
  "explain": true
}
```

结果，内容较为庞大，不过应该不是很难懂

```
{
  "took": 58,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 4,
    "hits": [
      {
        "_shard": "[myindex][2]",
        "_node": "Q58S9K26RxqIiVA-UQNrgg",
        "_index": "myindex",
        "_type": "test",
        "_id": "Fbggy20B5sayAd6PVrE2",
        "_score": 4,
        "_source": {
          "name": "lixiaolong",
          "age": 5,
          "birthday": "2019-05-04",
          "test_field": "zzzzzz"
        },
        "_explanation": {
          "value": 4, # 最后得分
          "description": "sum of:",
          "details": [
            {
              "value": 1,
              "description": "max of:",
              "details": [
                {
                  "value": 1,
                  "description": "name:lixiao*",
                  "details": []
                },
                {
                  "value": 1,
                  "description": "name.keyword:lixiao*",
                  "details": []
                }
              ]
            },
            {
              "value": 3,
              "description": "max of:",
              "details": [
                {
                  "value": 3,
                  "description": "name:*long^3.0",
                  "details": []
                },
                {
                  "value": 3,
                  "description": "name.keyword:*long^3.0",
                  "details": []
                }
              ]
            }
          ]
        }
      },
      {
        "_shard": "[myindex][3]",
        "_node": "Q58S9K26RxqIiVA-UQNrgg",
        "_index": "myindex",
        "_type": "test",
        "_id": "1",
        "_score": 4,
        "_source": {
          "name": "lixiaolong",
          "age": 55,
          "birthday": "2019-05-04"
        },
        "_explanation": {
          "value": 4,
          "description": "sum of:",
          "details": [
            {
              "value": 1,
              "description": "max of:",
              "details": [
                {
                  "value": 1,
                  "description": "name:lixiao*",
                  "details": []
                },
                {
                  "value": 1,
                  "description": "name.keyword:lixiao*",
                  "details": []
                }
              ]
            },
            {
              "value": 3,
              "description": "max of:",
              "details": [
                {
                  "value": 3,
                  "description": "name:*long^3.0",
                  "details": []
                },
                {
                  "value": 3,
                  "description": "name.keyword:*long^3.0",
                  "details": []
                }
              ]
            }
          ]
        }
      },
      {
        "_shard": "[myindex][3]",
        "_node": "Q58S9K26RxqIiVA-UQNrgg",
        "_index": "myindex",
        "_type": "test",
        "_id": "FLjrym0B5sayAd6PrrGN",
        "_score": 3,
        "_source": {
          "name": "CHENLONG",
          "age": 15,
          "birthday": "2011-02-14"
        },
        "_explanation": {
          "value": 3,
          "description": "sum of:",
          "details": [
            {
              "value": 3,
              "description": "max of:",
              "details": [
                {
                  "value": 3,
                  "description": "name:*long^3.0",
                  "details": []
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

> 从改结果中我们其实可以了解到：
>
> es 官方的分析内容表示官方其实已经将原有的TF-IDF(词频-逆文档频率)进行移除
>
> 现在默认使用的是BM25
>
> 并无法进行yml配置操作

### 8.5.1 解释一篇文档不匹配的原因

```
POST /myindex/test/5/_explain
{
  "query":{
    "match": {
      "name": "xiaolong"
    }
  }
}
```

## 8.6 使用查询再打分来减小评分操作影响

### 8.6.1 官方文档介绍 6.2版本

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-request-rescore.html

- 使用脚本评分
- 进行phrase 查询，搜索一段距离内的单词，使用很大的slop值

使用es的再打分特性，计算返回结果集合的第二轮计算得分

### 8.6.2 使用rescore 特性，对于匹配文档子集再评分

```
POST /myindex/_search
{
  "query": {
    "query_string": {
      "query": "lixiao* OR *long^3"
    }
    
  },
  "rescore": {
    "window_size": 20,
    "query": {
      "rescore_query": {
        "match_phrase":{
          "name": {
            "query": "li",
            "slop": 5
          }
        }
      },
      "query_weight": 0.8,
      "rescore_query_weight": 1.3
    }
  }
  
}
```

query_weight：初始查询得分权重

rescore_query_weight：再评分查询得分权重

## 8.7 function_score 来定制得分

### 8.7.1 function_score 的基本结构

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "name": "lixiaolong"
        }
      },
      "functions": [
        
      ]
    }
  }
}
```

### 8.7.2 weight 函数

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_phrase": {
          "name":{
            "query": "lixiaolong",
            "slop": 5
          }
        }
      },
      "functions": [
        {
          "weight": 3
          "filter": {
            "term": {
              "age": 5
            }
          }
        }
      ]
    }
  }
}
```

大致意识是将过滤出age =5 的文档将权重加3倍

#### 可以指定多个weight函数

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_phrase": {
          "name":{
            "query": "lixiaolong",
            "slop": 5
          }
        }
      },
      "functions": [
        {
          "weight": 1.5,
          "filter": {
            "term": {
              "age": 5
            }
          }
        },
        {
          "weight": 1.8,
          "filter": {
            "term": {
              "test_field": "zzzzzz"
            }
          }
        }
      ]
    }
  }
}
```

### 8.7.3 合并得分

得分合并的两种因素：

- 从每个单独的函数而来的分数如何合并，score_mode
- 从函数而来的得分如何同原始查询得分合并，boost_mode

第一个因素称为score_mode 参数， 处理不同函数得分如何合并

可以设置的参数如下

- multiply
- sum
- avg
- first
- max
- min

默认是得分相乘 multiply

如果设置为first,则会优先选择第一个匹配的Boost因子

第二种得分合并设置，称为boost_mode。控制原始查询的得分和函数得分如何合并

可以设置的参数如下:

- sum
- avg
- max
- min
- replayce

默认为初始查询得分和函数得分相乘

设置为replace 则原有的查询被函数得分替换

### 8.7.4 field_value_factor 函数

该函数将包含数值的字段的名称作为输入，选择性的乘以常熟，然后进行数学运算

### 8.7.5 在function_score 查询中使用field_value_factor

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "name": "lixiaolong"
        }
      },
      "functions": [
        {
          "field_value_factor": {
            "field": "age",
            "factor": 1.2, 
            "modifier": "ln"
          }
        }
      ]
    }
  }
}
```

modifier 可选参数：

- none (默认)
- log
- log1p
- log2p
- ln1p
- ln2p
- square
- sqrt
- reciprocal
- ln

## 8.8 painless脚本(5.0后go语言脚本支持已删除)

使用一个脚本进行评分

5.0 之后已经将go语言等脚本移除，使用Lucene官方指定的脚本语言 painless 类js语言

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "term": {
          "name": "lixiaolong"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": {
              "lang": "painless", 
              "params": {
                "mytest":555
              }, 
              "source": """
                int total = 0;
                for (int i = 0; i < doc['age'].length; ++i) {
                  total += doc['age'][i] + params.mytest;
                }
                return total;
              """
              
            }
          }
        }
      ]
    }
  }
}
```

如果要引用params的值，需要使用params.xxx进行脚本的常量引用

### 8.7.5 random_score 函数

```
POST /myindex/_search
{
  "query": {
    "function_score": {
      "query": {
        "term": {
          "name": "lixiaolong"
        }
      },
      "functions": [
        {
          "random_score": {
            "seed": 1234,
            "field": "age"
          }
        }
      ]
    }
  }
}
```

> It was possible to set a seed without setting a field, but this has been deprecated as this requires loading fielddata on the `_id` field which consumes a lot of memory.

注意需要指定field 对于文档字段的排序

否则会报一个必要参数提示

### 8.7.6 衰减函数

- linear
- gauss
- exp

#### 官方文档

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-function-score-query.html#function-decay

官方配置案例如下:

```
GET /_search
{
    "query": {
        "function_score": {
            "gauss": {
                "date": {
                      "origin": "2013-09-17", 
                      "scale": "10d",
                      "offset": "5d", 
                      "decay" : 0.5 
                }
            }
        }
    }
}
```

#### 配置选项

- origin：曲线原点，表示期望值。地理距离，日期或者数值型
- offset：分数开始衰减的位置
- scale 和 decay ：字段值为scale的时候，分数减少到指定的decay

根据上面分析，从5天开始，到第10天的时候，衰减到原分数的0.5

## 8.9 使用脚本排序

# 9. Elasticsearch REST API 的学习使用

## 9.0 API官网

rest 低级 api：https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/6.2.4/org/elasticsearch/client/package-summary.html

rest 高级 api：https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/6.2.4/index.html

## 9.1 REST 低级 Api

低级客户端的功能包括：

- 最小依赖
- 跨所有可用节点进行负载平衡
- 发生节点故障并根据特定响应代码进行故障转移
- 失败的连接惩罚（是否重试失败的节点取决于失败的连续次数；失败尝试次数越多，客户端在再次尝试该相同节点之前将等待的时间越长）
- 持续的联系
- 跟踪记录请求和响应
- 可选的自动[发现群集节点](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/6.2/sniffer.html)

### 9.1.1 起步

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>6.2.4</version>
</dependency>
```

### 9.1.2 初始化

```
可以通过相应的RestClientBuilder类（通过RestClient＃builder（HttpHost ...）静态方法创建）来构建RestClient实例。 唯一必需的参数是客户端将与之通信的一个或多个主机，以HttpHost的实例形式提供，如下所示：
public static void main(String[] args) {
        RestClientBuilder restClientBuilder = RestClient.builder(
                new HttpHost("192.168.92.180", 9200, "http"));
        // 设置最大超时时间
        restClientBuilder.setMaxRetryTimeoutMillis(10000);
        // 监听失效
        restClientBuilder.setFailureListener(new RestClient.FailureListener() {
            @Override
            public void onFailure(HttpHost host) {
                System.err.println("连接失效");
            }
        });
        // 设置连接的回调内容
        restClientBuilder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
            @Override
            public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder builder) {
                System.err.println("sss");
                System.err.println("sss");
                System.err.println("sssf");
                // socket套接字连接超时时间
                return builder.setSocketTimeout(10000);
            }
        });
        // 设置回调的代理客户端
        restClientBuilder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setProxy(new HttpHost("proxy", 9000, "http"));
            }
        });

        RestClient restClient = restClientBuilder.build();


    }
```

- RestClient 是线程安全的
- 可以再建造器里面设置连接最大超时时间，默认为30秒
- 可以设置代理客户端
- 可以设置socket套接字超时时间

### 9.1.3 执行请求

一旦`RestClient`被创建，请求可以通过调用可用的一个被发送`performRequest`或`performRequestAsync`方法的变体。这些`performRequest`方法是同步的，并且`Response`直接返回，这意味着客户端将阻塞并等待返回响应。该`performRequestAsync`变种返回`void`并接受额外的 `ResponseListener`作为参数来代替，这意味着它们是异步执行的。请求完成或失败时，将通知提供的侦听器。

```
        RestClientBuilder restClientBuilder = RestClient.builder(
                new HttpHost("192.168.92.180", 9200, "http"));
        RestClient restClient = restClientBuilder.build();
        // 1. 发送一个普通的get请求
//        Map<String, String> params = Collections.singletonMap("pretty", "true");
//        Response response = restClient.performRequest("GET", "/", params);

        // 2. 发送一个带参数的请求，并创建一个文档
        Map<Object, Object> singletonMap = Collections.emptyMap();
        // 请求url参数
        Map<String, String> params = Collections.singletonMap("pretty", "true");
        String jsonString = "{" +
                "\"user\":\"kimchy\"," +
                "\"postDate\":\"2013-01-30\"," +
                "\"message\":\"trying out Elasticsearch\"" +
                "}";
        // 构建http请求体
        HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
        // 创建一个文档
//        Response response = restClient.performRequest("PUT", "/posts/doc/1", params, entity);
        // 3. 创建一个_search 请求， 并且指定缓冲区的大小
        HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory heapBufferedResponseConsumerFactory
                = new HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory(30 * 1024 * 1024);
        // 创建一个响应监听器
        ResponseListener responseListener = new ResponseListener() {
            @Override
            public void onSuccess(Response response) {
                System.err.println("响应成功");
            }

            @Override
            public void onFailure(Exception e) {
                System.err.println("响应失败");
            }
        };
        restClient.performRequestAsync("GET", "/posts/_search", params, null, heapBufferedResponseConsumerFactory, responseListener);
```

> 注意：为HttpEntity指定的ContentType很重要，因为它将用于设置Content-Type标头，以便Elasticsearch可以正确解析内容。

### 9.1.4 接受响应体的处理

```
Response response = restClient.performRequest("PUT", "/posts/doc/1", params, entity);
RequestLine requestLine = response.getRequestLine();
HttpHost host = response.getHost();
int statusCode = response.getStatusLine().getStatusCode();
Header[] headers = response.getHeaders();
String responseBody = EntityUtils.toString(response.getEntity());

System.err.println(requestLine.getUri());
System.err.println(responseBody);
```

> 对于返回404状态代码的HEAD请求，不会引发ResponseException，因为这是预期的HEAD响应，它仅表示未找到资源。 除非ignore参数包含404，否则所有其他HTTP方法（例如GET）都将引发404响应的ResponseException。ignore是一个特殊的客户端参数，不会发送到Elasticsearch，并且包含逗号分隔的错误状态代码列表。 它允许控制是否将某些错误状态代码视为预期的响应而不是异常。 例如，这对于使用get api很有用，因为当缺少文档时它可以返回404，在这种情况下，响应主体将不包含错误，而是通常的get api响应，只是没有找到未找到的文档。

## 9.2 REST 高级 API

### 9.2.1 机器翻译介绍

Java高级REST客户端在Java高级REST客户端之上工作。它的主要目的是公开API特定的方法，这些方法接受请求对象作为参数并返回响应对象，以便请求编组和响应解编组由客户端本身处理。

每个API可以同步或异步调用。同步方法返回一个响应对象，而名称以`async`后缀结尾的异步方法则需要一个侦听器参数，一旦收到响应或错误，该参数将被通知（在低级客户端管理的线程池上）。

Java高级REST客户端取决于Elasticsearch核心项目。它接受与相同的请求参数，`TransportClient`并返回相同的响应对象。

### 9.2.2 [兼容性](https://github.com/elastic/elasticsearch/edit/6.2/docs/java-rest/high-level/getting-started.asciidoc)

Java高级REST客户端需要Java 1.8，并依赖于Elasticsearch核心项目。客户端版本与为其开发客户端的Elasticsearch版本相同。它接受与相同的请求参数，`TransportClient` 并返回相同的响应对象。 如果需要将应用程序从其迁移到新的REST客户端，请参阅《[*迁移指南》*](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/6.2/java-rest-high-level-migration.html)`TransportClient`。

确保高级客户端能够与在相同主要版本和较大或相等的次要版本上运行的任何Elasticsearch节点进行通信。它不需要与与其通信的Elasticsearch节点处于相同的次要版本，因为它是前向兼容的，这意味着它支持与比其开发的版本更高的Elasticsearch通信。

6.0客户端可以与任何6.x Elasticsearch节点进行通信，而6.1客户端可以与6.1、6.2和任何更高版本的6.x版本进行通信，但是与先前的Elasticsearch节点进行通信时可能会出现不兼容问题如果6.1客户端支持6.0节点不知道的某些API的新请求正文字段，则版本介于6.1和6.0之间。

建议将Elasticsearch群集升级到新的主要版本时升级High Level Client，因为REST API的重大更改可能会导致意外结果，具体取决于请求所命中的节点，并且新添加的API仅受支持。客户端的较新版本。一旦集群中的所有节点都已升级到新的主要版本，客户端应始终最后更新。

### 9.2.3 起步

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>6.2.4</version>
</dependency>
```

- org.elasticsearch.client:elasticsearch-rest-client
- org.elasticsearch:elasticsearch

### 9.2.4 创建一个rest 连接

### 9.2.5 【6.2.4】版本 支持的相关API （重点）

#### 创建索引

```
/**
     * 创建索引
     */
    private static void createIndex(RestHighLevelClient client) throws Exception {
        // 1. 创建索引请求，并且指定索引名称
        CreateIndexRequest indexRequest = new CreateIndexRequest("twitter");
        // 2. 设置索引的 settings 注意和es原生方式不同需要加index.
        indexRequest.settings(Settings.builder()
            .put("index.number_of_shards", 3)
            .put("index.number_of_shards", 2));
        // 3. 设置mappings 映射。由于5.0升级需要加上 header请求类型
        indexRequest.mapping("tweet",
                "  {\n" +
                        "    \"tweet\": {\n" +
                        "      \"properties\": {\n" +
                        "        \"message\": {\n" +
                        "          \"type\": \"text\"\n" +
                        "        }\n" +
                        "      }\n" +
                        "    }\n" +
                        "  }",
                XContentType.JSON);
        // 4. 指定索引的别名
        indexRequest.alias(
                new Alias("twitter_alias")
        );
        // 5. 可选参数提供
        // TODO: 这两个参数作用目前不太明确
        indexRequest.timeout(TimeValue.timeValueMinutes(2));
        indexRequest.timeout("2m");

        // 5.1连接到主节点的超时时间
        indexRequest.masterNodeTimeout(TimeValue.timeValueMinutes(1));
        indexRequest.masterNodeTimeout("1m");

        // 5.2 在创建索引API返回响应之前，要等待的活动碎片副本数
        indexRequest.waitForActiveShards(2);
        indexRequest.waitForActiveShards(ActiveShardCount.DEFAULT);

        // 6. 同步执行的方式
        CreateIndexResponse createIndexResponse = client.indices().create(indexRequest);

        // 7. 异步执行方式
        // 由于异步方式不能立刻返回结果，所以需要监听器去监听异步请求结果
//        ActionListener<CreateIndexResponse> listener = new ActionListener<CreateIndexResponse>() {
//            @Override
//            public void onResponse(CreateIndexResponse createIndexResponse) {
//                System.err.println("返回结果");
//            }
//
//            @Override
//            public void onFailure(Exception e) {
//                System.err.println("出现异常");
//
//            }
//        };
//        client.indices().createAsync(indexRequest, listener);

        // 8. 处理返回结果
        boolean acknowledged = createIndexResponse.isAcknowledged();
        boolean shardsAcknowledged = createIndexResponse.isShardsAcknowledged();

    }
```

#### 删除索引

```
/**
     * 删除索引
     * @param client
     * @throws Exception
     */
    private static void deleteIndex(RestHighLevelClient client) throws Exception {
        // 创建删除索引的对象
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("twitter");
        // 2. TODO: 这两个参数作用目前不太明确
        deleteIndexRequest.timeout(TimeValue.timeValueMinutes(2));
        deleteIndexRequest.timeout("2m");

        // 2.1 连接到主节点的超时时间
        deleteIndexRequest.masterNodeTimeout(TimeValue.timeValueMinutes(1));
        deleteIndexRequest.masterNodeTimeout("1m");

        // 3. 设置IndicesOptions可以控制如何解决不可用的索引以及如何扩展通配符表达式
        // 3.1 IndicesOptions.lenientExpandOpen()
        // 索引选项将忽略不可用的索引，仅将通配符扩展为打开的索引，并不允许通配符表达式解析任何索引（不返回错误）
        deleteIndexRequest.indicesOptions(IndicesOptions.lenientExpandOpen());

        // 4.1 同步方式
        DeleteIndexResponse delete = client.indices().delete(deleteIndexRequest);

        // 4.2 异步方式
//        client.indices().deleteAsync(deleteIndexRequest, new ActionListener<DeleteIndexResponse>(){
//
//            @Override
//            public void onResponse(DeleteIndexResponse deleteIndexResponse) {
//                System.err.println("返回结果");
//            }
//
//            @Override
//            public void onFailure(Exception e) {
//                System.err.println("出现异常");
//            }
//        });

        // 5. 如果删除的时候没有索引会发生什么？
        // 请根据如下代码进行处理
//        try {
//            DeleteIndexRequest request = new DeleteIndexRequest("does_not_exist");
//            client.indices().delete(request);
//        } catch (ElasticsearchException exception) {
//            if (exception.status() == RestStatus.NOT_FOUND) {
//              //处理异常
//            }
//        }
    }
```

#### 开放索引

```
/**
     * 开放索引
     * @param client
     */
    public static void openIndex(RestHighLevelClient client) throws Exception{
        // 1. 创建一个开放索引的请求
        OpenIndexRequest openIndexRequest = new OpenIndexRequest("myindex");
        // 2. 等待所有节点确认索引的超时被打开
        openIndexRequest.timeout(TimeValue.timeValueMinutes(2));
        // 3. 设置请求开放索引处理策略
        // 要求每个指定索引都存在的索引选项，仅将通配符扩展为打开的索引，并且不允许通配符表达式解析任何索引（不返回错误）。
        openIndexRequest.indicesOptions(IndicesOptions.strictExpandOpen());
        // 4. 设置开放索引API返回响应之前要等待的活动分片副本数
        openIndexRequest.waitForActiveShards(2);
        // 4.1 ActiveShardCount 存在多个选下那个
        openIndexRequest.waitForActiveShards(ActiveShardCount.ONE);
        // 5. 同步方式
        OpenIndexResponse openIndexResponse = client.indices().open(openIndexRequest);

        // 6. 异步方式
        // 由于异步方式不能立刻返回结果，所以需要监听器去监听异步请求结果
//        ActionListener<OpenIndexResponse> listener = new ActionListener<OpenIndexResponse>() {
//            @Override
//            public void onResponse(CreateIndexResponse createIndexResponse) {
//                System.err.println("返回结果");
//            }
//
//            @Override
//            public void onFailure(Exception e) {
//                System.err.println("出现异常");
//
//            }
//        };
//        client.indices().openAsync(openIndexRequest, listener);

        // 7. 分片是否正常响应
        openIndexResponse.isShardsAcknowledged();
    }
```

#### 关闭索引

```
  /**
     * 关闭索引
     * @param client
     */
    private static void closeIndex(RestHighLevelClient client) throws Exception{
        // 1. 创建一个关闭索引的对象
        CloseIndexRequest closeIndexRequest = new CloseIndexRequest("myindex");
        // 2. 基本设置： 大同小异，不多啰嗦了
        closeIndexRequest.timeout(TimeValue.timeValueMinutes(2));
        closeIndexRequest.timeout("2m");
        closeIndexRequest.masterNodeTimeout(TimeValue.timeValueMinutes(1));
        closeIndexRequest.masterNodeTimeout("1m");
        closeIndexRequest.indicesOptions(IndicesOptions.lenientExpandOpen());
        // 3. 同步方式
        CloseIndexResponse closeIndexResponse = client.indices().close(closeIndexRequest);
        // 4. 异步方式
        client.indices().closeAsync(closeIndexRequest, new ActionListener<CloseIndexResponse>() {
            @Override
            public void onResponse(CloseIndexResponse closeIndexResponse) {
                // ....
            }

            @Override
            public void onFailure(Exception e) {
                // ....
            }
        });

        boolean acknowledged = closeIndexResponse.isAcknowledged();
    }
```

### 9.2.6 索引本身的API

#### 直接根据索引创建一个文档需要如下参数

1. 索引名称
2. 索引类型
3. 文档id
4. 文档数据：提供一个json数据

#### 提供文档来源的多种方式

可以使用不同的方式来创建文档源数据

```
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postData", new Date());
jsonMap.put("message", "trying out Elasticsearch");
// 5. 使用map创建
indexRequest.source(indexRequest, jsonMap);

// 6. es 官方开发了 XContentBuilder 来构建_source 的请求json
XContentBuilder xContentBuilder = XContentFactory.jsonBuilder();
xContentBuilder.startObject();
{
    xContentBuilder.field("user", "kimchy");
    xContentBuilder.field("postDate", new Date());
    xContentBuilder.field("message", "trying out Elasticsearch");
}
xContentBuilder.endObject();
IndexRequest source = indexRequest.source(xContentBuilder);

// 7. 也可以在_soucre 的时候直接传入键值对数据
indexRequest.source("user", "kimchy",
                    "postDate", new Date(),
                    "message", "trying out Elasticsearch");
```

#### 可选参数

设置路由

设置parent 值

设置分片处理的超时时间

将策略刷新为WriteRequest.RefreshPolicy实例

设置初始版本号

设置版本类型?

提供类型形式修改操作类型，以及以字符串形式提供的操作类型：可以创建或更新（默认）

索引文档之前要执行的管道流的名称

```
// 8. 可选参数
// 8.1 设置路由
indexRequest.routing("routing");
// 8.2 设置父索引
indexRequest.parent("parent");
// 8.3 设置分片处理的超时时间
indexRequest.timeout(TimeValue.timeValueSeconds(1));
indexRequest.timeout("1s");
// 8.4 设置刷新策略
indexRequest.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
indexRequest.setRefreshPolicy("wait_for");
// 8.5 设置版本号
indexRequest.version(2);
// 8.6 设置版本类型
indexRequest.versionType(VersionType.EXTERNAL);
// 8.7 提供类型形式修改操作类型，以及以字符串形式提供的操作类型：可以创建或更新（默认）
indexRequest.opType(DocWriteRequest.OpType.CREATE);
indexRequest.opType("create");
// 8.8 索引文档之前要执行的管道流的名称
indexRequest.setPipeline("pipeline");
```

#### 获取响应内容

```
// 11. 处理响应体内容
indexResponse.getIndex();
indexResponse.getId();
indexResponse.getType();
indexResponse.getVersion();
if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
    // 如果是创建

} else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {
    // 如果是修改
}
ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    // 如果存在失败分片（即返回部分信息）如何处理
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        // 获取失败原因
        String reason = failure.reason();
    }
}
```

#### 根据上述内容整合

```
/**
     * 根据索引创建一个索引文档
     * @param client
     * @throws Exception
     */
    private static void createIndexDocumentByIndex(RestHighLevelClient client) throws Exception{
        // 1. 创建 索引请求
        IndexRequest indexRequest = new IndexRequest(
                "posts",
                "doc",
                "1");
        // 2. _source 元数据
        String jsonString = "{" +
                "\"user\":\"kimchy\"," +
                "\"postDate\":\"2013-01-30\"," +
                "\"message\":\"trying out Elasticsearch\"" +
                "}";
        // 3. 设置元数据
        indexRequest.source(indexRequest, jsonString);

        // 4. 提供另外一种方式
        Map<String, Object> jsonMap = new HashMap<>();
        jsonMap.put("user", "kimchy");
        jsonMap.put("postData", new Date());
        jsonMap.put("message", "trying out Elasticsearch");
        // 5. 使用map创建
        indexRequest.source(indexRequest, jsonMap);

        // 6. es 官方开发了 XContentBuilder 来构建_source 的请求json
        XContentBuilder xContentBuilder = XContentFactory.jsonBuilder();
        xContentBuilder.startObject();
        {
            xContentBuilder.field("user", "kimchy");
            xContentBuilder.field("postDate", new Date());
            xContentBuilder.field("message", "trying out Elasticsearch");
        }
        xContentBuilder.endObject();
        IndexRequest source = indexRequest.source(xContentBuilder);

        // 7. 也可以在_soucre 的时候直接传入键值对数据
        indexRequest.source("user", "kimchy",
                        "postDate", new Date(),
                        "message", "trying out Elasticsearch");

        // 8. 可选参数
        // 8.1 设置路由
        indexRequest.routing("routing");
        // 8.2 设置父索引
        indexRequest.parent("parent");
        // 8.3 设置分片处理的超时时间
        indexRequest.timeout(TimeValue.timeValueSeconds(1));
        indexRequest.timeout("1s");
        // 8.4 设置刷新策略
        indexRequest.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
        indexRequest.setRefreshPolicy("wait_for");
        // 8.5 设置版本号
        indexRequest.version(2);
        // 8.6 设置版本类型
        indexRequest.versionType(VersionType.EXTERNAL);
        // 8.7 提供类型形式修改操作类型，以及以字符串形式提供的操作类型：可以创建或更新（默认）
        indexRequest.opType(DocWriteRequest.OpType.CREATE);
        indexRequest.opType("create");
        // 8.8 索引文档之前要执行的管道流的名称
        indexRequest.setPipeline("pipeline");

        // 9. 同步方式
        IndexResponse indexResponse = client.index(indexRequest);
        // 10. 异步方式
        client.indexAsync(indexRequest, new ActionListener<IndexResponse>() {
            @Override
            public void onResponse(IndexResponse indexResponse) {
                // ...
            }

            @Override
            public void onFailure(Exception e) {
                // ....
            }
        });

        // 11. 处理响应体内容
        indexResponse.getIndex();
        indexResponse.getId();
        indexResponse.getType();
        indexResponse.getVersion();
        if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
            // 如果是创建

        } else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {
            // 如果是修改
        }
        ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
        if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
            // 如果存在失败分片（即返回部分信息）如何处理
        }
        if (shardInfo.getFailed() > 0) {
            for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
                // 获取失败原因
                String reason = failure.reason();
            }
        }
    }
```

# 10. Elasticsearch JAVA API 的学习使用

## 10.1 起步：

### 10.1.1 Maven 依赖

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>transport</artifactId>
    <version>6.2.4</version>
</dependency>
```

# 11. Es painless 脚本插件使用（Elasticsearch script plugin）

如果需要使用原生java语言实现自定义评分，需要实现官方提供的一个scriptEngine接口，官方在git上面给出了一个案例

## 11.1 前置条件：windows安装gradle

官方推荐用gradle 来构建插件的Jar包，为了怕出现麻烦，直接按照官方给的要求来，这里先列出一下gradle的基本安装步骤

### 11.1.1 官方下载地址

https://services.gradle.org/distributions/

下载 **gradle-5.6.3-all.zip** 包，注意别下错了

### 11.1.2 将gradle放入合适位置，将gradle加入path环境变量

### 11.1.3 配置阿里云的镜像

# N kibana 学习es的所有内容

## 2019.10.14

```
GET _search
{
  "query": {
    "match_all": {}
  }
}

PUT /lib/

{

  "settings":{

  "index":{
  
    "number_of_shards": 5,
    
    "number_of_replicas": 1
    
    }
    
  }

}


GET _all/_settings

POST /lib/user/
{
    "first_name" :  "wang",
    "last_name" :   "laji",
    "age" :         333,
    "about" :       "I hide to collect rock albums",
    "interests":  [ "basket" ]
}


GET /+get-toge*,-get-together/_search

GET /lib/user/_search?from=2&size=2

GET /lib/_search?sort=age:_id&_source=first_name,age

GET /lib/user/_search?q=first_name:ne

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 2
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["*_name", "a*"]
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": {
    "includes": ["*_name", "age"],
    "excludes": ["first_name"]
  }
}
 
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
 
    "_score"
  ]
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2,
  "_source": ["*_name", "age"],
  "sort": [
    {
      "age":"desc"  
    }
    
  ]
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": "Jane"
    }
  }
}

GET /lib/user/_search
{
  
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "about": "hide"
          }
        }
      ], 
      "filter": {
        "term": {
          "first_name": "wang"
        }
      }
    }
  }
}

GET /lib/_search?q=wang

GET /lib/_search
{
  "query": {
    "query_string": {
      "default_field": "first_name",
      "query": "first_name:wang OR first_name:jane AND age:[0 TO  999]"
    }
  }
}

GET /lib/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "_id": 1
        } 
        
      }
    }
  }
}

GET /lib/_search
{
  "query": {
    "terms": {
      "about": [
        "i",
        "hide"
      ]
    }
    
  }
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "query": "Jane wang",
        "operator": "or"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "type": "phrase_prefix",
        "query": "hide"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match_phrase": {
      "about": {
        "query": "to rock"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match_phrase_prefix": {
      "first_name": {
        "query": "wa",
        "max_expansions": 5
      }
    }
  }
}

POST /lib/user/_search
{
  "query": {
    "multi_match": {
        "query": "wang smith",
        "fields": ["first_name", "last_name"]
    }
  }
}

POST /lib/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase":  {
            "about": {
              "query": "i to",
              "slop": 1
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "first_name": {
              "value": "wang"
            }
          }
        }
      ],
      "must_not": [
        {
          "query_string": {
            "default_field": "age",
            "query": "first_name:jane"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}

POST /myindex/test/
{
  "name": "lixiaolong",
  "age": 5,
  "birthday": "2019-05-04",
  "test_field": "zzzzzz"
}

GET /myindex/test/_search
{
  "query": {
    "range": {
      "name": {
        "gte": "li",
        "lte": "o"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "Li"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "lix?ao*"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "bool": {
      "filter": {
        "exists": {
          "field": "name"
        }
      }
    }
  }
}

GET /myindex/_search
{
  "query": {
    "bool": {
      "filter": {
        "missing": {
          "field": "test_field",
          "existence": true,
          "null_value": true
        }
      }
    }
  }
}

GET /myindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "query_string": {
            "default_field": "name",
            "query": "che*"
          }
        }
      ]
    }
  }
}


POST /mymyindex/
```

## 2019.10.15

```
GET _search
{
  "query": {
    "match_all": {}
  }
}

PUT /lib/

{

  "settings":{

  "index":{
  
    "number_of_shards": 5,
    
    "number_of_replicas": 1
    
    }
    
  }

}


GET _all/_settings

POST /lib/user/
{
    "first_name" :  "wang",
    "last_name" :   "laji",
    "age" :         333,
    "about" :       "I hide to collect rock albums",
    "interests":  [ "basket" ]
}


GET /+get-toge*,-get-together/_search

GET /lib/user/_search?from=2&size=2

GET /lib/_search?sort=age:_id&_source=first_name,age

GET /lib/user/_search?q=first_name:ne

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 2
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["*_name", "a*"]
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": {
    "includes": ["*_name", "age"],
    "excludes": ["first_name"]
  }
}
 
GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
 
    "_score"
  ]
}

GET /lib/user/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2,
  "_source": ["*_name", "age"],
  "sort": [
    {
      "age":"desc"  
    }
    
  ]
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": "Jane"
    }
  }
}

GET /lib/user/_search
{
  
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "about": "hide"
          }
        }
      ], 
      "filter": {
        "term": {
          "first_name": "wang"
        }
      }
    }
  }
}

GET /lib/_search?q=wang

GET /lib/_search
{
  "query": {
    "query_string": {
      "default_field": "first_name",
      "query": "first_name:wang OR first_name:jane AND age:[0 TO  999]"
    }
  }
}

GET /lib/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "_id": 1
        } 
        
      }
    }
  }
}

GET /lib/_search
{
  "query": {
    "terms": {
      "about": [
        "i",
        "hide"
      ]
    }
    
  }
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "query": "Jane wang",
        "operator": "or"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match": {
      "first_name": {
        "type": "phrase_prefix",
        "query": "hide"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match_phrase": {
      "about": {
        "query": "to rock"
      }
    }
  }
}

GET /lib/user/_search
{
  "query": {
    "match_phrase_prefix": {
      "first_name": {
        "query": "wa",
        "max_expansions": 5
      }
    }
  }
}

POST /lib/user/_search
{
  "query": {
    "multi_match": {
        "query": "wang smith",
        "fields": ["first_name", "last_name"]
    }
  }
}

POST /lib/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase":  {
            "about": {
              "query": "i to",
              "slop": 1
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "first_name": {
              "value": "wang"
            }
          }
        }
      ],
      "must_not": [
        {
          "query_string": {
            "default_field": "age",
            "query": "first_name:jane"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}

POST /myindex/test/
{
  "name": "lixiaolong",
  "age": 5,
  "birthday": "2019-05-04",
  "test_field": "zzzzzz"
}

GET /myindex/test/_search
{
  "query": {
    "range": {
      "name": {
        "gte": "li",
        "lte": "o"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "Li"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "lix?ao*"
      }
    }
  }
}

GET /myindex/test/_search
{
  "query": {
    "bool": {
      "filter": {
        "exists": {
          "field": "name"
        }
      }
    }
  }
}

GET /myindex/_search
{
  "query": {
    "bool": {
      "filter": {
        "missing": {
          "field": "test_field",
          "existence": true,
          "null_value": true
        }
      }
    }
  }
}

GET /myindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "query_string": {
            "default_field": "name",
            "query": "che*"
          }
        }
      ]
    }
  }
}

DELETE /mymyindex/ 

PUT /mymyindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": { 
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        "my_text": {
          "type": "text",
          "analyzer": "std_folded" 
        }
      }
    }
  }
}

GET /mymyindex/_mapping

POST /mymyindex/_analyze
{
  "tokenizer": "whitespace", 
  "filter": [
    "reverse"
  ],
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone.",
  "field": "my_text"
}

GET /get-together/group/3q8nz20BXNiyabSjyH-c/_termvectors


PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "& => and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
        } 
    }
}

 GET /my_index/_mapping
 GET /mymyindex/_mapping
 
POST /lib/_analyze
{
  "tokenizer": "standard",
  "text": "I have, photos"
}

POST /lib/_analyze
{
  "tokenizer": "letter",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone"
}


POST _analyze
{
  "tokenizer": "lowercase",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

POST _analyze
{
  "tokenizer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

POST _analyze
{
  "tokenizer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

POST _analyze
{
  "tokenizer": "uax_url_email",
  "text": "Email me at john.smith@global-international.com"
}

PUT /asciifold_example
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "default" : {
                    "tokenizer" : "standard",
                    "filter" : ["standard", "asciifolding"]
                }
            }
        }
    }
}

DELETE /kstem-filter

PUT /kstem-filter
{
  "settings": {
    "analysis": {
      "filter": {
        "myfilter":{
          "type": "kstem"
        }
      }, 
      "analyzer": {
        "defulat": {
          "tokenizer": "standard",
          "filter": ["lowercase", "myfilter"]
        }
      }
    }
  }
}
GET kstem-filter/_settings
POST /kstem-filter/_analyze
{
  "tokenizer": "standard", 
  "text": "Austrian"
}

GET /n-gram-filter/_analyze
{
  "text": "ilikebeijingjingbei"
}

POST _analyze
{
  "tokenizer": " synonyms",
  "text": "Email me at john.smith@global-international.com"
}
```