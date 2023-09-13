---
title: 【Linux】Linux Zero-Copy Using sendfile
subtitle: 零拷贝
author: lazytime
url_suffix: zerocopy
tags:
  - Linux
categories:
  - Linux
keywords: linux,zerocopy
description: 零拷贝
copyright: true
date: 2023-09-13 14:08:23
---
# Source

Source：[Linux Zero-Copy Using sendfile(). sendfile() has been gradually becoming… | by CocCoc Techblog | The Startup | Medium](https://medium.com/swlh/linux-zero-copy-using-sendfile-75d2eb56b39b)
# Why Zero-copy?

What’s happening under the hood when the OS is copying a file / transfering a file to another host? 

当操作系统将文件复制/传输到另一台主机时，在系统底层下发生了什么？

For our naked eyes the process can be simple, OS first reads content of the file, then writes it to another file, then it’s done! 

我们**肉眼**看到的过程可能很简单：**操作系统首先读取文件内容，然后将其写入另一个文件，接着就完成了！**

However, things become complicated when we look more closely and memory is taken into account.

然而，当我们仔细观察并将内存考虑在内时，事情就变得复杂了。

As depicted in the dataflow below, the file read from disk must go through kernel system cache — which resides in the kernel space, then the data is copied to userspace’s memory area before being written back to a new file — which then in turn goes to kernel memory buffer before really flushed out to disk. 

如下面的数据流所示，从磁盘读取的文件必须经过内核系统缓存（位于内核空间），然后数据被复制到用户空间的内存区域，再写回一个新文件，然后再进入内核内存缓冲区，最后被刷新到磁盘。

The procedure takes quite many unnecessary operations of copying back and forth between kernel and userspace without actually doing anything, and the operations consume system resources and context switches as well. 

这个过程需要在内核和用户空间之间进行大量不必要的来回复制操作，但实际上什么都没做，而且这些操作还会消耗系统资源和上下文切换。

There’re room for improvement.

当然这部分还有改进余地。

<!-- more -->

Zero-copy technique comes into play with the purpose of eliminating all the unnecessary copies. In the Linux world the system call for that kind of work is **_sendfile()._**

零拷贝技术的目的是**消除所有不必要的拷贝**。在 Linux 世界中，这种工作的系统调用是 **_sendfile()。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230825130421.png)


Differences between data transfer using read()+write() / sendfile()

使用 `read()+write() / sendfile()` 进行数据传输的区别

# What is Zero-copy

_sendfile()_ claims to make data transfer happening under kernel space only — i.e data transferred from kernel system cache to NIC buffer (or traversed through kernel system cache if local copy), thus doesnt require context switches as in read+write combination. 

_sendfile()_ 声称只在内核空间进行数据传输，即数据从内核系统缓存传输到网卡缓冲区（如果是本地拷贝，则穿过内核系统缓存），因此不需要像读写结合那样进行上下文切换。

_sendfile()_ has now been widely used as a supported data transfering technique especially under **_nginx_** _and_ **_kafka._**

现在，_sendfile()_ 已被广泛用作一种受支持的数据传输技术，尤其是在**_nginx_** _和**_kafka下。

For ease of understanding we demonstrate a simple local file copy rather than file transfer over networking, and all the code’s error checking procedures are left out for clarity as well.

为了便于理解，我们演示的是简单的本地文件复制，而不是通过网络进行文件传输，而且为了清楚起见，我们也省略了代码中所有的错误检查程序。

**_readwrite.c_**

```c
#include <unistd.h>  
#include <sys/types.h>  
#include <sys/stat.h>  
#include <fcntl.h>#define BUF_SIZE 4096*1000

int main(int argc, char **argv) {  
    char buf[BUF_SIZE];  
    const char *fromfile = argv[1];  
    const char *tofile = argv[2];  
    struct stat stat_buf;  
    int fromfd = open(fromfile, O_RDONLY);  
    fstat(fromfd, &stat_buf);  
    int tofd = open(tofile, O_WRONLY | O_CREAT, stat_buf.st_mode);    int n;  
    while ((n = read(fromfd, &buf, sizeof(buf))) > 0) {  
        write(tofd, &buf, n);  
    }  
}
```

**_buf[BUF_SIZE]_** is the _user-space buffer_ that we’re talking about, as can be seen for every iteration, _read()_ copies data from file (through system memory cache) to this buffer, and _write()_ copies data from the buffer to another file (through system memory buffer)

**_buf[BUF_SIZE]_** 就是我们所说的**用户空间缓冲区**，可以看到，每次迭代时，_read()_ 都会将数据从文件（通过系统内存缓存）复制到该缓冲区，而 _write()_ 则会将数据从缓冲区复制到另一个文件（通过系统内存缓冲区）。

In the process memory map, _buf[BUF_SIZE]_ can be seen as a allocation of 4MB on stack area. Reducing the buffer size can help reduce the waste of memory, but it in turn increases number of read() and write() system calls, which is expensive as well.

在进程内存映射中，_buf[BUF_SIZE]_ 可以看作是在堆栈区域分配的 4MB 内存。减小缓冲区大小有助于减少内存浪费，但这反过来又会增加 read() 和 write() 系统调用的次数，而且代价高昂。

```
00007f53e08f6000       4       4       4 rw---   [ anon ]  
00007fff5a6b1000    4012    4008    4008 rw---   [ stack ]  
00007fff5ab3e000      12       0       0 r----   [ anon ]  
00007fff5ab41000       8       4       0 r-x--   [ anon ]
```

In the example, we demonstrate only one file transfer, for many transfers the memory waste might be significantly noticable using this naive technique.

在示例中，我们只演示了一次文件传输，而对于多次传输，使用这种简单的技术可能会造成明显的内存浪费。

**_sendfile.c_**

```c
#include <sys/types.h>  
#include <sys/stat.h>  
#include <fcntl.h>  
#include <sys/sendfile.h>#define BUF_SIZE 4096*1000int main(int argc, char **argv) {  
    const char *fromfile = argv[1];  
    const char *tofile = argv[2];  
    struct stat stat_buf;  
    int fromfd = open(fromfile, O_RDONLY);  
    fstat(fromfd, &stat_buf);  
    int tofd = open(tofile, O_WRONLY | O_CREAT, stat_buf.st_mode);    int n = 1;  
    while (n > 0) {  
        n = sendfile(tofd, fromfd, 0, BUF_SIZE);  
    }  
}
```

There’s no user-space buffer for _sendfile()._ For that reason, sendfile can send all the data from file at once, which eliminates the need of BUF_SIZE and a while loop, however we still keep it for comparing with read/write technique.

因此，`sendfile` 可以一次性发送文件中的所有数据，这样就不需要 用户空间 BUF_SIZE 和 while 循环，但我们需要仍然保留它，以便与读/写技术进行比较。
# Performance benchmark

Copy of a ~1G file. BUF_SIZE = 4K.

案例，复制一个1G 文件。BUF_SIZE = 4K。

**readwrite**

```c
syscall            calls    total       min       avg       max       
                               (msec)    (msec)    (msec)    (msec)  
   --------------- -------- --------- --------- --------- ---------   
   read              244797 16974.624     0.002     0.069   457.333  
   write             245169  2182.295     0.004     0.009   268.689
```

number of read / write is nearly the same, read() takes significanly more time because of major page faults.

读/写次数几乎相同，但 `read()` 要花费更多时间，因为会出现大的页面故障。

**sendfile**

```c
syscall            calls    total       min       avg       max     
                               (msec)    (msec)    (msec)    (msec)     
   --------------- -------- --------- --------- --------- ---------  
   sendfile          245261 13559.231     0.004     0.055   185.970 

```

number of sendfile() calls is by half of the total of read()+write(), which also helps reduce total execution time. For context switches, there’s lack of observation tool so it’s difficult to show the differences.

**sendfile()** 调用次数是 `read()+write()` 调用次数的一半，这也有助于减少总执行时间。至于上下文切换，由于缺乏观察工具，很难显示出差异。

In conclusion, _sendfile()_ brings to the table several benefits, including reduction of context switches, memory usage, number of system calls, and eventually faster operations. 

总之，_sendfile()_ 能带来多种好处，包括减少上下文切换、内存使用、系统调用次数，以及最终加快操作速度。

It is, however, not the silver bullet for everything, we once encountered the [problem of large file download on nginx,](https://trac.nginx.org/nginx/ticket/1870) therefore usage of _sendfile()_ should be considered and tested carefully before production use.

然而，它并不是万能的，我们曾经遇到过[nginx 上下载大文件的问题](https://trac.nginx.org/nginx/ticket/1870)，因此在生产使用 _sendfile()_ 之前，应仔细考虑和测试。

# References

- Chapter 61 — The Linux Programming Interface — Michael Kerrisk  
- 第 61 章 - Linux 编程接口 - Michael Kerrisk
- [https://developer.ibm.com/articles/j-zerocopy/](https://developer.ibm.com/articles/j-zerocopy/)
- [http://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile](http://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile)
- [https://kafka.apache.org/08/documentation.html](https://kafka.apache.org/08/documentation.html)