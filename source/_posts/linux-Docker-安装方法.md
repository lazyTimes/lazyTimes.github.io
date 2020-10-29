---
title: linux Docker 安装方法
subtitle: 如何在Linux上安装docker
author: lazytime
url_suffix: note21
tags:
  - docker
categories:
  - linux
keywords: docker,linux
description: linux安装docker
copyright: true
date: 2020-08-29 22:52:18
---

# 1. linux Docker 安装方法

## 前置条件

- 64-bit 系统
- kernel 3.10+ （内核为 3.1 以上）
- linux 系统

<!-- more -->

## 检查内核版本，返回的值大于3.10即可

```
uname -r
```

## 更新yum

```
yum update
```

## 添加yum仓库（直接拷贝即可）

```
tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```

## 安装

```
yum install -y docker-engine
```

## 启动docker

```
systemctl start docker.service
```

## 验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

```
docker version
```

## 设置开机自启动

```
sudo systemctl enable docker
```

## 查看当前进程

```
docker ps
```

# 2. 什么是Dokcer

一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口；

使用go语言编写，在LCX（linux容器）基础上进行的封装

## 简单来说：

1）就是可以快速部署启动应用 2）实现虚拟化，完整资源隔离 3）一次编写，四处运行（有一定的限制，比如Docker是基于Linux 64bit的，无法在32bit的linux/Windows/unix环境下使用）

## 为什么使用

开箱即用，快速部署，可移植性强，环境隔离

1、提供一次性的环境，假如需要安装Mysql，则需要安装很多依赖库、版本等，如果使用Docker则通过镜像就可以直接启动运行

2、快速动态扩容，使用docker部署了一个应用，可以制作成镜像，然后通过Dokcer快速启动

3、组建微服务架构，可以在一个机器上模拟出多个微服务，启动多个应用

4、更好的资源隔离和共享

## linux 简介：

```
Linux Standard Base的缩写，lsb_release命令用来显示LSB和特定版本的相关信息
	命令： lsb_release -a 
阿里云安装手册：
	https://help.aliyun.com/document_detail/51853.html?spm=a2c4g.11186623.6.820.RaToNY
	
常见问题：
	https://blog.csdn.net/daluguishou/article/details/52080250
Docker 镜像 - Docker images：
            容器运行时的只读模板，操作系统+软件运行环境+用户程序

            class User{
            private String userName;
            private int age;
            }


			Docker 容器 - Docker containers：
					容器包含了某个应用运行所需要的全部环境
					
					 User user = new User()


			Docker 仓库 - Docker registeries： 
					用来保存镜像，有公有和私有仓库，好比Maven的中央仓库和本地私服
					镜像仓库：	
					
					（参考）配置国内镜像仓库：https://blog.csdn.net/zzy1078689276/article/details/77371782

			对比面向对象的方式
			Dokcer 里面的镜像 : Java里面的类 Class
			Docker 里面的容器 : Java里面的对象 Object
			通过类创建对象，通过镜像创建容器
```

## 阿里云安装linux docker (centeros7 版本)

## 使用脚本安装 Docker

1、使用 `sudo` 或 `root` 权限登录 Centos。

2、确保 yum 包更新到最新。

```
$ sudo yum update
```

3、执行 Docker 安装脚本。

```
$ curl -fsSL https://get.docker.com/ | sh
```

执行这个脚本会添加 `docker.repo` 源并安装 Docker。

4、启动 Docker 进程。

```
$ sudo service docker start
```

5、验证 `docker` 是否安装成功并在容器中执行一个测试的镜像。

```
$ sudo docker run hello-world
```

到此，docker 在 CentOS 系统的安装完成。

## 概念

```
Docker 镜像 - Docker images：
		容器运行时的只读模板，操作系统+软件运行环境+用户程序
					 
		 class User{
			private String userName;
			private int age;
		}


Docker 容器 - Docker containers：
    容器包含了某个应用运行所需要的全部环境

    User user = new User()


Docker 仓库 - Docker registeries： 
    用来保存镜像，有公有和私有仓库，好比Maven的中央仓库和本地私服
    镜像仓库：	

（参考）配置国内镜像仓库：	
https://blog.csdn.net/zzy1078689276/article/details/77371782

对比面向对象的方式
Dokcer 里面的镜像 : Java里面的类 Class
Docker 里面的容器 : Java里面的对象 Object
通过类创建对象，通过镜像创建容器
```

# docker 常用命令

常用命令（安装部署好Dokcer后，执行的命令是docker开头）,xxx是镜像名称

```
搜索镜像：docker search xxx
		
		列出当前系统存在的镜像：docker images
		
		拉取镜像：docker pull xxx
			xxx是具体某个镜像名称(格式 REPOSITORY:TAG)
			REPOSITORY：表示镜像的仓库源,TAG：镜像的标签

运行一个容器：docker run -d --name "xdclass_mq" -p 5672:5672 -p 15672:15672 rabbitmq:management
			docker run - 运行一个容器
			-d 后台运行
			-p 端口映射
			rabbitmq:management  (格式 REPOSITORY:TAG)，如果不指定tag，默认使用最新的
			--name "xxx"
		
		列举当前运行的容器：docker ps

		检查容器内部信息：docker inspect 容器名称

		删除镜像：docker rmi IMAGE_NAME
			 强制移除镜像不管是否有容器使用该镜像 增加 -f 参数，

		停止某个容器：docker stop 容器名称

		启动某个容器：docker start 容器名称

		移除某个容器： docker rm 容器名称 （容器必须是停止状态）


	文档：
		https://blog.csdn.net/permike/article/details/51879578
```

# 3. Docker部署Nginx服务器

```
简介：讲解使用Docker部署Nginx服务器实战
		1、获取镜像 
			docker run (首先会从本地找镜像，如果有则直接启动，没有的话，从镜像仓库拉起，再启动)

			docker search nignx
		2、列举
			docker images
		3、拉取
			docker pull nignx
		3、启动
			docker run -d --name "xdclass_nginx" -p 8088:80 nginx
			docker run -d --name "xdclass_nginx2" -p 8089:80 nginx
			docker run -d --name "xdclass_nginx3" -p 8090:80 nginx
		4、访问
			如果是阿里云服务，记得配置安全组，腾讯云也需要配置，这个就是一个防火墙
```

# 4. 构建自己的私人镜像

## 教程



```
1)登录： docker login --username=794666918@qq.com registry.cn-shenzhen.aliyuncs.com		
		2) 推送本地镜像：
		docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/xdclass/xdclass_images:[镜像版本号]
		例子：
		docker tag 2f415b0e9a6e registry.cn-shenzhen.aliyuncs.com/xdclass/xdclass_images:xd_rabbitmq-v1.0.2

		docker push registry.cn-shenzhen.aliyuncs.com/xdclass/xdclass_images:xd_rabbitmq-v1.0.2

		3)拉取镜像
			线上服务器拉取镜像：
				docker login --username=794666918@qq.com registry.cn-shenzhen.aliyuncs.com

				docker pull registry.cn-shenzhen.aliyuncs.com/xdclass/xdclass_images:xd_rabbitmq-v1.0.2

				启动容器：
				docker run -d --name "xdclass_mq" -p 5672:5672 -p 15672:15672 2f415b0e9a6e
1. 登录阿里云Docker Registry
$ sudo docker login --username=xiaolaodong11 registry.cn-shenzhen.aliyuncs.com
用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

您可以在产品控制台首页修改登录密码。

2. 从Registry中拉取镜像
$ sudo docker pull registry.cn-shenzhen.aliyuncs.com/zxd-registry/zxd-res:[镜像版本号]
3. 将镜像推送到Registry
$ sudo docker login --username=xiaolaodong11 registry.cn-shenzhen.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/zxd-registry/zxd-res:[镜像版本号]
$ sudo docker push registry.cn-shenzhen.aliyuncs.com/zxd-registry/zxd-res:[镜像版本号]
请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。

4. 选择合适的镜像仓库地址
从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

如果您使用的机器位于经典网络，请使用 registry-internal.cn-shenzhen.aliyuncs.com 作为Registry的域名登录，并作为镜像命名空间前缀。
如果您使用的机器位于VPC网络，请使用 registry-vpc.cn-shenzhen.aliyuncs.com 作为Registry的域名登录，并作为镜像命名空间前缀。
5. 示例
使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry。

$ sudo docker images
REPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
registry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB
$ sudo docker tag 37bb9c63c8b2 registry-vpc.cn-shenzhen.aliyuncs.com/acs/agent:0.7-dfb6816
使用"docker images"命令找到镜像，将该镜像名称中的域名部分变更为Registry专有网络地址。

$ sudo docker push registry-vpc.cn-shenzhen.aliyuncs.com/acs/agent:0.7-dfb6816
```