---
title: elasticsearch Linux安装教程
subtitle: elasticsearch Linux安装教程
author: lazytime
url_suffix: note24
tags:
  - elastic
categories:
  - 中间件-elastic
keywords: elasticsearch,Linux安装教程
description: 安装教程
copyright: true
date: 2020-08-29 23:13:52
---

# elasticsearch Linux安装教程

<!-- more -->

## 基本步骤

1. 安装Jdk环境

   1. 省略

2. 下载官方的es包放在usr目录下

3. 在/usr目录下建立一个文件夹

   **注意这里不能使用root用户进行安装！！！**

   **注意这里不能使用root用户进行安装！！！**

   **注意这里不能使用root用户进行安装！！！**

   1. groupadd elsearch

   2. useradd elsearch -g elsearch

      //该命令是更改该文件夹下所属的用户组的权限

   3. chown -R elsearch:elsearch elasticsearch-5.6.3

4. 修改config 目录下的yml文件

   ```
   #配置es的集群名称，默认是elasticsearch，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。
   
   cluster.name: my-es
   
   \#节点名称
   
   node.name: node-1  
   
   \#设置索引数据的存储路径
   
   path.data: /usr/export/servers/data  
   
   \#设置日志的存储路径
   
   path.logs: /usr/export/servers/logs  
   
   \#设置当前的ip地址,通过指定相同网段的其他节点会加入该集群中
   
   network.host: 0.0.0.0
   
   \#设置对外服务的http端口
   
   http.port: 9200  
   
   \#设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
   
   discovery.zen.ping.unicast.hosts: ["node-1"]
   
   记得修改下列两条
   bootstrap.memory_lock: false
   bootstrap.system_call_filter: false
   ```

配置elasticsearch 用户以及权限

```
mkdir -p /usr/export/servers/data

mkdir -p /usr/export/servers/logs



chown -R es:es /usr/export/servers/elasticsearch

chown -R es:es /usr/export/servers/data

chown -R es:es /usr/export/servers/logs
```

1. 切换用户 su es

2. 集群启动 bin/elasic。

3. **报错！！**

   > 1.[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536] 意思是说你的进程不够用了
   >
   > 解决办法 切到root 用户：进入到security目录下的limits.conf；执行命令 在文件的末尾添加下面的参数值：

   ```
    vim /etc/security/limits.conf
   * soft nofile 65536
   * hard nofile 131072
   * soft nproc 2048
   * hard nproc 4096
   前面的*符号必须带上，然后重新启动就可以了。执行完成后可以使用命令 ulimit -n 查看进程数      
   ```

   > 2.[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144] 需要修改系统变量的最大值了
   >
   > 解决方案：切换到root用户修改配置sysctl.conf 增加配置值： vm.max_map_count=655360
   >
   > 执行命令 sysctl -p 这样就可以了，然后重新启动ES服务 就可以了

   > 3.[3] snamp centerOs6 不支持 snamp 需要在 yml 中修改配置
   >
   > 解决方案
   >
   > bootstrap.system_call_filter: false

## 基本错误处理

```
原因：无法创建本地文件问题,用户最大可创建文件数太小

解决方案：切换到root用户，编辑limits.conf配置文件，  添加类似如下内容：

vi /etc/security/limits.conf

添加如下内容: 注意*不要去掉了

\* soft nofile 65536

\* hard nofile 131072

备注：* 代表Linux所有用户名称（比如  hadoop）

需要保存、退出、重新登录才可生效。
原因：无法创建本地线程问题,用户最大可创建线程数太小

解决方案：切换到root用户，进入limits.d目录下，修改90-nproc.conf 配置文件。

vi /etc/security/limits.d/90-nproc.conf

找到如下内容：

\* soft nproc 1024

\#修改为

\* soft nproc 4096
原因：最大虚拟内存太小

每次启动机器都手动执行下。

root用户执行命令：

执行命令：sysctl -w  vm.max_map_count=262144
sudo sysctl -w vm.max_map_count=262144
查看修改结果命令：sysctl -a|grep  vm.max_map_count  看是否已经修改

永久性修改策略：

echo "vm.max_map_count=262144" >>  /etc/sysctl.echo 
或者
// 拷贝一本临时文件并且覆盖
echo "vm.max_map_count=262144" >> /tmp/system_sysctl.conf
mv /tmp/system_sysctl.conf /etc/sysctl.conf
```

更多问题自行百度解决...

## 基本命令

> 查看集群状态：localhost:9200/_cat/health?v 查看索引列表：localhost:9200/_cat/indices?v

添加一个索引: curl -XPUT '[http://主机IP:9200/dept/employee/32](http://xn--IP-wz2cm89g:9200/dept/employee/32)' -d '{ "empname": "emp32"}'

### 配置重点:

**centeros 6 版本执行下列方法**

1. 开放Linux 9200 端口
2. vi /etc/sysconfig/iptables
3. 加入内容并保存：-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
4. /etc/init.d/iptables restart
5. /sbin/iptables -L -n

# 安装head 插件

### 1.解压插件 (如果不存在需要安装解压软件)

```
yum install zip unzip
```

### 2.安装nodeJS支持,执行如下命令

## 具体参考链接

```
1. curl -sL https://rpm.nodesource.com/setup_8.x | bash -

2. yum install -y nodejs

3. npm install grunt --save-dev

4. 修改head目录下面的 ./_site/app.js

   （无法安装请查看nodejs安装教程）
5. npm install phantomjs-prebuilt@2.1.16 --ignore-scripts(必要依赖)

在 head 根目录下执行: npm run start

6. 如果连接不上需要执行下列命令
    vi el/config/elasticsearch.yml
    
    添加如下
    允许远程访问
    http.cors.enabled: true
    http.cors.allow-origin: "*"
   
```

### 3.app.js修改大概位置

```
init: function(parent) {
    this._super();
    this.prefs = services.Preferences.instance();
    this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://10.215.4.166:9200";
    if( this.base_uri.charAt( this.base_uri.length - 1 ) !== "/" ) {
    // XHR request fails if the URL is not ending with a "/"
    this.base_uri += "/";
}
```

#### 4.更换grunt

```
npm install -g yo bower grunt-cli gulp
```

## 5.添加索引目录位置

```
[root@localhost ~]# vi ~/.bash_profile

# .bash_profile

# Get the aliases and functions

if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/usr/local/src/node-v8.2.1-linux-x86/bin
```

## 6.补充内容

> 现在使用的较多的多半为elasticsearch-head 插件或者自己开发界面以及api接口

### springboot-elasticsearch 部署的配置文件

```
spring.data.elasticsearch.cluster-name=my-es
#注意： 此处不能和elasticsearch服务一致，因为有可能出现端口攻击问题导致服务器崩溃
spring.data.elasticsearch.cluster-nodes=192.168.1.73:9300
spring.data.elasticsearch.repositories.enabled=true
```

### 接口的注入

```
@Component 
//@Repository
public interface ArticleRepository extends ElasticsearchRepository<Article, Long> {
}
```