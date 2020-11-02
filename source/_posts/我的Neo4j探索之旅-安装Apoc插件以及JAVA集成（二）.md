---
title: 我的Neo4j探索之旅 - 安装Apoc插件以及JAVA集成（二）
subtitle: 对于neo4j的apoc插件进行二次扩展
url_suffix: note
author: lazytime
tags:
  - neo4j
categories:
  - 图数据库
keywords: neo4j,apoc
description: 使用apoc继承neo4j，同事根据此来进行下一步的处理操作
copyright: true
date: 2020-11-03 00:06:48
---
# 上一篇文章：

不知道为什么掘金我发不出文章，找不到那里违规了，上一篇文章我发布到思否了：

https://segmentfault.com/a/1190000037690548

<!-- more -->

# 如何安装neo4j - apoc 插件

有英语阅读能力建议参考官方文档：https://neo4j.com/developer/neo4j-apoc/

## 1. 下载neo4j - apoc 插件

进入github ： https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases

找到和当前neo4j 匹配的版本， 我选择[3.5.0.12](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/tag/3.5.0.12) 的版本

> 这里提供我的：
>
> 链接：https://pan.baidu.com/s/1Tb7f-bYjwVZhfQtGfMr2Gw 
> 提取码：bzwh

## 2. 具体的安装步骤

2. 下载好之后，放入到 `D:\zxd\tool\neo4j-community-3.5.12-windows\neo4j-community-3.5.12\plugins` 下面
3. 执行`neo4j stop`，关闭neo4j 服务
4. 进入 ~/conf 下面，找到`neo4j.conf` ~表示你的neo4j 安装位置
5. 修改`#dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.*` 在这一行的下面增加`dbms.security.procedures.unrestricted=apoc.*`的配置，安装apoc插件

> 下面的图看起来就像这样：
>
> \#dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.*
> dbms.security.procedures.unrestricted=apoc.*

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827153443.png)

6. 输入 `neo4j start` 启动neo4j 服务
7. 在可视化界面，输入`return apoc.version()` ,如果报错说明没安装对，显示如下页面，证明apoc 插件安装成功

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20200827154111.png)





# Neo4j 集成到java里面

## 1. 配置maven，加入依赖

```xml
<!-- neo4j 依赖包 -->
<dependency>
    <groupId>org.neo4j.driver</groupId>
    <artifactId>neo4j-java-driver</artifactId>
    <version>1.5.0</version>
</dependency>
```

## 2. 使用neo4j 集成java实战

这是之前实战的是用的方式，这里说下我大致的设计记录

需求：

1. neo4j 实现插拔式配置，没有配置的情况下进行连接不会影响程序运行
2. 所有的配置都需要放在`application_setting.xml`当中
3. 如果连接neo4j 失败，不做处理



#### 抽象接口

在`core` 包里面， 设计公用开放接口。

```java
	/**
     * 构建neo4j url地址
     * @return
     */
    String buildUrl();

    /**
     * 构建neo4j 用户名
     * @return
     */
    String buildUsername();

    /**
     * 构建neo4j 密码
     * @return
     */
    String buildPassword();

    /**
     * 构建neo4j 配置， 目前使用默认 配置
     * @return
     */
    Config buildConfig();

    /**
     * 是否开启自定义配置
     * @return
     */
    boolean enableCustomConfig();

    /**
     * 构建 Neo4j csv同步地址的前置
     * 默认为 file:///
     *
     * @return
     */
    String buildCvsPrefix();

    /**
     * 构建 neo4j csv同步的普通标签文件名
     * @return
     */
    String buildNormalTagSyncFileName();

    /**
     * 构建 neo4j csv同步的业务标签文件名
     * @return
     */
    String buildBusinessTagSyncFileName();

    /**
     * 构建 neo4j csv同步的用户标签文件名
     * @return
     */
    String buildUserTagSyncFileName();

    /**
     * 普通标签和业务标签的关联csv文件名称
     * @return
     */
    String buildNormalTagBizSyncFileName();

    /**
     * 普通标签和用户标签的关联csv文件名称
     * @return
     */
    String buildNormalTagUserSyncFileName();

    /**
     * 业务标签和用户标签的关联csv文件名称
     * @return
     */
    String buildBizTagUserSyncFileName();

    /**
     * 主题分类（业务）标签子父关联csv文件名称
     * @return
     */
    String buildBizChildSyncFileName();

    /**
     * 普通标签标签子父关联csv文件名称
     * TODO: 目前普通标签没有子父关联关系，后续需要拓展请开放此接口 by zhaoxudong
     * @return
     */
//    String buildNormalChildSyncFileName();

    /**
     * 用户标签的子父关联csv文件名称
     * @return
     */
    String buildUserChildSyncFileName();
```

#### 具体实现

maven的其他项目工程，只要实现了上面的接口，就可以根据自己的设定去设置如何读取配置，下面给出案例

```java
@Override
    public String buildUrl() {
        return Setter.getString("neo4j.address");
    }

    @Override
    public String buildUsername() {
        return Setter.getString("neo4j.username");
    }

    @Override
    public String buildPassword() {
        return Setter.getString("neo4j.password");
    }

    @Override
    public Config buildConfig() {
        return Config.defaultConfig();
    }

    @Override
    public boolean enableCustomConfig() {
        return Setter.getBoolean("neo4j.enablecustomconfig");
    }

    @Override
    public String buildCvsPrefix() {
        return Setter.getString("neo4j.datasyncprefix");
    }

    @Override
    public String buildNormalTagSyncFileName() {
        return Setter.getString("neo4j.normaltagsyncfilename");
    }

    @Override
    public String buildBusinessTagSyncFileName() {
        return Setter.getString("neo4j.businesstagsyncfilename");
    }

    @Override
    public String buildUserTagSyncFileName() {
        return Setter.getString("neo4j.usertagsyncfilename");
    }

    @Override
    public String buildNormalTagBizSyncFileName() {
        return Setter.getString("neo4j.normalbizsyncfilename");
    }

    @Override
    public String buildNormalTagUserSyncFileName() {
        return Setter.getString("neo4j.normalusersyncfilename");
    }

    @Override
    public String buildBizTagUserSyncFileName() {
        return Setter.getString("neo4j.bizusersyncfilename");
    }

    @Override
    public String buildBizChildSyncFileName() {
        return Setter.getString("neo4j.bizrelsyncfilename");
    }

    @Override
    public String buildUserChildSyncFileName() {
        return Setter.getString("neo4j.userrelsyncfilename");
    }
```

#### 结合到neo4j 连接

1. 根据上面两个步骤， ssm项目启动之后，会自动装载BEAN, 装载之后，只要在具体的程序里面，拿取配置即可（具体的配置获取实现可以由自己实现）
2. 根据上面的方法使用java拿到xml配置之后，就实现了下面的方式实现neo4j 的数据连接

```java
/**
     * 连接图数据库
     * @return
     */
private void createDriver(Neo4jConfigBuilder builder) {
    if(driver == null){
        try{
            driver =  GraphDatabase.driver(builder.buildUrl(), AuthTokens.basic(builder.buildUsername(), builder.buildPassword()));
        }catch(Exception e){
            driver = null;
            LOGGER.debug("无法建立图数据库连接，请检查图Neo4j 服务是否启动");
            throw new RuntimeException("无法建立图数据库连接，请检查图Neo4j 服务是否启动");
        }
    }
}
```

#### 具体开发

```java
 /**
     * 执行添加cql
     *
     * @param cql 查询语句
     */
    @Override
    public StatementResult run(Neo4jConfigBuilder builder,String cql) {
        createDriver(builder);
        Session session = driver.session();
        StatementResult run = null;
        try {
            run = session.run(cql);
        }catch (Exception e){
            e.printStackTrace();
        } finally {
            if(session!=null){
                session.close();
            }
        }
        return run;
    }

    @Override
    public StatementResult run(Neo4jConfigBuilder builder,String cql, Object... objects) {
        createDriver(builder);
        Session session = driver.session();
        StatementResult run = null;
        try {
            run = session.run(cql, Values.parameters(objects));
        }catch (Exception e){
            e.printStackTrace();
        } finally {
            if(session!=null){
                session.close();
            }
        }
        return run;
    }
```

## 曾经开发的时候查找的一些博客记录:

[NEO4J的安装配置及使用总结](https://blog.csdn.net/weixin_37197708/article/details/79453703)

[neo4j︱neo4j批量导入neo4j-import （五）](https://blog.csdn.net/sinat_26917383/article/details/82424508)

[neo4j - 查询效率的几种优化思路](https://blog.csdn.net/pan15125284/article/details/96144658)

[Neo4j如何对大量数据(千万节点及以上)进行初始化](https://www.jianshu.com/p/32ce953604d5)

[关于Neo4j和Cypher批量更新和批量插入优化的5个建议](https://blog.csdn.net/hwz2311245/article/details/60963383)

[Neo4j的查询速度为何这么慢？这能商用吗？](https://www.zhihu.com/question/45401120)

[Neo4j清空所有数据](https://www.cnblogs.com/jpfss/p/11578903.html)

[Neo4j安装APOC和图算法Neo.ClientError.Procedure.ProcedureRegistrationFailed？](https://cloud.tencent.com/developer/ask/129792)

[官方网站对于Apoc插件的介绍](https://neo4j.com/blog/intro-user-defined-procedures-apoc/)

neo4j cypher 语言的语法（非常非常重要）：

**[neo4j--Cypher语法练习(START、CREATE、MERGE)](https://blog.csdn.net/qq_37503890/article/details/101479680)**

[Neo4j中使用Cypher进行大批量节点删除的优化](https://www.jianshu.com/p/e0ffeaa59159)

[thinbug](https://www.thinbug.com/)  我的很多前端疑难杂症就是靠这网站解决的