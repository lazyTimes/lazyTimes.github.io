---
title: 【Linux】What Is dev shm And Its Practical Usage
subtitle: /dev/shm只不过是传统共享内存 概念的实现
author: lazytime
url_suffix: devshmread
tags:
  - Linux
categories:
  - Linux
keywords: /dev/shm特殊
description: /dev/shm使用和实现
copyright: true
date: 2023-07-27 11:21:58
---

# 原文

[What Is /dev/shm And Its Practical Usage - nixCraft (cyberciti.biz)](https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html)



<!-- more -->

# 介绍

**/dev/shm** is nothing but implementation of traditional **shared memory** concept. 

**/dev/shm**只不过是传统**共享内存**概念的实现。

It is an efficient means of passing data between programs.

它是一种在程序间传递数据的有效方法。

One program will create a memory portion, which other processes (if permitted) can access.

一个程序将创建一个内存部分，其他进程（如果允许）可以访问该部分。

This will result into speeding up things on Linux.  

这将（意味着）加快 Linux 的运行速度。 

shm / shmfs is also known as tmpfs, which is a common name for a temporary file storage facility on many Unix-like operating systems. 

`shm / shmfs` 也称为 `tmpfs`，是许多类 `Unix` 操作系统中临时文件存储设备的通用名称。

It is intended to appear as a mounted file system, but one which uses virtual memory instead of a persistent storage device.

`tmpfs` 的内容是显示已加载的文件系统、但是**使用的是用户虚拟内存而不是一个永久存储设备**。

If you type the mount command you will see /dev/shm as a tempfs file system. 

如果键入挂载命令，就会看到 `/dev/shm` 作为 `tempfs` 文件系统。

Therefore, it is a file system, which keeps all files in virtual memory. 

因此，（`/dev/shm`）**它是一个将所有文件保存在虚拟内存中的文件系统**。

Everything in tmpfs is temporary in the sense that no files will be created on your hard drive.

注意，所有在`tmpfs`中发生的事情都是临时的，不会在硬盘上创建任何文件。

If you unmount a tmpfs instance, everything stored therein is lost. 

**如果你卸载一个tmpfs 实例，它内部的所有存储内容都将丢失。**

By default almost all Linux distros configured to use /dev/shm:  

默认情况下，几乎所有 Linux 发行版都配置使用` /dev/shm`：

比如你使用下面的命令。

```sh
$ df -h
```

Sample outputs:

输出示例

```
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/wks01-root
                      444G   70G  351G  17% /
tmpfs                 3.9G     0  3.9G   0% /lib/init/rw
udev                  3.9G  332K  3.9G   1% /dev
tmpfs                 3.9G  168K  3.9G   1% /dev/shm
/dev/sda1             228M   32M  184M  15% /boot
```

## Nevertheless, where can I use /dev/shm? 但是，在哪里可以使用 /dev/shm？

You can use /dev/shm to improve the performance of application software such as Oracle or overall Linux system performance.

您可以使用 /dev/shm 来提高 Oracle 等应用软件的性能或 Linux 系统的整体性能。

On heavily loaded system, it can make tons of difference. 

在负载较重的系统中，它可以产生巨大的差异。

For example VMware workstation/server can be optimized to improve your Linux host’s performance (i.e. improve the performance of your virtual machines).

例如，可以对 VMware 工作站/服务器进行优化，以提高 Linux 主机的性能（即提高虚拟机的性能）。

In this example, remount /dev/shm with 8G size as follows:

比如下面这个例子，重新挂载 8G 大小的 /dev/shm，如下所示：

```sh
mount -o remount,size=8G /dev/shm
```

To be frank, if you have more than 2GB RAM + multiple Virtual machines, this hack always improves performance. 

坦率地说，**如果你有超过 2GB 内存和多个虚拟机，这个黑客程序总能提高性能。**

In this example, you will give you tmpfs instance on /disk2/tmpfs which can allocate 5GB RAM/SWAP in 5K inodes and it is only accessible by root:

在本例中，你将在 `/disk2/tmpfs` 上创建一个 tmpfs 实例，它可以在 5K inodes 中分配 5GB RAM/SWAP，并且只能由 root 访问：

```sh
mount -t tmpfs -o size=5G,nr_inodes=5k,mode=700 tmpfs /disk2/tmpfs
```

Where,

命令解释：

- **-o opt1,opt2** : Pass various options with a -o flag followed by a comma separated string of options. In this examples, I used the following options:

- **-o opt1,opt2** ： 使用 -o 标志传递各种选项，后跟逗号分隔的选项字符串。在本例中，我使用了以下选项：

	- **remount** : Attempt to remount an already-mounted filesystem. In this example, remount the system and increase its size.

	- **remount** : 尝试重新挂载已挂载的文件系统。在此示例中，重新挂载系统并增大其大小。
	
	- **size=8G or size=5G** : Override default maximum size of the /dev/shm filesystem. he size is given in bytes, and rounded up to entire pages. 
	  覆盖 /dev/shm 文件系统的默认最大大小。大小以字节为单位，四舍五入为整页。
	  
	  The default is half of the memory. 
	    默认值为内存的一半。
	  
	  The size parameter also accepts a suffix % to limit this tmpfs instance to that percentage of your pysical RAM: the default, when neither size nor nr_blocks is specified, is size=50%. 
	  size 参数也接受后缀 %，用于将此 tmpfs 实例限制为实际内存的该百分比：**当既未指定 size 也未指定 nr_blocks 时，默认值为 size=50%**。
	  
	  In this example it is set to 8GiB or 5GiB. The tmpfs mount options for sizing ( size, nr_blocks, and nr_inodes) accept a suffix k, m or g for Ki, Mi, Gi (binary kilo, mega and giga) and can be changed on remount.
	  在本例中，它被设置为 8GiB 或 5GiB。tmpfs 挂载选项的大小（size、nr_block 和 nr_inodes）接受 Ki、Mi、Gi（二进制千、兆和千兆）的后缀 k、m 或 g，并可在重新挂载时更改。
	  
	- **nr_inodes=5k** : 
	- The maximum number of inodes for this instance. The default is half of the number of your physical RAM pages, or (on a machine with highmem) the number of lowmem RAM pages, whichever is the lower.
	  此实例的最大 inodes 数量。默认值是物理 RAM 页数的一半，或（在使用高内存的机器上）低内存 RAM 页数的一半，以较低者为准。
	  
	- **mode=700** : Set initial permissions of the root directory.
	  设置根目录的初始权限。
	  
	- **tmpfs** : Tmpfs is a file system which keeps all files in virtual memory.
	  Tmpfs 是一种将所有文件保存在虚拟内存中的文件系统。

## How do I restrict or modify size of /dev/shm permanently?如何永久限制或修改 /dev/shm 的大小？

You need to add or modify entry in /etc/fstab file so that system can read it after the reboot. Edit, /etc/fstab as a root user, enter:  

您需要添加或修改 `/etc/fstab` 文件中的配置，以便系统能在重启后读取它。以根用户身份编辑 `/etc/fstab`，输入  

```sh
vi /etc/fstab
```

Append or modify `/dev/shm` entry as follows to set size to 8G

添加或修改 `/dev/shm` 条目如下，将大小设置为 8G

```sh
none      /dev/shm        tmpfs   defaults,size=8G        0 0
```

Save and close the file. For the changes to take effect immediately remount /dev/shm:  

保存并关闭文件。要使更改立即生效，请重新挂载 `/dev/shm`：  

```sh
mount -o remount /dev/shm
```

Verify the same:  

验证是否一致：

```sh
df -h
```

# Recommend readings:

# 推荐读物：

- See man pages of mount regarding tmpfs options.
- Details regarding tmpfs is available in [/usr/share/doc/kernel-doc-/Documentation/filesystems/tmpfs.txt](https://www.cyberciti.biz/files/linux-kernel/Documentation/filesystems/tmpfs.txt) file.

- 有关 tmpfs 选项，请参阅 mount 的手册。
- 有关 tmpfs 的详细信息，请参阅 [/usr/share/doc/kernel-doc-/Documentation/filesystems/tmpfs.txt](https://www.cyberciti.biz/files/linux-kernel/Documentation/filesystems/tmpfs.txt) 文件。
