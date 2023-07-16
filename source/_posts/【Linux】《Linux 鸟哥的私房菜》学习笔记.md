---
title: LINUX鸟哥的私房菜
subtitle: 鸟哥的私房菜笔记
author: lazytime
url_suffix: unix1
tags:
  - 服务端-Linux
categories:
  - 读书笔记
keywords: 鸟哥的私房菜笔记
description: 鸟哥的私房菜笔记
copyright: true
date: 2020-07-26 16:56:58
---

# 《Linux 鸟哥的私房菜》部分笔记：

## 第一章：计算机概论

- CPU
  - 基本结构
    - 控制单元
    - 存储单元
    - 内存
    - 输入单元
    - 输出单元
  - 内存
  - 指令集
    - 精简指令集 RISC
      - SPARC
      - PowerPC
      - Cell
    - 复杂指令集 CISC
      - AMD
      - Inter
      - VIA
      - X86 的起源
  - Bit： CPU一次读取的最大量
- 人体和电脑设备的比喻
- 电脑分类
  - 超级计算机
  - 大型计算机
  - 迷你计算机
  - 工作站
- 电脑常用的计量单位
  - Mbit
  - Ghz
- CPU的工作频率：外频与倍频
  - 外频：CPU与外部组件数据传输的速度
  - 倍频：CPU内部加速工作性能的倍数
  - 超频
- 内存
  - 多通道设计
  - DDR
  - DRAM 和 SRAM
  - 二级缓存：CPU内部的内存缓存
  - ROM
    - BIOS
    - 现在已写入到闪存或者硬件中
    - 固件
- 显卡
  - 主要的连接接口
    - D-Sub
    - DVI
    - HDMI
    - DisplayPort
- 硬盘：
  - 组成：
    - 碟片
    - 磁头
    - 主轴马达
    - 机器手臂
  - 最小单元：扇区
  - 传输接口：
    - SATA
    - USB
    - SAS
  - 固态硬盘
- 使用须知
- 扩展接口
- 主板
  - 设备IO地址和IRQ中断请求
  - 连接外置设备
- 主机电源
- 数据的表示方式
  - 数字系统
  - 字符编码系统
  - 操作系统
    - 概念
      - 只管理硬件资源
      - 内核参考硬件写成
      - 应用程序参考操作系统
    - 内核功能
      - 系统调用
      - 进程管理
      - 内存管理
      - 文件系统管理
      - 设备驱动
    - 注意点：
  - 重点回顾
    - 计算机定义
    - 计算机五大单元
    - CPU的作用
    - CPU频率，外频和倍频，以及超频
    - 新CPU的主要变化
    - CPU处理数据
    - 内存分类
      - 动态随机存取内存
      - 静态随机存取内存



<!-- more -->

## 第二章：Linux起源

- Unix的发展背景
- GNU计划，开放源代码
- Minix 的发展
- Linux的雏形
  - Minix不满足要求
  - 学到的东西
    - 基础知识和技能
    - 一点成功之后，勇于挑战
    - 把“玩具”发扬光大
- 虚拟团队对于LInux的改进
- Linux版本
  - 主次版本为奇数：开发中
  - 主次版本为偶数：稳定版本
  - 主线版本：长期维护
    - 判断是否为长期版本的办法
      - `uname -r`
- Linux发行版
- Linux应用
  - 云端应用
  - 虚拟化
- 从头学习Linux
  - 选择一本好用的工具书
    - 推荐的网络书： NETMAN
  - 发生问题怎么处理
  - FAQ：
    - /usr/share/doc
    - http://www.tldp.org
  - 必要掌握点：
    - 计算机概论与硬件相关知识
    - 从Linux安装和命令学起
    - Linux操作系统的基本技能
    - 学会VI编辑器
    - Shell与脚本学习
    - 一定要会软件管理
      - Tarball
      - RPM
      - DPKG
      - YUM
      - APT
    - 网络基础学习
    - 网站搭建
  - 网络的书推荐
    - [Http://linux.vbird.org](http://Http://linux.vbird.org)
  - 实践大于一切
  - 习惯：
    - 有系统的设计文件目录
    - 养成做记录的习惯
    - 作为用户人迁就机器，作为开发，机器迁就人
    - 会“偷”，“偷”了会改，改了会变，变则通
  - 兴趣
  - 成就感
  - 建立兴趣
  - 协助回答问题
  - 参与讨论
  - 不同环境，解决办法很多，只要行得通就是好办法

### 简答题：

1. 你再主机上安装了一块网卡，但是开机之后，系统却无法使用，网卡是好的，可能哪里出问题，如何解决？

```
链接：https://www.nowcoder.com/questionTerminal/1dc7f18e85e04269b1a355b32692c8ba?orderByHotValue=1&page=1&onlyReference=false
来源：牛客网

网卡是否启动，是否配置了开机自启。可以修改 /etc/sysconf/network-sripts/ifcfg-ethX，其中X是网卡号，
DEVICE=eth0                              #网卡对应的设备别名
BOOTPROTO=static                    #网卡获得ip地址的方式（默认为dhcp，表示自动获取）
HWADDR=00:07:E9:05:E8:B4    #网卡MAC地址（物理地址）
IPADDR=192.168.100.100          #IP地址
NETMASK=255.255.255.0          #子网掩码 
ONBOOT=yes                              #系统启动时是否激活此设备
```

2.一个操作系统至少可以完整控制整个硬件，请问操作系统要控制哪些单元

> （1）运算单元，用来执行当前指令所规定的算术运算和逻辑运算，具有定点和浮点运算功能；（2）控制单元，指挥微处理器执行指令操作的功能； （3）寄存器组，用来暂存操作数，中间结果和处理结果，它构成了微处理器内部的小型存贮空间，其容量大小影响到微处理器的效率； （4）总线接口单元，提供微处理器与周围其它硬件的接口，有效地将微处理器的地址、数据和控制等信息通过总线和各相关部件接通； （5）输入/[输出接口](https://www.baidu.com/s?wd=输出接口&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)单元。

1. window上的游戏能不能到Linux去玩

```
不能，操作系统不一样
```

1. Unix 是谁写出来的？ GNU 计划是谁发起的？

```
GNU 是 GNU is Not Unix 的简写，是个无穷循环！ 另外，这个计划是由自由软件基金会 (Free Software Foundation, FSF) 所支
持的！ 两者都是由 Stallman 先生所发起的！
```

1. 何谓多人 ( Multi-user ) 多任务 ( Multitask )？

```
Multiuser 指的是 Linux 允许多人同时连上主机之外，每个用户皆有其各人的使用环境，并且可以同时使用系统的资源！
Multitask 指的是多任务环境，在 Linux 系统下， CPU 与其他例如网络资源可以同时进行多项工作， Linux 最大的特色之一
即在于其多任务时，资源分配较为平均
```

1. Linux 本身仅是一个核心与相关的核心工具而已，不过，他已经可以驱动所有的硬件， 所以，可以算是一 个很阳春的操作系统了。经过其他应用程序的开发之后，被整合成为 Linux distribitions。请问众多的 distributions 之间，有何异同？

```
相同：(1)同样使用 http://www.kernel.org 所释出的核心； (2)支持同样的标准，如 FHS、LSB 等； (3)使用几乎相同的自由软
件 (例如 GNU 里面的 gcc/glibc/vi/apache/bind/sendmail... )； (4)几乎相同的操作接口 (例如均使用 bash/KDE/GNOME 等等)。
不同：使用的 kernel 与各软件的版本可能会不同；各开发商加入的应用工具不同，使用的套件管理模式不同(dpkg 与 RPM)
```

1. 什么是 POSIX ?为何说 Linux 使用 POSIX 对于发展有很好的影响？

```
POSIX 是一种标准规范，主要针对在 Unix 操作系统上面跑的程序来进行规范。 若你的操作系统符合 POSIX ，则符合 POSIX
的程序就可以在你的操作系统上面运作。 Linux 由于支持 POSIX ，因此很多 Unix 上的程序可以直接在 Linux 上运作， 因
此程序的移植相当简易！也让大家容易转换平台，提升 Linux 的使用率。
```

## 第三章：主观规划与磁盘分区

- Linux和硬件的搭配
  - 认识计算机硬件
    - 注意硬件的性价比
    - 注意电费
  - 选择Linux搭配的主机
    - CPU
      - i3最低要求
    - 内存
      - 越大越好
    - 硬盘：
      - 通常：20G
      - 高级：磁盘阵列
    - 显卡
      - 32M
    - 网卡：
      - 网络IO频繁要选好网卡
  - 各硬件设备在Linux的文件名
- 磁盘分区
  - 磁盘连接方式和设备文件名的关系
    - 正常：/dev/sd[a-p]
    - 虚拟化环境：/dev/vd[a-p]
    - 决定关系：Linux的检测磁盘顺序
- 分区几点：
  - MBR
  - 四个主分区和一个扩展分区
    - 
  - MBR 与 GPT 磁盘分区表
    - MBR
      - 512字节大小
        - 主引导记录：引导程序的地方，446字节
        - 分区表：记录整个硬盘分区状态，64字节
          - 最多只能有四组记录
          - 记录该区开始和结束柱面号码
          - 注意点
            - 所谓分区仅仅是对于64字节的分区表设置
            - 磁盘默认只能写入四组分区信息
            - 主要分区和扩展分区
            - 最小单位为柱面
          - 写入磁盘必须参考分区表才能操作
      - 数据安全性
      - 系统性能
    - GPT
      - 出现原因
        - MBR无法操作2.2T以上磁盘
        - MBR只有一个区块，破会后无法恢复
        - MBR存放引导程序只有446字节，无法存储较多程序代码
      - 分区
        - 34个LBA区块记录分区
        - LBA0（MBR兼容区块）
          - 存放引导信息
        - LBA1 （GPT表头记录）
          - 分区表的本身位置和大小
          - **后面34个备份GPT分区**
        - LBA2-33(实际分区信息记录)
          - 4 * 32 =128 组
          - 【2^64 * 512 = 2^63 * 1K字节 = 2^33 * TB = 8ZB】 1ZB = 2^30TB
        - 现在的内核使用特殊方式没有所谓的主分区，扩展分区和逻辑分区
- 启动程序BIOS和UEFI启动检测程序
  - BIOS
    - 系统启动的第一个程序
    - 直接写在硬件上的一个程序
    - 启动过程：
      - BIOS:启动固件，认识第一个可启动设备
      - MBR：第一个可启动的设备第一个扇区的主引导记录块，内部引导代码
      - 启动引导程序（boot loader）：可读内核文件的软件
      - 内核文件，开始启动系统
    - Boot Loader 作用
      - 提供不同选项，多重引导
      - 加载内核文件
      - 启动管理功能转交给其他引导程序
        - **启动引导程序可以装在分个分区的启动扇区**
    - 多重引导
      - 每个分区都有自己启动扇区
      - 第一以及第二扇区
      - 实际可启动的内核文件放到各个分区
      - 功能：认识自己分区的可启动内核文件，其他引导程序
      - 直接或者间接管理权给另一个启动引导管理程序
  - UEFI 搭配GPT启动流程
    - 某些时刻可能需要关闭UEFI 的 secure boot 功能
    - 双系统清务必先安装windows
- Linux 安装模式分区选择（极其重要）
  - 目录树结构
    - 根目录
  - 文件系统与目录树挂载
    - 什么是挂载？
      - 利用目录成为进入点，磁盘分区放置到目录下面
      - 进入该目录就可以读取该分区
    - 判断文件在哪个分区
      - 哪个“进入点”先被查到，被查到的就是进入点
  - 规划磁盘分区
    - 自定义安装
      - 初学： “/” 以及 "swap" 分区即可
      - 预备一个备用的剩余磁盘容量
- 安装Linux之前的规划
  - 选择适当发行版
    - 镜像站
- 主机服务和硬件的关系
  - window和Linux共存
  - NAT
  - SAMBA
  - Mail
  - Web
  - DHCP
  - FTP
- 主机硬盘规划
  - 主机硬件出问题，文件能否安全保存
  - 最简单分区办法
    - 如上的新人办法
    - 很不靠谱
  - 稍微麻烦一些
    - 比较符合读写容量大而且读写频繁的场景分区办法
      - /boot
      - /
      - /home
      - /var
      - swap
    - 注意服务种类
- 两个案例
  - 家用小型Linux服务器，IP共享和文件共享中心
  - Linux的PC集群

第三章：安装CenterOs7

本章基本全靠实操，务必多操作几遍

- 练习规划
  - 配置
    - CPU： I5以上
    - 内存：最少提供1.2G以上内存
    - 硬盘：40GB Virtio 接口
    - 网卡：Bridge 桥接对外网卡
    - 显卡：60MB左右显存
    - 其他
      - 键盘
      - 鼠标
      - 屏幕
  - 磁盘分区参考
    - 强制使用GPT模式
    - BIOS boot 2MB
    - /boot
    - /
    - /home
    - swap
  - 启动引导程序
  - 选择软件
  - 检查
- 开始安装CenterOs7
  - 步骤：
    - 调整BIOS
    - 选择安装模式
    - 选择语言
    - 软件选择
    - 磁盘分区
    - 启动引导程序
    - 安装后的首次设置
  - 调整BIOS和虚拟机创建流程
    - `dd if=centeros7.iso of=/dev/sdc`
    - 光盘错误处理
      - 硬件不支持
      - 光盘跳盘
      - 光盘有问题
  - 安装模式启动
    - 正常安装
    - 测试后再安装
    - 除错模式
      - 图形化界面安装
      - 恢复系统
      - 运行内存测试程序
      - 本地磁盘启动，不使用光盘
  - 如何GPT强制执行（关键）
    - 正常安装CENTERos 7
    - 按下Tab键
    - 输入参数，自行百度
- 安装过程
  - 不记录，请看书
- 其他功能：
  - RAM测试，安装笔记本的内核参数
    - 内存压力测试：memtest86
  - 安装笔记本电脑的参数
    - 去掉笔记本的一些配置影响
      - 正常安装centeros 7
      - `nofb apm=off acpi=off pci=noacpi`
      - nofb 取消缓存检测
  - 多重引导安装和管理（可选）
  - 安装规划

## 第四章：首次登陆和在线求助

- 基础命令操作
  - date : 时间
    - `date +%y/%m/%d`
    - `date +%H/%m`
  - cal：日历
    - cal 年份
    - cal 月份 年份
    - cal 13 2015 错误结果
  - bc：计算器
- 重要热键
  - Tab
  - Ctrl-c
  - Ctrl-d
    - 相当于 exit
  - CenterOS7 补全功能有可能补全命令
    - Bash-completion
  - Shift + PageUp 或者 PageDown
    - 相当于翻页
- Linux 在线求助 man page 和 info page
  - g 开头命令
  - man 的组成
    - name
    - synopsis
    - description
    - options
    - command
    - files
    - see also
    - example
  - 看帮助文档技巧
    - 查看Name部分
    - 详细看下Description
    - 如果熟悉命令，直接看options
    - see also 查看相关使用
    - 列举有关的file 部分
  - 查看命令
    - 空格：下翻
    - page down 下翻
    - page up 向上
    - Home 第一页
    - end 最后一页
    - /string 向下查找
    - ?string 向上查找
    - n,N 继续查询
    - q 结束
  - man的位置
    - `/usr/share/man`
    - `/etc/man_db.conf`
  - info
    - 网页显示
    - 默认位置
      - /usr/share/info
    - 内容
      - File
      - Node
      - Next
      - Up
      - Prev
  - 其他有用的文件
    - /usr/share/doc
      - 例子：
        - /usr/share/doc/grub2-tools-20.2
  - nano
    - 简单文本编译器
  - 正确的关机办法
    - 正确使用
      - shutdown
        - /sbin/shutdown [-krhc] [时间] [警告信息]
      - reboot
    - 常用
      - shutdown
    - 重启
      - rebbot
      - halt poweroff
    - 查看状况
      - who
    - 同步写入磁盘
      - sync
    - systemctl 重要命令

### 练习题

1. 终端信息如何来的，/etc/issue 文件当中

```
结果
\S
Kernel \r on an \m
2. 
```

\r 代表内核

\m 硬件等级

1. man issue 查找文件的个数

## 第五章：文件权限和目录

- linux 用户记录和用户身份的文件
  - 记录 `/etc/passwd`
  - `/etc/shadow`
- linux 文件属性
  - drwx------- 5 root root 4096 May 29 16:08
    - 第一个栏目代表文件类型和权限
    - 第一个字符代表是如下
      - 文件
      - 目录
      - 连接
      - b 设备文件
      - c 串行端口设备
    - 接下来设置
      - r 读取
      - w 写
      - x 运行
    - 第一组是自己的权限
    - 第二组是同用户组的权限
    - 第三组为别人的权限
  - 第二个大足 5 代表多少个文件名连接到此节点
  - 第三个代表当前目录或者文件的所有者
  - 第四个代表所在用户组
  - 第五个代表文件大小
  - 地浏览创建日期或者最近修改日期
- 修改文件属性和权限
  - 三个常用命令
    - chgrp
      - 修改所属组
      - `chgrp users init-ss.cfg`
    - chown
      - `chown [-R] 账号:用户组`
    - chmod
      - `chmod xyz 文件或者目录`
      - `chmod` u=rwx,go=rx 文件或者目录
      - `chmod a-x 文件名或者目录`
  - 权限的延伸意义
    - r
      - 可以ls
    - w
      - 建立目录和文件
      - 删除已有文件或目录
      - 更名
      - 移动位置
    - x
      - 目录的x代表能否成为工作目录
    - 总结
      - 分配权限至少需要 rx 的权限
- 文件种类和扩展名
  - 常规文件
    - 纯文本
    - 二进制
    - 数据文件
      - 用户登录记录在 /var/log/wtmp
  - 目录
  - 链接
  - 设备和设备文件
    - 区块设备
    - 字符设备
  - 数据接口
    - /run 或者 /tmp
  - FIFO 数据传输文件
    - 解决并发读写的问题
- Linux文件扩展名
- 文件名字长度限制
  - 单一文件目录最大为255字节
    - 128个汉字左右
  - 避免特殊字符
- FHS 目录配置
  - 可分享和不可分享
  - 不变和可变动
  - 规范
    - / : 和系统有关
    - /usr : 软件的安装和执行有关
    - /var ： 系统运行过程有关
  - 规范要求的目录
    - /
      - /bin
        - 单人维护下依然可以使用的命令
      - /boot
        - 内核常用文件
      - /dev
        - 设备
      - /etc
        - 不要放可执行文件
        - 系统的主要配置文件
      - /lib
        - 库函数
      - /media
        - 媒体设备
      - /mnt
        - 挂载外部硬盘
      - /opt
        - 第三方软件
        - 个人公司的标准
      - /run
        - 新版可以用内存模拟
      - /sbin
        - 只有root 操作的命令
      - /srv
        - service
        - 网络服务
      - /tmp
        - 临时文件
      - /usr
        - /usr/bin
          - 一般用户能使用的命令
        - /usr/lib
          - /lib基本相同的功能
        - /usr/local
          - 系统管理员建议安装目录
        - /usr/sbin
          - 非系统正常运行需要的命令
        - /usr/share
          - **只读**数据文件
        - /usr/games
        - /usr/include
          - c,c++
        - /usr/libexec
          - 不被常用的执行文件或者脚本
        - /usr/lib<qual>/
        - usr/src
          - 源代码建议位置
      - /var
        - /var/cache
          - 程序本身的缓存
        - /var/lib
        - /var/lock
        - /var/log
        - /var/mail
        - /var/run
        - /var/spool
      - /home
        - 用户家目录
        - ~：表示目前用户家目录
        - ~xxx（用户名）：谁的家目录
      - /root
        - 系统管理员的家
      - /lost+found
        - 标准的ext 文件系统的目录
      - /proc
        - 虚拟文件系统
      - /sys
        - 虚拟文件系统
  - 早期系统必备五个目录挂载点
    - /etc /dev /lib /sbin /bin

### 练习题

早期的 Unix 系统文件名最多允许 14 个字符，而新的 Unix 与 Linux 系统中，文件名最多可以容许几个 字符？

- 当一个一般文件权限为 -rwxrwxrwx 则表示这个文件的意义为？

- 我需要将一个文件的权限改为 -rwxr-xr-- 请问该如何下达指令？

- 若我需要更改一个文件的拥有者与群组，该用什么指令？ chown, chgrp

- 请问底下的目录与主要放置什么数据： /etc/, /boot, /usr/bin, /bin, /usr/sbin, /sbin, /dev, /var/ log, /run

  /etc/：几乎系统的所有配置文件案均在此，尤其 passwd,shadow /boot：开机配置文件，也是预设摆放核心 vmlinuz 的地方 /usr/bin, /bin：一般执行档摆放的地方 /usr/sbin, /sbin：系统管理员常用指令集 /dev：摆放所有系统装置文件的目录 /var/log：摆放系统注册表文件的地方 /run：CentOS 7 以后才有，将经常变动的项目(每次开机都不同，如程序的 PID)移动到内存暂存，所以 /run 并不占实 际磁盘容量

- 若一个文件的档名开头为『 . 』，例如 .bashrc 这个文件，代表什么？另外，如何显示出这个文件名与他的 相关属性？

  有『 . 』为开头的为隐藏档，需要使用 ls -a 这个 -a 的选项才能显示出隐藏文件的内容，而使用 ls -al 才能显示出属性。

## 第六章 Linux文件与目录管理

- 目录和路径
- 相对路径和绝对路径
- 目录相关操作
  - . 代表当前
  - ..
  - \- 前一个工作目录
  - ~account 代表账号对应的家目录
- cd
- pwd
- mkdir
- rmdir
- 执行路径的变量（$PATH）
  - 为什么我可以在任何地方执行 ls
  - `echo $PATH`
  - PATH="${PATH}:/root" 添加环境变量
  - 不要用 "." 作为环境变量
  - 可以做的事
    - 不同身份用户默认PATH不一样
    - PATH可以修改
    - 使用绝对或者相对指定某个文件的文件名来执行，比PATH准确 ** 因为环境变量的命令存在重名优先级的问题
    - 本目录不要放到环境变量
- ls
  - `ls --full-time ~`
- cp rm mv
  - cp
    - cp 源文件 目标文件
      - **-a 相当于 -dr---preserve=all 保留指定的属性（默认值：mode，ownership，timestamps）（如果可能）传统属性：上下文，链接，xattr，所有**
      - -d 与--no-dereference --preserve = links相同。源文件为链接，则复制文件属性而非文件
      - -f 如果无法打开现有目标文件，将其删除，然后重试（此选项为 当同时使用-n选项时，将被忽略）
      - **-i 覆盖前提示（覆盖先前的-n选项）**
      - **-p 连同文件的属性一起复制，而非默认（备份常用）**
      - -r 递归复制
      - -s 复制为链接形式
      - -t 根据时间排序
    - 注意需要多文件复制，最后一个一定是目录
    - 执行cp前的思考
      - 是否需要保留完整源文件信息
      - 源文件是否为符号链接
      - 源文件是否是特殊文件
      - 源文件是否为目录！
  - rm
    - rm 文件或者目录
    - 超级危险的命令
  - mv
    - mv source des
      - -f 强制
      - -i 询问
      - -u 只有新文件才能更新
- 获取路径的文件名和目录名称
  - basename
  - dirname
- 文件内容的读取
  - cat 第一行开始
- tac 最后一行开始
  - nl 显示行号
- more 一页一页
  - less 和More差不多，但是可以往前翻页
- head 只看前几行
  - tail 只看后几行
- od 以二进制读取
  - 直接查看内容
    - cat
      - -A 列出一些空白字符不是空白而已
      - 空格和制表符分别代表如下
        - 制表符: ^I
    - -b 列出行号
      - -n 空白行也有行号
    - -E 结尾换行符显示
      - -v 特殊字符显示
    - tac
      - 反向显示
    - nl
      - -b 指定行号的格式
      - -b a 是否空行都有行号
        - -b t 如果有空行，空行不显示
    - -n 列出行号的方法
      - -n ln 左对齐
      - -n rn 右对齐
        - -n rz 自己栏位的最右方显示
    - -w 行号栏位占用字符数
      - `nl -b a -n rz /etc/issue`
      - `nl -b a -n rz -w 3 /etc/issue`
    - more
      - 空格 下一页
      - /字符串
      - :f 立刻显示文件名和显示行数
      - q
      - b 回翻
    - less
      - page up 前翻
      - page down 后翻
      - N 反向重复前一个查找
      - g 前进到这个数据前一行
      - G 前进到这个数据最后一行
    - q 离开
  - head
    - -n 显示几行
    - tail
      - 取后面几行
    - -n 行数
  - 如何去除10到20行的数据
    - `head -n 20 /etc/man_db.conf | tail -n 10`
    - `cat -n /etc/man_db.conf | head -n 20 | tail -n 10`
    - od 非纯文本文件
    - a 默认字符
      - c ascii
    - d 十进制
      - f 浮点数
- 修改文件或者创建新文件
  - 修改时间 mtime
    - 内容修改的时候：如增加或者删除字符
    - 状态时间 ctime
      - 状态被改变：如权限被改了
    - 读取时间 atime
      - 只要被读取
- ls 默认为 m time
  - touch 命令
  - -a 仅自定义access time
    - -c 修改文件的时间，不存在不建立新文件
    - -d 后面可以接预定义的日期而不用目前的时间，
    - -m 仅修改mtime
    - -t 后面接预定义时间 [YYYYMMDDhhmm]
    - 案例
      - `date ; ll --time=atime .; ll --time=ctime .; ll --time=mtime .`
  - 常用情况
    - 建立空文件
    - 某个日期改成目前（mtime, atime）
  - 文件和目录隐藏权限
    - 新增文件和目录之后，默认的权限是什么？
      - umask
        - -S 使用字符表示权限
        - 指的是需要减掉的权限
          - 如0 就是所有权限都不要减掉
        - `umask 002`
        - umask 在搭建文件服务器的作用很大
    - 文件隐藏属性
      - 注意必须在 ext 2 3 4 的系统上完全生效
      - chattr [+-=] 文件和目录
        - \+ 增加一个特殊参数
        - \- 删除特殊参数
        - = 直接设置
        - A 设置这个属性的时候。atime 不会改动、避免过度读写磁盘
        - S 非同步写磁盘，修改文件会同步写入磁盘
        - a 设置之后只能增加数据,不能删除和修改，只有root有权限
        - c 默认对文件压缩存储
        - d 阻止dump程序dump文件
        - **i 让文件不能删除，改名，设置连接也无法写入或者新增， 只有root有权限**
        - s 如果设置s，文件被删除，完全从磁盘删除
        - u 相反，被删除了，内容还在磁盘，用于文件恢复
      - 注意点：
        - a 和 i 比较常用
        - xfs 文件系统仅仅支持 AadiS
      - lsattr [-adR] 文件或者目录
        - -a 隐藏文件也显示
        - -d 如果是目录，仅仅列出目录
        - -R 子目录一起列出来
  - 文件特殊权限
    - SUID
      - 只对二进制文件有效
      - 执行者对于该程序具有x可执行权限
      - 仅在执行过程中有效
      - 执行者具有拥有者的权限
      - 处理密码这种机密文件只能让root修改，但是又要让用户可以修改自己密码的时候这种情况的处理方式
      - 案例
        - /etc/bin/passwd
        - 用户对于passwd 有执行权限，所以是执行者
        - passwd 拥有者是root
        - 用户执行passwd的时候，会得到root权限
        - 由于有暂时的root权限，所以可以修改密码，但是只能修改自己的那部分
      - **只能是二进制文件，只能是二进制文件，只能是二进制文件**
    - SGID
      - ls -l /usr/bin/local
      - 文件的情况
        - 对于二进制有用
        - 程序执行者对于程序来说是 x权限
        - 执行者在执行过程中获得用户组的支持
      - 目录的情况
        - r和x权限，可以进入此目录
        - 此目录的有效用户组，会变成改目录的用户组
        - 如果在此目录新建文件，该文件的用户组会和此目录的用户组相同
      - 举例
        - `sudo ls -l /usr/bin/locate /var/lib/mlocate/mlocate.db`
    - SBIT
      - 用户对于此目录具有w,x权限的时候，具有写入的权限
      - 该目录新建文件的时候，仅有自己和root有权利删除该文件
    - 后面的章节再来回顾,请看16章
  - 观察文件类型 file 命令
    - 案例 `file /etc/bin/passwd`
- 文件的查找
  - which
    - -a 将所有由PATH 环境变量可以找到的列出来
    - 注意是以 PATH为环境变量起点的
  - find
    - 能不用就不用
  - whereis
    - -l 列出会去查看的几个主要目录
    - -b 只看二进制文件
    - -m 只查找说明文件Manual 路径的文件( man 记录的)
    - -s 执照源文件
    - -u 查找不在上述文件的三个文件
    - 案例
      - 找出 ifconfig 文件名
      - 执照出和passwd有关的说明文件
  - **locate** / updatedb
    - -I 忽略大小写
    - -c 不输出文件名，只输出数量
    - -l 输出几行，五行就是 -l 5
    - -S 输出locate 使用的数据库信息
    - -r 正则方式
    - 案例
      - 找出系统中和passwd 相关的所有文件名，只需要5个
      - 列出locate查询使用的信息和列出数据的数量
    - 注意点
      - 新文件有可能找不到，因为数据库一般是一天更新一次
      - 更新方法：updatedb
    - 原理
      - 按照 /var/lib/mlocate 数据库近路，找出关键词的文件名
    - 

### 练习题

情境模拟题一：假设系统中有两个账号，分别是 alex 与 arod ，这两个人除了自己群组之外还共同支持一个名为 project 的群组。假设这两个用户需要共同拥有 /srv/ahome/ 目录的开发权，且该目录不许其他人进入查阅。 请问 该目录的权限设定应为何？请先以传统权限说明，再以 SGID 的功能解析。  目标：了解到为何项目开发时，目录最好需要设定 SGID 的权限！  前提：多个账号支持同一群组，且共同拥有目录的使用权！  需求：需要使用 root 的身份来进行 chmod, chgrp 等帮用户设定好他们的开发环境才行！ 这也是管理员的 重要任务之一！

### 简答题

什么是绝对路径与相对路径 绝对路径的写法为由 / 开始写，至于相对路径则不由 / 开始写！此外，相对路径为相对于目前工作目录的路径！  如何更改一个目录的名称？例如由 /home/test 变为 /home/test2 mv /home/test /home/test2  PATH 这个环境变量的意义？ 这个是用来指定执行文件执行的时候，指令搜寻的目录路径。  umask 有什么用处与优点？ umask 可以拿掉一些权限，因此，适当的定义 umask 有助于系统的安全， 因为他可以用来建立默认的目录或文件的权限。  当一个使用者的 umask 分别为 033 与 044 他所建立的文件与目录的权限为何？ 在 umask 为 033 时，则预设是拿掉 group 与 other 的 w(2)x(1) 权限，因此权限就成为『文件 -rw-r--r-- ， 目录 drwxr--r-- 』 而当 umask 044 时，则拿掉 r 的属性，因此就成为『文件 -rw--w--w-，目录 drwx-wx-wx』  什么是 SUID ？ 当一个指令具有 SUID 的功能时，则： o SUID 权限仅对二进制程序(binary program)有效； o 执行者对于该程序需要具有 x 的可执行权限； o 本权限仅在执行该程序的过程中有效 (run-time)； o 执行者将具有该程序拥有者 (owner) 的权限。  当我要查询 /usr/bin/passwd 这个文件的一些属性时(1)传统权限；(2)文件类型与(3)文件的隐藏属性，可以使 用什么指令来查询？ ls -al file lsattr  尝试用 find 找出目前 linux 系统中，所有具有 SUID 的文件有哪些？ find / -perm +4000 -print  找出 /etc 底下，文件大小介于 50K 到 60K 之间的文件，并且将权限完整的列出 (ls -l)： find /etc -size +50k -a -size -60k -exec ls -l {} ; 注意到 -a ，那个 -a 是 and 的意思，为符合两者才算成功  找出 /etc 底下，文件容量大于 50K 且文件所属人不是 root 的档名，且将权限完整的列出 (ls -l)； find /etc -size +50k -a ! -user root -exec ls -ld {} ; find /etc -size +50k -a ! -user root -type f -exec ls -l {} ; 上面两式均可！注意到 ! ，那个 ! 代表的是反向选择，亦即『不是后面的项目』之意！  找出 /etc 底下，容量大于 1500K 以及容量等于 0 的文件： find /etc -size +1500k -o -size 0 相对于 -a ，那个 -o 就是或 (or) 的意思啰！

## 第七章 Linux 磁盘和文件系统的管理

- 磁盘组成和分区复习

  - 请看第一章对于磁盘的记录
  - GPT和MBR分区
  - 磁盘组成

- 文件系统的特性

  - Linux正统文件系统为ext2
    - windows 不支持 ext2 文件系统
  - 通常一个可挂载的数据为一个文件系统而不是分区
    - 通常将文件权限和文件属性放在不同的区块
      - 权限和属性放到inode
      - 实际数据放到数据区块
    - 每个inode都有编号
      - 超级区块
        - 文件系统的整体信息，inode 数据区块的总量，使用量，剩余量，文件系统的格式和相关信息
      - inode 记录文件的属性，一个文件占用一个inode, 记录文件的数据所在区块号码
      - 数据区块：实际记录的文件内容，文件过大会占用多个区块
    - 这种方式是**索引式文件系统**
    - ext2 的限制
      - 区块的大小和数量格式化之后不能修改
      - 每个区块最多防止一个文件的数据
      - 承上，如果文件大于区块，占用多个区块
      - 承上，反之，剩余空间会被浪费
  - inode table
    - inode 记录如下
      - 读写属性
      - 拥有者和用户组
      - 文件大小
      - 建立或者状态改变的时间 ctime
      - 最后一次读取时间
      - 最后修改时间
      - 定义文件的标识
      - 文件真正的内容指向
    - 特点
      - 每个inode 大小固定128B （ext4 到了 256B）
      - 文件占用一个 inode
      - 建立文件数量和inode数量有关
      - 读取文件需要先找到inode, 分析Inode 记录的权限和用户是否符合，符合才可以读取
    - 1KB的块最大单一文件限制为16GB是如何计算的？
      - 12个直接、一个间接、一个三间接
      - 12个直接指向 12 * 1 K = 12 K
      - 间接 256*1K = 256K
      - 1的大小可以存256条记录
      - 双间接
        - 256*256* 1K = 256*2K
      - 三间接
        - 256 * 256 * 256
      - 总额，三者相加 = 12 + 256 +256 * 256 + 256 * 256 * 256 = 16GB
  - 超级区块
    - 主要信息
      - 数据区块 和 inode 总量
      - 未使用和已经使用的inode 数量
      - 数据区块的inode 大小
      - 文件系统的挂载时间、最近一次的写入数据时间、最近一次检验磁盘时间、文件系统的信息
      - 一个有效数值、如果此文件已经被挂载，有效为0，否则为1
    - 区块对照表
    - inode 对照表
    - dump2fs

- 和目录树的关系

  - 读取一个文件路径的底层步骤
  - 例子
    - 读取/etc/passwd 的流程如下
      - / 的 inode
      - / 的区块
      - etc/ 的 inode
      - etc/ 的区块
      - passwd 的 inode
      - passwd 的 区块

- ext 文件的存取和日志式文件系统功能

  - 新增文件系统操作
    - 确定是否有w和x权限
    - 根据inode 对照表找到对应的inode号码，新文件写入
    - 根据区块对照表找到没有使用的区块代码，将实际的数据写入区块，更新inode的区块数据
    - 将刚刚写入的inode 与区块数据同步更新inode对照表和区块对照表，并更新超级区块的内容
    - 同步操作
  - 日志式文件系统
    - 预备：写入一个文件的时候，需要日志记录区块记录某个文件要写入的信息
    - 实际写入：开始写入文件权限和数据，更新Metadata 的数据
    - 结束：完成数据和metadata的更新，在日志记录区块记录

- Linux文件系统的执行

  - 异步处理
    - 没被修改的数据设置为clear
    - 被修改后，设置为脏数据
    - 此时操作还在进行，没有写入磁盘
  - 文件系统贺内存的关系
    - 系统会常用文件数据放到内存缓冲区，加速读写
    - Linuxd物理内存最后都会用光，这才是正常 情况
    - 手动sync 强制内存脏数据回写磁盘
    - 正常关机，会主动sync回写磁盘
    - 不正常关机，需要重启之后磁盘校验，甚至文件系统的损坏

- 挂载点的意义

  - 传统文件系统：
    - ext2
    - minix
    - fat
    - iso9660
  - 日志式文件系统
    - ext3
    - ext4
    - reiserFs
    - windows ntfs
    - ibm jfs
    - sgi xfs
    - zfs
  - 网络文件系统
    - nfs
    - smbfs
  - 查询
    - `ls -l /lib/moudules/$(uname -r)/kernel/fs`
    - `cat /proc/filesystems`
  - linux VFS

- XFS 文件系统

  - ext的问题

    - 支持面最广，格式化最慢

  - 数据区

    - inode 容量可以调整
    - 实际最高可使用区块为4K
    - inode 可以再 256b - 2MB

  - 文件系统活动登录区

  - 实时运行区

  - 查看命令

    - ```
      xfs_info 挂载点|设备
      ```

      - `xfs_info /dev/vda2`
      - `df -T /boot`

- 文件系统的简单操作

  - 磁盘和目录的容量

    - df

      - 列出文件系统的整体磁盘使用量

      - 参数

        - -a 列出所有文件系统
        - -k kbyte 显示文件系统容量
        - -m mbbyte 显示文件系统容量
        - **-h 以人们较容易阅读的Gbyte,mbyte,kbyte 等格式自行显示**
        - -H 以 M=1000K 替换 M=1024K进位方式
        - -T 连同磁盘分区文件系统名称也列出
        - **-i 不用磁盘容量，inode 数量显示**

      - 格式

        - Filesystem: 文件系统是哪个硬盘分区
        - 1K-blocks：说明下面的数字是1KB
        - Used：使用掉的
        - Available：剩下的磁盘大小
        - Use%：使用率，如果大于90%注意一下
        - mounted on：磁盘的挂载目录

      - 案例

        - ```
          df -aT
          ```

          - 列出系统内所有特殊文件以及名称都列出来

        - ```
          df -h
          ```

          - 易读方式读取

    - du

      - 查看文件系统的磁盘使用量（常用在查看目录所占用磁盘空间）
      - 参数
        - -a 列出所有的文件和目录容量，因为默认仅仅是统计目录下面的文件量
        - -h 易读的方式显示
        - -s 仅仅李处总量，列出各个的目录占用量
        - -S 不包含子目录下面的总计，与-s有点差别
        - -k kb 列出
        - -m mb列出
      - 案例
        - `du`
        - `du -sm /*`

- 硬链接和符号链接

  - 回顾
    - 每个文件占用一个inode
    - 读取文件必须要inode号码
  - 硬链接知识某个目录增加一个文件名链接到Inode的关联记录
  - 任何一个Inode硬链接删除，都不会影响另一个文档
  - 缺点：
    - 不能链接目录
      - 为什么？
    - 不能跨文件系统
      - 为什么？
  - 符号链接
    - 特点
      - 相当于windows 快捷方式，但是改变这个快捷方式的内容会直接影响源文档的内容
      - 相当于文件引用的完全拷贝
      - 如果源文件删除，引用会找不到内容
    - 符号链接的容量计算
      - 根据引用对象文件名字长度来确定
    - Ln 默认使用硬链接
    - 参数
      - ln
        - -s 符号链接，如果建立必须此参数
        - -f 如果文件存在，删除后建立

- 磁盘的格式化、分区、校验、挂载

  - 新增磁盘的做法

    - 对磁盘划分，建立可用分区
    - 对于磁盘进行格式化，建立分区可用文件系统
    - 仔细一点，则对于文件系统校验，
    - Linux 上需要建立挂载点，挂载上来

  - 如何知道自己的系统的文件系统

    - lsblk device

    - 选项与参数： -d ：仅列出磁盘本身，并不会列出该磁盘的分区数据 -f ：同时列出该磁盘内的文件系统名称 -i ：使用 ASCII 的线段输出，不要使用复杂的编码 (再某些环境下很有用) -m ：同时输出该装置在 /dev 底下的权限数据 (rwx 的数据) -p ：列出该装置的完整文件名！而不是仅列出最后的名字而已。 -t ：列出该磁盘装置的详细数据，包括磁盘队列机制、预读写的数据量大小等

    - 显示列解读

       NAME：就是装置的文件名啰！会省略 /dev 等前导目录！  MAJ:MIN：其实核心认识的装置都是透过这两个代码来熟悉的！分别是主要：次要装置代码！  RM：是否为可卸除装置 (removable device)，如光盘、USB 磁盘等等  SIZE：当然就是容量啰！  RO：是否为只读装置的意思  TYPE：是磁盘 (disk)、分区槽 (partition) 还是只读存储器 (rom) 等输出  MOUTPOINT：就是前一章谈到的挂载点

    - 案例

      - `lsblk -ip /dev/vda`

  - blkid 列出设备的 UUID等参数

  - parted 列出磁盘的 分区 表类型与 分区

    - `parted /dev/vda print`

    - ```
      Model: Virtio Block Device (virtblk)
      Disk /dev/vda: 42.9GB
      Sector size (logical/physical): 512B/512B
      Partition Table: msdos
      Disk Flags: 
      
      Number  Start   End     Size    Type     File system  标志
       1      1049kB  42.9GB  42.9GB  primary  ext4         启动
      ```

  - 磁盘分区：gdisk / fdisk

    - MBR 分区表请使用 fdisk 分区， GPT 分区表请 使用 gdisk 分区！

    - gdisk

      - gdisk 设备名称

      - 显示参数

         Number：分区槽编号，1 号指的是 /dev/vda1 这样计算。  Start (sector)：每一个分区槽的开始扇区号码位置  End (sector)：每一个分区的结束扇区号码位置，与 start 之间可以算出分区槽的总容量  Size：就是分区槽的容量了  Code：在分区槽内的可能的文件系统类型。Linux 为 8300，swap 为 8200。不过这个项目只是一个提示而 已，不见得真的代表此分区槽内的文件系统喔！  Name：文件系统的名称等等。

    - 注意不要加数字

    - gdisk 新增分区

      - `cat /proc/partitions`

    - partprobe 更新 linux 内核的分区表信息

      - partprobe -s 需要root权限
      - 查看内核分区记录
        - `cat /proc/partitions`

    - 用gdisk 删除一个分区

  - fdisk

    - `fdisk /dev/vda`

      ```
      a toggle a bootable flag
      b edit bsd disklabel
      c toggle the dos compatibility flag
      d delete a partition <==删除一个 partition
      l list known partition types
      m print this menu
      n add a new partition <==新增一个 partition
      o create a new empty DOS partition table
      p print the partition table <==在屏幕上显示分区表
      q quit without saving changes <==不储存离开 fdisk 程序
      s create a new empty Sun disklabel
      t change a partition's system id
      u change display/entry units
      v verify the partition table
      w write table to disk and exit <==将刚刚的动作写入分区表
      x extra functionality (experts only)
      ```

- 磁盘格式化

  - xfs 文件系统 Mkfs.xfs

    - `mkfs.xfs [ - b bsize] [ - d parms] [- i parms] [ [ - l parms] [ - L label] [ - f] \ [ [- - r parms] 装置名称`
    - 参数

    ```
    选项与参数：
    关于单位：底下只要谈到『数值』时，没有加单位则为 bytes 值，可以用 k,m,g,t,p (小写)等来解释
    比较特殊的是 s 这个单位，它指的是 sector 的『个数』喔！
    -b ：后面接的是 block 容量，可由 512 到 64k，不过最大容量限制为 Linux 的 4k 喔！
    -d ：后面接的是重要的 data section 的相关参数值，主要的值有：
    agcount=数值 ：设定需要几个储存群组的意思(AG)，通常与 CPU 有关
    agsize=数值 ：每个 AG 设定为多少容量的意思，通常 agcount/agsize 只选一个设定即可
    file ：指的是『格式化的装置是个文件而不是个装置』的意思！(例如虚拟磁盘)
    size=数值 ：data section 的容量，亦即你可以不将全部的装置容量用完的意思
    su=数值 ：当有 RAID 时，那个 stripe 数值的意思，与底下的 sw 搭配使用
    sw=数值 ：当有 RAID 时，用于储存数据的磁盘数量(须扣除备份碟与备用碟)
    sunit=数值 ：与 su 相当，不过单位使用的是『几个 sector(512bytes 大小)』的意思
    swidth=数值 ：就是 su*sw 的数值，但是以『几个 sector(512bytes 大小)』来设定
    -f ：如果装置内已经有文件系统，则需要使用这个 -f 来强制格式化才行！
    -i ：与 inode 有较相关的设定，主要的设定值有：
    size=数值 ：最小是 256bytes 最大是 2k，一般保留 256 就足够使用了！
    internal=[0|1]：log 装置是否为内建？预设为 1 内建，如果要用外部装置，使用底下设定
    logdev=device ：log 装置为后面接的那个装置上头的意思，需设定 internal=0 才可！
    size=数值 ：指定这块登录区的容量，通常最小得要有 512 个 block，大约 2M 以上才行！
    -L ：后面接这个文件系统的标头名称 Label name 的意思！
    -r ：指定 realtime section 的相关设定值，常见的有：
    extsize=数值 ：就是那个重要的 extent 数值，一般不须设定，但有 RAID 时，
    最好设定与 swidth 的数值相同较佳！最小为 4K 最大为 1G 。
    ```

  - 其他

    ```
    范例：找出你系统的 CPU 数，并据以设定你的 agcount 数值
    [root@study ~]#  grep  'processor' /proc/cpuinfo
    processor : 0
    processor : 1
    # 所以就是有两颗 CPU 的意思，那就来设定设定我们的 xfs 文件系统格式化参数吧！！
    [root@study ~]#  mkfs.xfs - - f - - d agcount=2 /dev/vda4
    meta-data=/dev/vda4 isize=256 agcount=2, agsize=131072 blks
    = sectsz=512 attr=2, projid32bit=1
    = crc=0 finobt=0
    .....(底下省略).....
    # 可以跟前一个范例对照看看，可以发现 agcount 变成 2 了喔！
    # 此外，因为已经格式化过一次，因此 mkfs.xfs 可能会出现不给你格式化的警告！因此需要使用 -f
    ```

  - ext4 文件系统mkfs.ext4

    - `mkfs.ext4 [- - b size] [- - L label] 装 置名`

    选项与参数： -b ：设定 block 的大小，有 1K, 2K, 4K 的容量， -L ：后面接这个装置的标头名称

  - 其他文件系统 mkfs

- 文件系统检验

  - `xfs_repair` 处理 XFS 文件系统

    - `xfs_repair [ - fnd] 装 置名`

    ```
    选项与参数：
    -f ：后面的装置其实是个文件而不是实体装置
    -n ：单纯检查并不修改文件系统的任何数据 (检查而已)
    -d ：通常用在单人维护模式底下，针对根目录 (/) 进行检查与修复的动作！很危险！不要随便使用
    ```

    - 案例

    ```
     xfs_repair /dev/vda4
     
     范例：检查一下系统原本就有的 /dev/centos/home 文件系统
    [root@study ~]#  xfs_repair /dev/centos/home
    ```

  - fsck.ext4 处理 EXT4 文件系统

    ```
    fsck.ext4 [ - pf] [ - b superblock]  装 置名 称
    
    选项与参数：
    -p ：当文件系统在修复时，若有需要回复 y 的动作时，自动回复 y 来继续进行修复动作。
    -f ：强制检查！一般来说，如果 fsck 没有发现任何 unclean 的旗标，不会主动进入
    细部检查的，如果您想要强制 fsck 进入细部检查，就得加上 -f 旗标啰！
    -D ：针对文件系统下的目录进行优化配置。
    -b ：后面接 superblock 的位置！一般来说这个选项用不到。但是如果你的 superblock 因故损毁时，
    透过这个参数即可利用文件系统内备份的 superblock 来尝试救援。一般来说，superblock 备份在：
    1K block 放在 8193, 2K block 放在 16384, 4K block 放在 32768
    ```

  - 只有在极度严重的问题才使用这些命令

  - 文件系统挂载与卸除

  - 前提

    - 单一文件系统不应该被重复挂载在不同的挂载点(目录)中；
    - 单一目录不应该重复挂载多个文件系统；
    - 要作为挂载点的目录，理论上应该都是空目录才是。

  - 挂载方式

    - 案例

    ```
    [root@study ~]#  mount - -a a
    [root@study ~]#  mount [- - l]
    [root@study ~]#  mount [- - t  文件系 统 ] LABEL='' 挂 载点
    [root@study ~]#  mount [- - t  文件系 统 ] UUID='' 挂 载点 # 鸟哥近期建议用这种方式喔！
    [root@study ~]#  mount [- - t  文件系 统 ]  装 置文件名 挂 载点
    选项与参数：
    -a ：依照配置文件 /etc/fstab 的数据将所有未挂载的磁盘都挂载上来
    -l ：单纯的输入 mount 会显示目前挂载的信息。加上 -l 可增列 Label 名称！
    -t ：可以加上文件系统种类来指定欲挂载的类型。常见的 Linux 支持类型有：xfs, ext3, ext4,
    reiserfs, vfat, iso9660(光盘格式), nfs, cifs, smbfs (后三种为网络文件系统类型)
    -n ：在默认的情况下，系统会将实际挂载的情况实时写入 /etc/mtab 中，以利其他程序的运作。
    但在某些情况下(例如单人维护模式)为了避免问题会刻意不写入。此时就得要使用 -n 选项。
    -o ：后面可以接一些挂载时额外加上的参数！比方说账号、密码、读写权限等：
    async, sync: 此文件系统是否使用同步写入 (sync) 或异步 (async) 的
    内存机制，请参考文件系统运作方式。预设为 async。
    atime,noatime: 是否修订文件的读取时间(atime)。为了效能，某些时刻可使用 noatime
    ro, rw: 挂载文件系统成为只读(ro) 或可擦写(rw)
    auto, noauto: 允许此 filesystem 被以 mount -a 自动挂载(auto)
    dev, nodev: 是否允许此 filesystem 上，可建立装置文件？ dev 为可允许
    suid, nosuid: 是否允许此 filesystem 含有 suid/sgid 的文件格式？
    exec, noexec: 是否允许此 filesystem 上拥有可执行 binary 文件？
    user, nouser: 是否允许此 filesystem 让任何使用者执行 mount ？一般来说，
    mount 仅有 root 可以进行，但下达 user 参数，则可让
    一般 user 也能够对此 partition 进行 mount 。
    defaults: 默认值为：rw, suid, dev, exec, auto, nouser, and async
    remount: 重新挂载，这在系统出错，或重新更新参数时，很有用！
    ```

    - /etc/filesystems：系统指定的测试挂载文件系统类型的优先级；
    - /proc/filesystems：Linux 系统已经加载的文件系统类型

  - 我们 Linux 支持的文件系统之驱动程序都写在如下的目录中

    - `/lib/modules/$(uname -r)/kernel/fs/`

  - 挂载 xfs/ext4/vfat

    - 范例

    ```
    范例：找出 /dev/vda4 的 UUID 后，用该 UUID 来挂载文件系统到 /data/xfs 内
    [root@study ~]#  blkid /dev/vda4
    /dev/vda4: UUID="e0a6af55-26e7-4cb7-a515-826a8bd29e90" TYPE="xfs"
    
    [root@study ~]#  mount UUID="e0a6af55- - 26e7- - 4cb7- - a515- - 826a8bd29e90" /data/xfs
    mount: mount point /data/xfs does not exist # 非正规目录！所以手动建立它！
    [root@study ~]#  mkdir - - p /data/xfs
    [root@study ~]#  mount UUID="e0a6af55- - 26e7- - 4cb7- - a515- - 826a8bd29e90" /data/xfs
    [root@study ~]#  df /data/xfs
    Filesystem 1K-blocks Used Available Use% Mounted on
    /dev/vda4 1038336 32864 1005472 4% /data/xfs
    
    # 顺利挂载，且容量约为 1G 左右没问题！
    范例：使用相同的方式，将 /dev/vda5 挂载于 /data/ext4
    [root@study ~]#  blkid /dev/vda5
    /dev/vda5: UUID="899b755b-1da4-4d1d-9b1c-f762adb798e1" TYPE="ext4"
    [root@study ~]#  mkdir /data/ext4
    [root@study ~]#  mount UUID="899b755b- - 1da4- - 4d1d- - 9b1c- - f762adb798e1" /data/ext4
    [root@study ~]# 
    ```

  - 挂载 cd 或者 dvd

    ```
    范例：将你用来安装 Linux 的 CentOS 原版光盘拿出来挂载到 /data/cdrom！
    [root@study ~]#  blkid
    .....(前面省略).....
    /dev/sr0: UUID="2015-04-01-00-21-36-00" LABEL="CentOS 7 x86_64" TYPE="iso9660" PTTYPE="dos"
    [root@study ~]#  mkdir /data/cdrom
    [root@study ~]#  mount /dev/sr0 /data/cdrom
    mount: /dev/sr0 is write-protected, mounting read-only
    [root@study ~]#  df /data/cdrom
    Filesystem 1K-blocks Used Available Use% Mounted on
    /dev/sr0 7413478 7413478 0 100% /data/cdrom
    # 怎么会使用掉 100% 呢？是啊！因为是 DVD 啊！所以无法再写入了啊！
    ```

  - 挂载 vfat 中文随身碟 (USB

    - 案例
    - 注意：不能够是 NTFS 的文件系统

    ```
    范例：找出你的随身碟装置的 UUID，并挂载到 /data/usb 目录中
    [root@study ~]#  blkid
    /dev/sda1: UUID="35BC-6D6B" TYPE="vfat"
    [root@study ~]#  mkdir /data/usb
    [root@study ~]# mount - - o codepage=950,iocharset=utf8 UUID="35BC- - 6D6B" /data/usb
    [root@study ~]#  # mount - - o codepage=950,iocharset=big5 UUID="35BC- - 6D6B" /data/usb
    [root@study ~]#  df /data/usb
    Filesystem 1K-blocks Used Available Use% Mounted on
    /dev/sda1 2092344 4 2092340 1% /data/usb
    ```

    - 重新挂载根目录与挂载不特定目录

      - 案例

      ```
      范例：将 / 重新挂载，并加入参数为 rw 与 auto
      [root@study ~]# mount - - o remount,rw,auto /
      ```

      ```
      范例：将 /var 这个目录暂时挂载到 /data/var 底下：
      [root@study ~]#  mkdir /data/var
      [root@study ~]#  mount  -- bind /var /data/var
      [root@study ~]#  ls - - lid /var  /data/var
      16777346 drwxr-xr-x. 22 root root 4096 Jun 15 23:43 /data/var
      16777346 drwxr-xr-x. 22 root root 4096 Jun 15 23:43 /var
      # 内容完全一模一样啊！因为挂载目录的缘故！
      [root@study ~]#  mount | grep var
      ```

  - umount ( 将装置 文件 卸除)

    - 案例

    ```
    [root@study ~]#  umount [- - fn]  装 置文件名或挂 载点
    选项与参数：
    -f ：强制卸除！可用在类似网络文件系统 (NFS) 无法读取到的情况下；
    -l ：立刻卸除文件系统，比 -f 还强！
    -n ：不更新 /etc/mtab 情况下卸除。
    就是直接将已挂载的文件系统给他卸除即是！卸除之后，可以使用 df 或 mount 看看是否还存在目
    录树中？ 卸除的方式，可以下达装置文件名或挂载点，均可接受啦！底下的范例做看看吧！
    范例：将本章之前自行挂载的文件系统全部卸除：
    [root@study ~]#  mount
    .....(前面省略).....
    /dev/vda4 on /data/xfs type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=256k,sunit=512,..)
    /dev/vda5 on /data/ext4 type ext4 (rw,relatime,seclabel,data=ordered)
    /dev/sr0 on /data/cdrom type iso9660 (ro,relatime)
    /dev/sda1 on /data/usb type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=950,iocharset=...)
    /dev/mapper/centos-root on /data/var type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
    # 先找一下已经挂载的文件系统，如上所示，特殊字体即为刚刚挂载的装置啰！
    # 基本上，卸除后面接装置或挂载点都可以！不过最后一个 centos-root 由于有其他挂载，
    # 因此，该项目一定要使用挂载点来卸除才行！
    [root@study ~]#  umount /dev/vda4 <==用装置文件名来卸除
    [root@study ~]#  umount /data/ext4 <==用挂载点来卸除
    [root@study ~]#  umount /data/cdrom <==因为挂载点比较好记忆！
    [root@study ~]#  umount /data/usb
    [root@study ~]#  umount /data/var <==一定要用挂载点！因为装置有被其他方式挂载
    ```

- 磁盘 文件系统参数自定义

  - mknod

    - 就是透过文件的 major 与 minor 数值来替代的
    - 手动处理装置文件案例

    ```
    [root@study ~]#  mknod  装 置文件名 [bcp] [Major] [Minor]
    选项与参数：
    装置种类：
    b ：设定装置名称成为一个周边储存设备文件，例如磁盘等；
    c ：设定装置名称成为一个周边输入设备文件，例如鼠标/键盘等；
    p ：设定装置名称成为一个 FIFO 文件；
    Major ：主要装置代码；
    Minor ：次要装置代码；
    范例：由上述的介绍我们知道 /dev/vda10 装置代码 252, 10，请建立并查阅此装置
    [root@study ~]#  mknod /dev/vda10 b 252 10
    [root@study ~]#  ll /dev/vda10
    brw-r--r--. 1 root root 252, 10 Jun 24 23:40 /dev/vda10
    # 上面那个 252 与 10 是有意义的，不要随意设定啊！
    范例：建立一个 FIFO 文件，档名为 /tmp/testpipe
    [root@study ~]#  mknod /tmp/testpipe p
    [root@study ~]#  ll /tmp/testpipe
    prw-r--r--. 1 root root 0 Jun 24 23:44 /tmp/testpipe
    # 注意啊！这个文件可不是一般文件，不可以随便就放在这里！
    # 测试完毕之后请删除这个文件吧！看一下这个文件的类型！是 p 喔！^_^
    [root@study ~]#  rm /dev/vda10 /tmp/testpipe
    rm: remove block special file '/dev/vda10' ? y y
    rm: remove fifo '/tmp/testpipe' ? y y
    ```

    - xfs_admin 修改 XFS 文件系统的 UUID 与 Label name

      - 案例

      ```
      xfs_admin [-lu] [-L label] [-U uuid] 设备文件
      选项与参数：
      -l ：列出这个装置的 label name
      -u ：列出这个装置的 UUID
      -L ：设定这个装置的 Label name
      -U ：设定这个装置的 UUID 喔！
      ```

    - tune2fs 修改 ext4 的 label name 与 UUID

      ```
      [root@study ~]#  tune2fs [- - l] [- - L Label] [- - U uuid]  装 置文件名
      选项与参数：
      -l ：类似 dumpe2fs -h 的功能～将 superblock 内的数据读出来～
      -L ：修改 LABEL name
      -U ：修改 UUID 啰！
      范例：列出 /dev/vda5 的 label name 之后，将它改成 vbird_ext4
      [root@study ~]#  dumpe2fs - - h /dev/vda5 | grep name
      dumpe2fs 1.42.9 (28-Dec-2013)
      ```

  - 设定开机挂载

    - 系统挂载的一些限制

      - 根目录 / 是必须挂载的﹐而且一定要先于其它 mount point 被挂载进来。
      - 其它 mount point 必须为已建立的目录﹐可任意指定﹐但一定要遵守必须的系统目录架构原则 (FHS)
      - 所有 mount point 在同一时间之内﹐只能挂载一次。
      - 所有 partition 在同一时间之内﹐只能挂载一次。
      - 如若进行卸除﹐您必须先将工作目录移到 mount point(及其子目录) 之外

    - 查看

      - `cat /etc/fstab`
      - 描述信息如下:

      ```
      [装置/UUID 等] [挂载点] [文件系统] [文件系统参数] [dump] [fsck]
      ```

      - 解释
        - 第一栏：磁盘装置文件名/UUID/LABEL name：
          - 文件系统或磁盘的装置文件名，如 /dev/vda2 等
          - 文件系统的 UUID 名称，如 UUID=xxx
          - 文件系统的 LABEL 名称，例如 LABEL=xxx
        - 第二栏：挂载点 (mount point)：：
        - 第三栏：磁盘分区槽的文件系统：
        - 第四栏：文件系统参数：
        - 第五栏：能否被 dump 备份指令作用：
        - 第六栏：是否以 fsck 检验扇区：

- 特殊装置 loop 挂载

  - 挂载光盘/DVD 映象文件
  - 大型文件的格式化
  - 挂载

- 内存置换空间(swap)

  - 使用实体 分区置 槽建置 swap
    - 基本步骤
      1. 分区：先使用 gdisk 在你的磁盘中分区出一个分区槽给系统作为 swap 。由于 Linux 的 gdisk 预设会将分 区槽的 ID 设定为 Linux 的文件系统，所以你可能还得要设定一下 system ID 就是了。
      2. 格式化：利用建立 swap 格式的『mkswap 装置文件名』就能够格式化该分区槽成为 swap 格式啰
      3. 使用：最后将该 swap 装置启动，方法为：『swapon 装置文件名』。
      4. 观察：最终透过 free 与 swapon -s 这个指令来观察一下内存的用量吧！

- 使用 文件置 建置 swap

  - 透过一个案例学习
    1. 使用 dd 这个指令来新增一个 128MB 的文件在 /tmp 底下：
    2. 使用 mkswap 将 /tmp/swap 这个文件格式化为 swap 的文件格式
    3. 使用 swapon 来将 /tmp/swap 启动啰！
    4. 使用 swapoff 关掉 swap file，并设定自动启用

- 文件系统的特殊观察与操作

  - 磁盘空间之浪费问题
  - 利用 GNU 的 parted 进行 分区 行为(Optional)
  - **parted 可以直接在一行指令列就完成分区，是一个非常好用的指令！它常用的语法如下：**

  ```
  [root@study ~]#  parted [ 装 置 ] [ 指令 [ [ 参数 ]]
  选项与参数：
  指令功能：
  新增分区：mkpart [primary|logical|extended] [ext4|vfat|xfs] 开始 结束
  显示分区：print
  删除分区：rm [partition]
  范例一：以 parted 列出目前本机的分区表资料
  [root@study ~]#  parted /dev/vda  print
  Model: Virtio Block Device (virtblk) <==磁盘接口与型号
  Disk /dev/vda: 42.9GB <==磁盘文件名与容量
  Sector size (logical/physical): 512B/512B <==每个扇区的大小
  Partition Table: gpt <==是 GPT 还是 MBR 分区
  Disk Flags: pmbr_boot
  Number Start End Size File system Name Flags
  1 1049kB 3146kB 2097kB bios_grub
  2 3146kB 1077MB 1074MB xfs
  3 1077MB 33.3GB 32.2GB lvm
  4 33.3GB 34.4GB 1074MB xfs Linux filesystem
  5 34.4GB 35.4GB 1074MB ext4 Microsoft basic data
  6 35.4GB 36.0GB 537MB linux-swap(v1) Linux swap
  [ 1 ] [ 2 ] [ 3 ] [ 4 ] [ 5 ] [ 6 ]
  ```

  - 显示参数的意义

  ```
  1. Number：这个就是分区槽的号码啦！举例来说，1 号代表的是 /dev/vda1 的意思；
  2. Start：分区的起始位置在这颗磁盘的多少 MB 处？有趣吧！他以容量作为单位喔！
  3. End：此分区的结束位置在这颗磁盘的多少 MB 处？
  4. Size：由上述两者的分析，得到这个分区槽有多少容量；
  5. File system：分析可能的文件系统类型为何的意思！
  6. Name：就如同 gdisk 的 System ID 之意。
  ```

  - 案例
    - 使用parted 将 mbr 分区表改成 gpt
      - 非常危险，无法还原

## 第八章 文件和文件系统的压缩

- 压缩文件的用途和技术

- Linux 常见的压缩命令

  - gzip ， zcat 、zmore、zless、zgrep

    - 语法

      - `gzip [-cdtv] 文件名`
      - `zcat 文件名.gz`

      ```
      选项与参数：
      -c ：将压缩的数据输出到屏幕上，可透过数据流重导向来处理；
      -d ：解压缩的参数；
      -t ：可以用来检验一个压缩文件的一致性～看看文件有无错误；
      -v ：可以显示出原文件/压缩文件案的压缩比等信息；
      -# ：# 为数字的意思，代表压缩等级，-1 最快，但是压缩比最差、-9 最慢，但是压缩比最好！预设是 -6
      ```

    - `egrep` 查找

  - bzip2, bzcat/bzmore/bzless/bzgrep

  - 对于gzip 的升级

  - 优点

    - 比gzip 压缩率更好

  - 缺点

    - 时间要更久

  - xz, xzcat/xzmore/xzless/xzgrep

    - 优点
      - 比bzip2 压缩率还要好
    - 缺点
      - 很慢

- 打包命令（常用）

  - 选项和参数

  ```
  选项与参数：
  -c ：建立打包文件，可搭配 -v 来察看过程中被打包的档名(filename)
  -t ：察看打包文件的内容含有哪些档名，重点在察看『档名』就是了；
  -x ：解打包或解压缩的功能，可以搭配 -C (大写) 在特定目录解开
  特别留意的是， -c, -t, -x 不可同时出现在一串指令列中。
  -z ：透过 gzip 的支持进行压缩/解压缩：此时档名最好为 *.tar.gz
  -j ：透过 bzip2 的支持进行压缩/解压缩：此时档名最好为 *.tar.bz2
  -J ：透过 xz 的支持进行压缩/解压缩：此时档名最好为 *.tar.xz
  特别留意， -z, -j, -J 不可以同时出现在一串指令列中
  -v ：在压缩/解压缩的过程中，将正在处理的文件名显示出来！
  -f filename：-f 后面要立刻接要被处理的档名！建议 -f 单独写一个选项啰！(比较不会忘记)
  -C 目录 ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
  其他后续练习会使用到的选项介绍：
  -p(小写) ：保留备份数据的原本权限与属性，常用于备份(-c)重要的配置文件
  -P(大写) ：保留绝对路径，亦即允许备份数据中含有根目录存在之意；
  --exclude=FILE：在压缩的过程中，不要将 FILE 打包！
  ```

  - 常用如下命令

    - 压缩`tar -jcv -f filename.tar.bz2 文件或者目录名称`
    - 查询 `tar -jtv filename.tar.bz2`
    - 解压缩 `tar -jxv -f filename.tar.bz2 要解压的地方`

  - 案例

    - 压缩 /etc 下面的文件，查看优点特殊的地方
    - 为什么要去掉 / 根目录路径？
    - 如果要tar 里面加上这个根路径，要怎么做？
    - 备份数据解压缩，并且到特定的目录下面
      - 如何指定要解开的目录

  - 只解开单一文件的办法

    - 先查找文档名称
      - `tar - - jtv - - f /root/etc.tar.bz2 | grep 'shadow'`

  - 打包某目录，但不含该目录下的某些 文件

    - --exclude 参数用法

  - 仅备份比某个时刻还要新的 文件

    - 先使用搜索找到对应的文件
    - `tar -jcv -f /root/etc.newer.then.passwd.tar.bz2 --newer-mtime="2015/06/07" /etc/*`

  - 利用管道一遍打包一遍备份

    - ```
      tar -cvf - /etc | tar -xvf -
      ```

      - 注意 - 代表输入和输出

  - 例题：系统备份的案例

    - 结合上面所学。使用过滤方式备份数据

  - 解压缩后的 SELinux

    - 可能影响系统配置

- XFS 文件系统的备份与还原

  - 使用备份命令 xfs_dump

    - 备份行为可以累积，类似SVN的手法，新建备份只新增差异文件
    - 限制
      - 文件系统必须挂载
      - xfsdump 必须使用 root 的权限才能操作 (涉及文件系统的关系)
      - xfsdump 只能备份 XFS 文件系统啊！
      - xfsdump 备份下来的数据 (文件或储存媒体) 只能让 xfsrestore 解析
      - xfsdump 是透过文件系统的 **UUID** 来分辨各个备份档的，因此不能备份两个具有相同 UUID 的文件系统 喔！
      - **xfsdump 预设仅支持文件系统的备份，并不支持特定目录的备份**

  - 使用方法

    - `xfsdump [- - L S_label] [- - M M_label] [- - l #] [- - f 备 份 档 ] 待 备 份 资`
    - 选项和参数

    ```
    选项与参数：
    -L ：xfsdump 会纪录每次备份的 session 标头，这里可以填写针对此文件系统的简易说明
    -M ：xfsdump 可以纪录储存媒体的标头，这里可以填写此媒体的简易说明
    -l ：是 L 的小写，就是指定等级～有 0~9 共 10 个等级喔！ (预设为 0，即完整备份)
    -f ：有点类似 tar 啦！后面接产生的文件，亦可接例如 /dev/st0 装置文件名或其他一般文件档名等
    -I ：从 /var/lib/xfsdump/inventory 列出目前备份的信息状态
    ```

    - 案例
    - 累积备份

- XFS 文件系统还原 xfsrestore

  - 占位，等安装对应系统再来

- 用 xfsrestore 观察 xfsdump

  - 简单复原 level 0 的文件系统
  - 复原累积备份资料
  - 仅还原部分 文件的 的 xfsrestore 互动模式

- 光盘写入工具

  - 了解即可

- 其他常见的压缩和备份工具

  - dd

    - 作用
      - dd 可以读取磁盘装置的内容，然后将整个装置备份成一个文件呢
    - 使用方法
      - `dd if="input_file" of="output_file" bs="block_size" count="number"`
    - 选项和参数

    ```
    选项与参数：
    if ：就是 input file 啰～也可以是装置喔！
    of ：就是 output file 喔～也可以是装置；
    bs ：规划的一个 block 的大小，若未指定则预设是 512 bytes(一个 sector 的大小)
    count：多少个 bs 的意思。
    ```

    - 案例
      - 范例一：将 /etc/passwd 备份到 /tmp/passwd.back 当中
        - `dd if=/etc/passwd of=/tmp/passwd.back`
      - 范例二：将刚刚刻录的光驱的内容，再次的备份下来成为映像挡
      - 范例三：假设你的 USB 是 /dev/sda 好了，请将刚刚范例二的 image 刻录到 USB 磁盘中
      - 范例四：将你的 /boot 整个文件系统透过 dd 备份下来
    - 例题：

    ```
    你想要将你的 /dev/vda2 进行完整的复制到另一个 partition 上，请使用你的系统上面未分区完毕的容量再建立一
    个与 /dev/vda2 差不多大小的分区槽 (只能比 /dev/vda2 大，不能比他小！)，然后将之进行完整的复制 (包括需
    要复制 boot sector 的区块)。
    答：
    因为我们的 /dev/sda 也是个测试的 USB 磁盘，可以随意恶搞！我们刚刚也才测试过将光盘映像文件给它复制进
    去而已。 现在，请你分区 /dev/sda1 出来，然后将 /dev/vda2 完整的拷贝进去 /dev/sda1 吧！
    ```

    - 注意点
      - 需要注意dd 连同 uuid 一起复制，如果要复制两块完全相同的硬盘，需要重新定义一下 uuid

- cpio

  - 作用
    - cpio 可以备份任何东西，包括装置设备文件
    - 暂时只记住语法即可
  - 缺点
    - 不会主动去找文件，需要find 找到文件才能备份
  - 语法

  ```
  [root@study ~]#  cpio - - ovcB > [file|device] <==备份
  [root@study ~]#  cpio - - ivcdu < [file|device] <==还原
  [root@study ~]#  cpio - - ivct < [file|device] <==察看
  备份会使用到的选项与参数：
  -o ：将数据 copy 输出到文件或装置上
  -B ：让预设的 Blocks 可以增加至 5120 bytes ，预设是 512 bytes ！
  这样的好处是可以让大文件的储存速度加快(请参考 i-nodes 的观念)
  还原会使用到的选项与参数：
  -i ：将数据自文件或装置 copy 出来系统当中
  -d ：自动建立目录！使用 cpio 所备份的数据内容不见得会在同一层目录中，因此我们
  必须要让 cpio 在还原时可以建立新目录，此时就得要 -d 选项的帮助！
  -u ：自动的将较新的文件覆盖较旧的文件！
  -t ：需配合 -i 选项，可用在"察看"以 cpio 建立的文件或装置的内容
  一些可共享的选项与参数：
  -v ：让储存的过程中文件名可以在屏幕上显示
  -c ：一种较新的 portable format 方式储存
  ```

### 练习题

- 情境模拟题一：请将本章练习过程中产生的不必要的文件删除，以保持系统容量不要被恶搞！

- 情境模拟题二：你想要逐时备份 /home 这个目录内的数据，又担心每次备份的信息太多， 因此想要使用 xfsdump 的方式来逐一备份数据到 /backups 这个目录下。该如何处理？ o 目标：了解到 xfsdump 以及各个不同 level 的作用； o 前提：被备份的资料为单一 partition ，亦即本例中的 /home 实际处理的方法其实还挺简单的！我们可以这样做看看：

- 情境模拟三：假设过了一段时间后，妳的 /home 变的怪怪的，妳想要将该 filesystem 以刚刚的备份数据还 原， 此时该如何处理呢？妳可以这样做的：

  ​	由于 /home 这个 partition 是用户只要有登入就会使用，因此你应该无法卸除这个东西！因此，你必 须要注销所有一般用户， 然后在 tty2 直接以 root 登入系统，不要使用一般账号来登入后 su 转成 root ！ 这样才有办法卸除 /home 喔！

## 第九章 Vim编辑器

- vi 和 vim

- 为什么要学习vi 和vim

  - 内置命令会使用
  - unix 基本都会自带
  - 方便简单
  - 功能强大

- vi使用方法

  - 三种模式
    - 一般命令模式
      - 
    - 编辑模式
    - 命令行模式

- 注意

  - 一般指令模式可与编辑模式及指令列模式切换， 但编辑模式与指令列 模式之间不可互相切换喔

- 简单案例

  - 使用『 vi filename 』进入一般指令模式
  - 使用 i 进入编辑模式
  - 使用esc 退出到一般命令模式
  - `:wq` 进行保存

- 常用操作

  - 一般模式

    - hjkl 可以进行上下左右的移动
    - `ctrl + f` 和`ctrl + b` 上下移动一页
    - / 查找 继续按下n继续查找
    - 0 或者 home
    - $ 或者 end
    - gg 跳到第一行
    - n(数字) enter 光标下移 N 行
    - :n1,n2s/word1/word2/g
      - n1 与 n2 为数字。在第 n1 与 n2 列之间寻找 word1 这个字符串，并将该字符串取代 为 word2 ！举例来说，在 100 到 200 列之间搜寻 vbird 并取代为 VBIRD 则： 『:100,200s/vbird/VBIRD/g』。(常用)
    - :1,$s/word1/word2/gc
      - 从第一列到最后一列寻找 word1 字符串，并将该字符串取代为 word2 ！且在取代前显 示提示字符给用户确认 (confirm) 是否需要取代！(常用)
    - :1,$s/word1/word2/g
      - 从第一列到最后一列寻找 word1 字符串，并将该字符串取代为 word2 ！(常用)
    - x 退出但是不改变时间
    - dd 删除一行
    - yy 复制一行
    - nyy 复制多行 如3yy
    - u 恢复前一个操作
    - ctrl + r 重做上一个动作
    - . 重复上一个动作

  - 编辑模式

    - i I 插入模式
      - i 为『从目前光标所在处插入』
      - I 为『在目前所在列的第一个非空格符处开始插入】
    - a 和 A
      - a 为『从目前光标所在的下一个字符处开始插入』
      - A 为『从光标所在列的最后一个字符处开始插入』
    - o 和 O
    - r 和 R

  - 命令模式

    - w
    - q
    - !

  - 实际案例

    ```
    1. 请在 /tmp 这个目录下建立一个名为 vitest 的目录；
    2. 进入 vitest 这个目录当中；
    3. 将 /etc/man_db.conf 复制到本目录底下(或由上述的连结下载 man_db.conf 文件)；
    4. 使用 vi 开启本目录下的 man_db.conf 这个文件；
    5. 在 vi 中设定一下行号；
    6. 移动到第 43 列，向右移动 59 个字符，请问你看到的小括号内是哪个文字？
    7. 移动到第一列，并且向下搜寻一下『 gzip 』这个字符串，请问他在第几列？
    8. 接着下来，我要将 29 到 41 列之间的『小写 man 字符串』改为『大写 MAN 字符串』，并且一个一个挑
    选是否需要修改，如何下达指令？如果在挑选过程中一直按『y』， 结果会在最后一列出现改变了几个 man
    呢？
    9. 修改完之后，突然反悔了，要全部复原，有哪些方法？
    10. 我要复制 66 到 71 这 6 列的内容(含有 MANDB_MAP)，并且贴到最后一列之后；
    11. 113 到 128 列之间的开头为 # 符号的批注数据我不要了，要如何删除？
    12. 将这个文件另存成一个 man.test.config 的檔名；
    13. 去到第 25 列，并且删除 15 个字符，结果出现的第一个单字是什么？
    14. 在第一列新增一列，该列内容输入『I am a student...』；
    15. 储存后离开吧！
    ```

  - vim 缓存处理办法

    - 问题
      - 问题一：可能有其他人或程序同时在编辑这个文件：
      - 问题二：在前一个 vim 的环境中，可能因为某些不知名原因导致 vim 中断 (crashed)：

- vim 的额外功能

- 可视区块

  - v 开始反白
  - V 行反白
  - y 反白复制
  - d 反白删除

- 多文件编辑

  - :n 编辑下一个文件
  - :N 编辑上一个文件
  - :files 列出目前这个vim 开启的所有文件
  - 案例
    - 多文件编辑

- 多窗口功能

  - :sp {filename}
    - 存在文件名对比文件，
    - 否则新建一个窗口用于对比
  - 利用 ctrl + w + 上 或者 ctrl + w + 下 来切换窗口

- vim 关键词补全

  - 快捷键使用
    - [ctrl]+x -> [ctrl]+n 透过目前正在编辑的这个『文件的内容文字』作为关键词，予以补齐
    - [ctrl]+x -> [ctrl]+f 以当前目录内的『文件名』作为关键词，予以补齐
    - [ctrl]+x -> [ctrl]+o 以扩展名作为语法补充，以 vim 内建的关键词，予以补齐

- vim 环境设定与记录： ~/.vimrc, ~/.viminfo

- vim 常用图

  - 查看图片

- 其他vim 注意事项

  - 中文乱码
  - dos 和 linux 换行
    - dos2nnix
    - unix2dos

- 语系编码转换

  - `iconv -f big5 -t utf8 vi.big5 -o vi.utf8`
  - `iconv -f utf8 -t big5 vi utf8 | \`

### 练习题

- 在第七章的情境模拟题二的第五点，编写 /etc/fstab 时，当时使用 nano 这个指令， 请尝试使用 vim 去编 辑 /etc/fstab ，并且将第七章新增的那一列的 defatuls 改成 default ，会出现什么状态？ 离开前请务必要 修订成原本正确的信息。此外，如果将该列批注 (最前面加 #)，你会发现字体颜色也有变化喔！
- 尝试在你的系统中，你惯常使用的那个账号的家目录下，将本章介绍的 vimrc 内容进行一些常用设定，包 括： o 设定搜寻高亮度反白 o 设定语法检验启动 o 设定默认启动行号显示 o 设定有两行状态栏 (一行状态+一行指令列) :set laststatus=2

### 简答题

# 第十章 认识 BASH 这个 Shell

- 简单认识shell

  - 什么是shell?
    - 广义：就是指可以调动内核和硬件打交道的一个壳程序
    - 狭义：通过内核指挥硬件工作的命令程序
  - 为什么要学习shell？
    - 防止突发情况
    - 个人提升
    - 公司要求
    - 某些特殊情况只能使用shell去解决
  - 不学习shell会怎么样
    - 使用 x windows
    - 使用第三方应用程序
  - 如何学习shell
    - 多敲
    - 多练
    - 多模拟场景
      - 定期清理过期日志
      - 开启启动某些应用程序
      - 磁盘预警

- 系统合法的shell 和 /etc/shells 功能

  - cshell 和 bshell (bash)
  - 检查/etc/shell 可以得到系统可使用的shell
    - /bin/sh
    - /bin/bash 默认
    - /bin/tcsh
    - /bin/csh

- bash的功能

  - history 历史功能
    - 记录的位置：家目录的 .bash_history
      - 注意：按照登陆来记录，记录上次登陆的命令，本次的被存在缓存
  - 补全功能
    - 确定命令正确
  - 命令别名
    - alias
      - 简单用：`alias lm ='ls -al'`
  - 通配符

- 判断命令是否为 Bash shell 内置 ： `type`

  - `man bash`
  - type 基本使用
    - `type -t umask`
  - 命令快捷键：常用
    - ctrl + u 和 ctrl + k
    - ctrl + a 和 ctrl + e

- shell 基本内容学习

  - 变量定义
    - echo 和 unset
      - 案例:
        - `echo $HOME`
        - 设置：变量 `echo ${myname}`
      - 特殊点
        - echo 默认变量为“空”
    - 设置规则
      - 双引号和单引号的区别
    - 案例
      - 如何进入到内核的目录模块
        - `cd /lib/moudules/$(name -r)/kernel`
  - 变量取消
    - unset name

- 环境变量设置

  - env 查看环境变量

  - declare 定义变量的另一种方式，声明变量

    - declare -i number=$RANDOM*10/32768 ; echo $number
      - 随机生成一个数字

  - set 观察和设置自定义变量

  - PS1 提示字符

    - 有许多的定义，具体可以查看对应的百度内容

    - https://blog.csdn.net/qq_34208467/article/details/81019467

    - 参数意义

      ```
        \d ：可显示出『星期 月 日』的日期格式，如："Mon Feb 2"
        \H ：完整的主机名。举例来说，鸟哥的练习机为『study.centos.vbird』
        \h ：仅取主机名在第一个小数点之前的名字，如鸟哥主机则为『study』后面省略
        \t ：显示时间，为 24 小时格式的『HH:MM:SS』
        \T ：显示时间，为 12 小时格式的『HH:MM:SS』
        \A ：显示时间，为 24 小时格式的『HH:MM』
        \@ ：显示时间，为 12 小时格式的『am/pm』样式
        \u ：目前使用者的账号名称，如『dmtsai』；
        \v ：BASH 的版本信息，如鸟哥的测试主机版本为 4.2.46(1)-release，仅取『4.2』显示
        \w ：完整的工作目录名称，由根目录写起的目录名称。但家目录会以 ~ 取代；
        \W ：利用 basename 函数取得工作目录名称，所以仅会列出最后一个目录名。
        \# ：下达的第几个指令。
        \$ ：提示字符，如果是 root 时，提示字符为 # ，否则就是 $ 啰～
      ```

- $

  - 查看PID :``

- ?

  - 代表着上一个命令的返回值
  - 如何理解？
    - 一般命令执行之后会有一个0和非0的值，如果是0代表执行成功
    - 非0则代表失败

- export 自定义环境变量

  - env 和 set有什么区别？
    - env 是父进程环境变量
    - set 可以是子进程变量
    - **子进程仅仅继承父进程的环境变量，不继承自定义变量**
  - 使用export 可以让自定义变量变成环境变量

- declare 环境变量转自定义变量

- 语系变量

  - locale -a 查看系统支持的所有语言

- 变量有效范围

  - 变量键盘读取和数组声明：read 、array、declare
  - read
    - 作用
      - 等待键盘输入
    - 参数
      - -p 提示字符
      - -t 等待秒数
  - declare ， typeset
    - 作用
      - 声明变量类型
        - -a 定义数组类型
        - -i 定义整数类型
        - -e export 相同
        - -r readonly类型
    - 注意：
      - 默认字符串类型
      - 默认最多为整型运算

- 和文件系统程序有关的限制关系：ulimit

  - 作用
    - 限制开启文件数量，可使用CPU时间
    - 可使用内存总量
  - **参数和语法：**

  ```
  选项与参数：
  -H ：hard limit ，严格的设定，必定不能超过这个设定的数值；
  -S ：soft limit ，警告的设定，可以超过这个设定值，但是若超过则有警告讯息。
  在设定上，通常 soft 会比 hard 小，举例来说，soft 可设定为 80 而 hard
  设定为 100，那么你可以使用到 90 (因为没有超过 100)，但介于 80~100 之间时，
  系统会有警告讯息通知你！
  -a ：后面不接任何选项与参数，可列出所有的限制额度；
  -c ：当某些程序发生错误时，系统可能会将该程序在内存中的信息写成文件(除错用)，
  这种文件就被称为核心文件(core file)。此为限制每个核心文件的最大容量。
  -f ：此 shell 可以建立的最大文件容量(一般可能设定为 2GB)单位为 Kbytes
  -d ：程序可使用的最大断裂内存(segment)容量；
  -l ：可用于锁定 (lock) 的内存量
  -t ：可使用的最大 CPU 时间 (单位为秒)
  -u ：单一用户可以使用的最大程序(process)数量
  ```

  - 如何恢复？
    - 注销再登陆

- 变量的删除、取代和替换

  - 变量删除替换

    - 案例：

    ```
    ${variable#/*local/bin:} }
    上面的特殊字体部分是关键词！用在这种删除模式所必须存在的
    
    ${ variable#/*local/bin:}
    这就是原本的变量名称，以上面范例二来说，这里就填写 path 这个『变量名称』啦！
    
    ${variable# #/*local/bin:}
    # 号的作用
    这是重点！代表『从变量内容的最前面开始向右删除』，
    且仅删除最短的那个
    
    ${variable# /*local/bin:}
    代表要被删除的部分，由于 # 代表由前面开始删除，所以这里便由开始的 / 写起。
    需要注意的是，我们还可以透过通配符 * 来取代 0 到无穷多个任意字符
    以上面范例二的结果来看， path 这个变量被删除的内容如下所示：
    /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/dmtsai/.local/bin:/home/dmtsai/bin
    ```

    - 简单理解

    ```
    ${variable# #/*local/bin:}
    # 号的作用
    这是重点！代表『从变量内容的最前面开始向右删除』，
    且仅删除最短的那个 
    ```

    - \# 和 ## 代表如下
      - \# 符合替换字符的【最短的】哪一个
      - \## 符合替换字符最长的哪一个
    - 如何从后面向前面删除内容
      - % 和 %%

  - 替换

    - 案例

    ```
    范例六：将 path 的变量内容内的 sbin 取代成大写 SBIN：
    [dmtsai@study ~]$  echo ${path/sbin/SBIN}
    ```

  - 变量测试和内容替换

    - 案例

    ```
    范例一：测试一下是否存在 username 这个变量，若不存在则给予 username 内容为 root
    [dmtsai@study ~]$  echo ${username}
    <==由于出现空白，所以 username 可能不存在，也可能是空字符串
    [dmtsai@study ~]$  username=${username-root}
    ```

    - 关键点在于 - 号码
    - 后面接变量不存在的替换字符
    - 如果是空字符串也想替换要怎么做
      - :- 前面加上 : 冒号即可

  - 提示变量不存在 ?

    - 案例

      ```
      测试：若 str 不存在时，则 var 的测试结果直接显示 "无此变量"
      [dmtsai@study ~]$  unset str; var=${str? 无 此 变数} }
      -bash: str: 无此变量 <==因为 str 不存在，所以输出错误讯息
      ```

- 命名别名和历史命令

  - alias 和 unalias
  - history
    - 自行man 查询使用方式

- bash shell 的操作环境

  - 路径和命令的查找顺序（面试终点）
    - 根据绝对和相对路径查找的命令
    - 由于alias 查找出来的命令
    - bash 内置的 命令来执行
    - 根据$PATH 环境变量来查找

- 登陆的欢迎信息

  - `cat /etc/issue`
  - cat /etc/issue.net 用于ssh等远程登陆使用的欢迎信息
  - cat /etc/motd 用于所有登陆用户想要知道的某些信息

- bash 的环境配置文件

  - login 和 non-login shell
    - login shell 取得bash 的时候需要的完整登陆流程
    - non-login shell: 取得bash 的方法不需要重复登陆的操作
  - 为什么需要最先了解这两个东西
    - **因为不同的登陆形式获取的shell配置不一样**

- login shell读取的内容

  - /etc/profile 系统整体配置，很脆弱，一旦奔溃恢复比较麻烦
    - /etc/profile 的主要内容
      - PATH
      - MAIL
      - USER
      - HOSTNAME
      - HISTSIZE
      - umask
      - 调用外部文件
        - /etc/profile.d/*.sh
        - /etc/locale.conf
        - /usr/share/bash-completion/completions/*
  - ~/.bash_profile 或者 ~/.bash_login 或者 ~/.profile 用户个人的配置文件

- 读取配置文件的命令 source

  - 作用
    - 当案例需要多个变量环境的时候，很方便的切换变量环境
    - 安装某些软件的时候快速改变环境变量测试是否设置正确
  - 注意点：
    - /etc/bashrc (red hat 系统特有)
      - 作用
        - 根据不同的UID 设置umask
        - 根据不同UID 设置提示字符
        - 调用 /etc/profile.d/*.sh

- 终端环境设置 stty 、 set

  - https://linux.die.net/man/1/stty
  - https://www.computerhope.com/unix/ustty.htm