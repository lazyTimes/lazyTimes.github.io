---
title: nodejs安装教程
subtitle: NodeJS教程
author: lazytime
url_suffix: note12
tags:
  - 前端
categories:
  - 前端-nodejs
keywords: nodejs安装教程（linux）
description: nodejs笔记记录
copyright: true
date: 2020-07-26 22:37:22
---

# nodejs安装教程（linux）

## 1. 下载官方包

### 1.1. 相关命令

```
# wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz    // 下载
# tar xf  node-v10.9.0-linux-x64.tar.xz       // 解压
# cd node-v10.9.0-linux-x64/                  // 进入解压目录
# ./bin/node -v                               // 执行node命令 查看版本
v10.9.0
```

## 2. 设置软连接

```
ln -s /usr/software/nodejs/bin/npm   /usr/local/bin/ 
ln -s /usr/software/nodejs/bin/node   /usr/local/bin/
```

## 3.常见问题解决

```
> cat /etc/redhat-release
> //查看python版本
> python -v
> //查看gcc rpm gcc-c++是否安装
> rpm -q gcc rpm -q gcc-c++
> //安装gcc-c++
> yum -v install gcc-c++ kernel-devel
> //大招荡平一切环境问题
> yum -y update && yum -y groupinstall "Development Tools"

------

//复制官网链接（Source Code版本）进入/usr/src目录下载nodejs
>wget https://nodejs.org/dist/v6.11.4/node-v6.11.4.tar.gz
//解压
>tar -xf node-v6.11.4.tar.gz
//删除压缩包
>rm node-v6.11.4.tar.gz
//进入node-v6.11.4目录，进行配置
>./configure
//编译
>make
//安装
>sodu make install
```

**！！！编译Node 时候发现gcc 版本太低需要升级gcc编译器版本**

```
#获取源码(由于官方镜像速度较慢，这里使用了中国科学院开源协会的镜像
sudo wget http://mirrors.opencas.org/gnu/gcc/gcc-6.3.0/gcc-6.3.0.tar.bz2
#如果以上给出的镜像不可用，也可以是使用http://ftp.gnu.org/gnu/gcc/，但由于有墙的存在，通常这样都很慢，建议本地通过shadowsocks 下载后放到服务器上再继续以下步骤

#解压
sudo tar -jxvf gcc-6.3.0.tar.bz2
#下载编译所需的依赖项
#如果想更快，可以利用中国科学院开源协会的镜像加速下载gmp和mpfr这两个包（另外两个包镜像没有收录），手动替换./contrib/download_prerequisites的以下两处命令：
#1) 把wget ftp://gcc.gnu.org/pub/gcc/infrastructure/$MPFR.tar.bz2 || exit 1 替换成wget http://mirrors.opencas.org/gnu/mpfr/$MPFR.tar.bz2 || exit 1
#2) 把wget ftp://gcc.gnu.org/pub/gcc/infrastructure/$GMP.tar.bz2 || exit 1 替换成wget http://mirrors.opencas.org/gnu/gmp/$GMP.tar.bz2 || exit 1
cd gcc-6.3.0
sudo ./contrib/download_prerequisites
cd ..

#建立编译输出目录
sudo mkdir gcc-build-6.3.0

#进入此目录，执行以下命令，生成makefile文件
cd gcc-build-6.3.0
sudo ../gcc-6.3.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

#执行命令进行编译，此处利用4个job，需编译时约40分钟，此值不宜设置过高
sudo make -j4

#安装

sudo make install
```

强烈建议 centeros使用 7.0 以上版本

## 4. 问题

```
make[2]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2'
make[1]: *** [stage1-bubble] 错误 2
make[1]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2'

## make: *** [all] 错误 2

解决办法
ubuntu： apt-get install gcc g++ 
CentOS：yum install gcc gcc-c++
```

#### 

## 5. Npm 更换淘宝镜像

```
1. npm config set registry https://registry.npm.taobao.org
2. npm install
```

## 6.安装node JS pm2

作用: 后台运行npm start 程序

```
cnpm install pm2 -g 

pm2启动：
pm2 start "/usr/local/src/node/bin/npm" --name "law" -- start .
 
pm2 list
pm2 stop    
pm2 restart 
pm2 delete  

linux 找不到pm2 
ln -s /usr/local/nodebox/nodejs/lib/node_modules/pm2/bin/pm2 /usr/local/bin
```