---
title: 【JVM】Try to Avoid -XX UseGCLogFileRotation
subtitle: JVM启动参数注意
url_suffix: usegclogfilerotation
author: lazytime
tags:
  - JVM
categories:
  - JVM
keywords: UseGCLogFileRotation,jvm
description: JVM相关参数解释
copyright: true
date: 2023-02-02 12:14:30
---

# Try to Avoid -XX:+UseGCLogFileRotation

Source：[https://dzone.com/articles/try-to-avoid-xxusegclogfilerotation](https://dzone.com/articles/try-to-avoid-xxusegclogfilerotation)

Developers take advantage of the JVM argument -XX:+UseGCLogFileRotation to rotate GC log files.

开发人员利用JVM参数-XX:+UseGCLogFileRotation来递换GC日志文件。

```java
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/home/GCEASY/gc.log -
XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M"
```

As shown above, the JVM will rotate the GC log file whenever its size reaches 20MB. It will generate up to five files, with extensions `gc.log.0`,  `gc.log.1`, `gc.log.2`, `gc.log.3`, and `gc.log.4`.

如上所示的配置会产生5个日志文件，并且每个日志文件有20M。

<!-- more -->

# Losing Old GC Logs 日志丢失

Suppose you configured  `-XX:NumberOfGCLogFiles=5`, then over a period of time, five GC log files will be created:

假设你配置了`-XX:NumberOfGCLogFiles=5`，那么在一段时期内，将创建五个GC日志文件。

-   **gc.log.0** ← _oldest GC Log content_
-   **gc.log.1**
-   **gc.log.2**
-   **gc.log.3**
-   **gc.log.4** ← _latest GC Log content_

The most recent GC log contents will be written to `gc.log.4` and old GC log contents will be present in `gc.log.0`.

最新的GC日志内容将被写入`gc.log.4`，旧的GC日志内容将出现在`gc.log.0`。

When the application starts to generate more GC logs than the configured  `-XX:NumberOfGCLogFiles`, in this case, five, then old GC log contents in `gc.log.0` will be deleted. New GC events will be written to  `gc.log.0`. It means that you will end up not having all the generated GC logs. You will lose the visibility of all events.

当应用程序配置的`-XX:NumberOfGCLogFiles`产生更多的GC日志时（在本例中是5个），`gc.log.0`中的旧GC日志内容将被删除。新的GC日志将被写入`gc.log.0`。这意味着会存在旧日志文件的覆盖现象，将**失去所有日志的可见性**。

# Mixed-Up GC Logs 混杂日志

Suppose an application has created five GC log files, including:

假设一个应用程序创建了五个GC日志文件，包括：

-   **gc.log.0**
-   **gc.log.1**
-   **gc.log.2**
-   **gc.log.3**
-   **gc.log.4**

Then, let’s say you are restarting the application. Now, new GC logs will be written to `gc.log.0` file and old GC log content will be present in `gc.log.1`, `gc.log.2`, `gc.log.3`, `gc.log.4`, etc.

然后，假设你正在重启应用程序。现在新的GC日志将被写入`gc.log.0`文件，而旧的GC日志内容将出现在`gc.log.1`、`gc.log.2`、`gc.log.3`、`gc.log.4`。

-   **gc.log.0** ← GC log file content after restart
-   **gc.log.1** ← GC log file content before restart
-   **gc.log.2** ← GC log file content before restart
-   **gc.log.3** ← GC log file content before restart
-   **gc.log.4** ← GC log file content before restart

So, your new GC log contents get mixed up with old GC logs. Thus, to mitigate this problem, you might have to move all the old GC logs to a different folder before you restart the application.

所以新GC日志和旧的GC日志会混合在一起，为了缓解这个问题，你可能要在重启应用程序之前把所有旧的GC日志移到一个不同的文件夹里。


# Forwarding GC Logs to a Central Location 将GC日志转发到一个中心位置 

In this approach, the current active file to which GC logs are written is marked with the extension  `.current`. For example, if GC events are currently written to the file `gc.log.3`, it would be named as: `gc.log.3.current`.

在这种方法中，当前写入GC日志的活动文件被标记为扩展名`.current`。例如，如果GC事件当前被写入文件`gc.log.3`，它将被命名为。 `gc.log.3.current`。

If you want to forward GC logs from each server to a central location, then most DevOps engineers use  `rsyslog`. However, this file naming convention poses a significant challenge to use `rsyslog`, as [described in this blog](http://www.planetcobalt.net/sdb/forward_gc_logs.shtml).

如果你想把每台服务器的GC日志转发到一个中心位置，那么大多数DevOps工程师会使用`rsyslog`。然而，这种文件命名惯例给使用`rsyslog`带来了巨大的挑战，正如[这篇博客所述](http://www.planetcobalt.net/sdb/forward_gc_logs.shtml)。


# Tooling 工具

Now, to analyze the GC log file using the GC tools such as ([GCeasy](https://gceasy.io/), GCViewer, etc.), you will have to upload multiple GC log files instead of just one single GC Log file.

现在，为了使用GC工具分析GC日志文件，如（[GCeasy](https://gceasy.io/), GCViewer等），将不得不上传多个GC日志文件，而不是只有一个GC日志文件。

> 显然非常麻烦并且非常反人类。

# Recommended Solution 推荐方案

We can suffix the GC log file with the time stamp at which the JVM was restarted, then the GC Log file locations will become unique. Then, new GC logs will not override the old GC logs. It can be achieved by suffixing `%t` to the GC log file name, as shown below:

我们可以在GC日志文件后缀加入**JVM重启的时间戳**（解决这个问题），那么GC日志文件的位置将变得独一无二。然后新的GC日志就不会覆盖旧的GC日志了。这可以通过在GC日志文件名后缀`%t`来实现，如下所示：

> "-XX:+PrintGCDetails -XX:+PrintGCDateStamps **-Xloggc:/home/GCEASY/gc-%t.log**"

 `%t` suffixes timestamp to the GC log file in the format:  `YYYY-MM-DD_HH-MM-SS`. So, the generated GC log file name will start to look like: `gc-2019-01-29_20-41-47.log`.

 `%t`后缀为GC日志文件的时间戳，格式为:  `yyyy-mm-dd_hh-mm-ss`。因此，生成的GC日志文件名将开始看起来像： `gc-2019-01-29_20-41-47.log`.

This simple solution addresses all the shortcomings of `-XX:+UseGCLogFileRotation`.

**这个简单的解决方案解决了`-XX:+UseGCLogFileRotation`的所有缺点。**
