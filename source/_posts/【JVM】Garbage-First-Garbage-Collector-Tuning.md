---
title: 【JVM】Garbage First Garbage Collector Tuning
subtitle: 了解如何适应和调整G1 GC的评估、分析和性能
author: lazytime
url_suffix: adaptg1gc
tags:
  - Java
categories:
  - Java
keywords: garbage
description: gc
copyright: true
date: 2023-09-11 10:39:56
---
# Source

[Garbage First Garbage Collector Tuning (oracle.com)](https://www.oracle.com/technical-resources/articles/java/g1gc.html)

# **Learn about how to adapt and tune the G1 GC for evaluation, analysis and performance.** 了解如何适应和调整G1 GC的评估、分析和性能


The [Garbage First Garbage Collector (G1 GC)](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html) is the low-pause, server-style generational garbage collector for Java HotSpot VM. The G1 GC uses concurrent and parallel phases to achieve its target pause time and to maintain good throughput. When G1 GC determines that a garbage collection is necessary, it collects the regions with the least live data first (garbage first).

[Garbage First Garbage Collector (G1 GC)](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html)是 Java HotSpot VM 的低等待、Server端垃圾收集器。G1 GC 使用并发和并行阶段来实现其目标暂停时间，并保持良好的吞吐量。当 G1 GC 确定有必要进行垃圾收集时，它会首先收集实时数据最少的区域（垃圾优先）。

A garbage collector (GC) is a memory management tool. The G1 GC achieves automatic memory management through the following operations:

垃圾收集器（GC）是一种内存管理工具。G1 GC 通过以下操作实现自动内存管理：

- Allocating objects to a young generation and promoting aged objects into an old generation.
- Finding live objects in the old generation through a concurrent (parallel) marking phase. The Java HotSpot VM triggers the marking phase when the total Java heap occupancy exceeds the default threshold.
- Recovering free memory by compacting live objects through parallel copying.

- 将对象分配到年轻一代，并将老化对象提升到老一代。
- 通过并发（并行）标记阶段，在旧一代中查找有效对象。Java HotSpot VM 会在 Java 堆总占用率超过默认阈值时触发标记阶段。
- 通过并行复制压缩活对象来恢复空闲内存。

<!-- more -->

Here, we look at how to adapt and tune the G1 GC for evaluation, analysis and performance—we assume a basic understanding of Java garbage collection.

在此，我们将探讨如何调整 G1 GC 并使其满足评估、分析和性能要求--我们假设您对 Java 垃圾回收有基本的了解。

The G1 GC is a regionalized and generational garbage collector, which means that the Java object heap (heap) is divided into a number of equally sized regions. 

G1 GC 是一种区域化和世代垃圾收集器，这意味着 Java 对象堆（heap）被划分为多个大小相等的区域。

Upon startup, the Java Virtual Machine (JVM) sets the region size. 

启动时，Java 虚拟机（JVM）会设置区域大小。

The region sizes can vary from 1 MB to 32 MB depending on the heap size. 

根据堆的大小，区域大小从 1 MB 到 32 MB 不等。

The goal is to have no more than 2048 regions. 

目标是不超过 2048 个区域。

The eden, survivor, and old generations are logical sets of these regions and are not contiguous.

伊甸园"、"幸存者 "和 "旧世代 "是这些区域的逻辑集合，并不连续。

The G1 GC has a pause time-target that it tries to meet (soft real time).

G1 GC 有一个试图达到的暂停时间目标（软实时）。

 During young collections, the G1 GC adjusts its young generation (eden and survivor sizes) to meet the soft real-time target. 

在幼年期收集期间，G1 GC 会调整其幼年期生成（eden 和 survivor 大小），以满足软实时目标。

During mixed collections, the G1 GC adjusts the number of old regions that are collected based on a target number of mixed garbage collections, the percentage of live objects in each region of the heap, and the overall acceptable heap waste percentage.

在混合收集期间，G1 GC 会根据混合垃圾收集的目标数量、堆中每个区域中活对象的百分比以及可接受的总体堆浪费百分比，调整旧区域的收集数量。

The G1 GC reduces heap fragmentation by incremental parallel copying of live objects from one or more sets of regions (called Collection Set (CSet)) into different new region(s) to achieve compaction. 

G1 GC 通过将实时对象从一组或多组区域（称为集合集（CSet））增量并行复制到不同的新区域来实现压缩，从而减少堆碎片。

The goal is to reclaim as much heap space as possible, starting with those regions that contain the most reclaimable space, while attempting to not exceed the pause time goal (garbage first).

其目标是尽可能多地回收堆空间，从包含最多可回收空间的区域开始，同时尽量不超过暂停时间目标（垃圾优先）。

The G1 GC uses independent Remembered Sets (RSets) to track references into regions. 

G1 GC 使用独立的记忆集 (RSets) 来跟踪对区域的引用。

Independent RSets enable parallel and independent collection of regions because only a region's RSet must be scanned for references into that region, instead of the whole heap. 

独立 `RSet` 可实现并行和独立的区域收集，因为只需扫描区域的 `RSet` 以查找对该区域的引用，而无需扫描整个堆。

The G1 GC uses a post-write barrier to record changes to the heap and update the RSets.

G1 GC 使用写后屏障来记录堆的变化并更新 `RSets`。
# Garbage Collection Phases垃圾收集阶段

Apart from evacuation pauses (described below) that compose the stop-the-world (STW) young and mixed garbage collections, the G1 GC also has parallel, concurrent, and multiphase marking cycles. 

除了构成 "Stop-World"（STW）年轻模式和混合垃圾收集的疏散暂停（如下所述）外，G1 GC 还具有并行、并发和多阶段标记周期。

G1 GC uses the Snapshot-At-The-Beginning (SATB) algorithm, which takes a snapshot of the set of live objects in the heap at the start of a marking cycle. 

G1 GC 使用 "**开始时快照**"（SATB）算法，在标记周期开始时对堆中的实时对象集进行快照。

The set of live objects is composed of the live objects in the snapshot, and the objects allocated since the start of the marking cycle. 

实时对象集由快照中的实时对象和标记周期开始后分配的对象组成。

The G1 GC marking algorithm uses a pre-write barrier to record and mark objects that are part of the logical snapshot.

G1 GC 标记算法使用预写屏障来记录和标记逻辑快照中的对象。
# Young Garbage Collections 年轻代垃圾手机

The G1 GC satisfies most allocation requests from regions added to the eden set of regions. 

G1 GC 可满足大部分来自添加到 Eden 区域集的区域的分配请求。

During a young garbage collection, the G1 GC collects both the eden regions and the survivor regions from the previous garbage collection. 

在一次新的垃圾回收过程中，G1 GC 会同时回收上一次垃圾回收中的 eden 区域和 survivor 区域。

The live objects from the eden and survivor regions are copied, or evacuated, to a new set of regions. 

来自 eden 和 survivor 区域的实时对象会被复制或疏散到一组新的区域中。

The destination region for a particular object depends upon the object's age; an object that has aged sufficiently evacuates to an old generation region (that is, promoted); otherwise, the object evacuates to a survivor region and will be included in the CSet of the next young or mixed garbage collection.

特定对象的目标区域取决于该对象的年龄；年龄足够大的对象会被移动到老一代区域（即晋升）；否则，该对象会被移动到存活区域，并被纳入下一次年轻代或混合垃圾收集的 CSet 中。
# Mixed Garbage Collections

Upon successful completion of a concurrent marking cycle, the G1 GC switches from performing young garbage collections to performing mixed garbage collections. 

成功完成并发标记周期后，G1 GC 会从执行年轻垃圾收集切换到执行混合垃圾收集。

In a mixed garbage collection, the G1 GC optionally adds some old regions to the set of eden and survivor regions that will be collected. 

在混合垃圾收集中，G1 GC 会选择性地将一些old region添加到将被收集的 Eden 和 survivor 区域集合中。

The exact number of old regions added is controlled by a number of flags that will be discussed later (see "[Taming Mixed GCs](https://www.oracle.com/technical-resources/articles/java/g1gc.html#Taming)"). 

添加old region的具体数量由一些标志控制，稍后将讨论这些标志（参见"[驯服混合 GC](https://www.oracle.com/technical-resources/articles/java/g1gc.html#Taming)"）。

After the G1 GC collects a sufficient number of old regions (over multiple mixed garbage collections), G1 reverts to performing young garbage collections until the next marking cycle completes.

在 G1 GC 收集到足够数量的旧区域后（经过多次混合垃圾收集），G1 将恢复执行年轻垃圾收集，直到下一个标记周期完成。

# Phases of the Marking Cycle

The marking cycle has the following phases:

- **Initial mark phase**: The G1 GC marks the roots during this phase. This phase is piggybacked on a normal (STW) young garbage collection.
- **Root region scanning phase**: The G1 GC scans survivor regions of the initial mark for references to the old generation and marks the referenced objects. This phase runs concurrently with the application (not STW) and must complete before the next STW young garbage collection can start.
- **Concurrent marking phase**: The G1 GC finds reachable (live) objects across the entire heap. This phase happens concurrently with the application, and can be interrupted by STW young garbage collections.
- **Remark phase**: This phase is STW collection and helps the completion of the marking cycle. G1 GC drains SATB buffers, traces unvisited live objects, and performs reference processing.
- **Cleanup phase**: In this final phase, the G1 GC performs the STW operations of accounting and RSet scrubbing. During accounting, the G1 GC identifies completely free regions and mixed garbage collection candidates. The cleanup phase is partly concurrent when it resets and returns the empty regions to the free list.

垃圾回收的阶段如下：

- **初始标记阶段**： G1 GC 在此阶段对root进行标记。该阶段是正常（STW）年轻垃圾回收的前置阶段。  
- **根区域扫描阶段**： G1 GC 会扫描初始标记的`survivor`区域，查找对旧一代的引用，并标记引用对象。该阶段与应用程序（非 STW）同时运行，必须在下一次 STW 年轻垃圾收集开始前完成。  
- **并发标记阶段**： G1 GC 会在整个堆中找到可触及的（活）对象。该阶段与应用程序同时进行，可能会被 STW 年轻垃圾收集中断。  
- **标记阶段**： 此阶段为 STW 收集，有助于完成标记周期。G1 GC 会耗尽 SATB 缓冲区，跟踪未访问的实时对象，并执行引用处理。  
- **清理阶段**： 在这一最后阶段，G1 GC 执行核算和 RSet 擦除的 STW 操作。在核算过程中，G1 GC 会识别完全空闲的区域和混合垃圾收集候选对象。清理阶段的部分并发操作是重置空区域并将其返回到空闲列表。  

# Important Defaults

The G1 GC is an adaptive garbage collector with defaults that enable it to work efficiently without modification. 

G1 GC 是一种自适应垃圾回收器，其默认值使其无需修改即可高效工作。

Here is a list of important options and their default values. 

下面列出了重要选项及其默认值。

This list applies to the latest Java HotSpot VM, build 24. 

此列表适用于最新的 Java HotSpot VM 第 24 版。

You can adapt and tune the G1 GC to your application performance needs by entering the following options with changed settings on the JVM command line.

您可以通过在 JVM 命令行中输入以下已更改设置的选项，调整 G1 GC 以满足您的应用程序性能需求。

- `-XX:G1HeapRegionSize=n`  
  
    Sets the size of a G1 region. The value will be a power of two and can range from 1MB to 32MB. The goal is to have around 2048 regions based on the minimum Java heap size.
    
    设置 G1 region 的大小。该值是 2 的幂次，范围从 1MB 到 32MB。我们的目标是根据 Java 堆的最小大小，设置大约 2048 个区域。
    
- `-XX:MaxGCPauseMillis=200`  
  
    Sets a target value for desired maximum pause time. The default value is 200 milliseconds. The specified value does not adapt to your heap size.
     设置所需**最大暂停时间的目标值**。默认值为 **200** 毫秒。指定值与堆大小无关。

- `-XX:G1NewSizePercent=5`  
  
    Sets the percentage of the heap to use as the minimum for the young generation size. The default value is 5 percent of your Java heap. This is an experimental flag. See "[How to unlock experimental VM flags](https://www.oracle.com/technical-resources/articles/java/g1gc.html#Unlock)" for an example. This setting replaces the `-XX:DefaultMinNewGenPercent` setting. This setting is not available in Java HotSpot VM, build 23.
    
- `-XX:G1MaxNewSizePercent=60`  
  
    Sets the percentage of the heap size to use as the maximum for young generation size. The default value is 60 percent of your Java heap. This is an experimental flag. See "[How to unlock experimental VM flags](https://www.oracle.com/technical-resources/articles/java/g1gc.html#Unlock)" for an example. This setting replaces the `-XX:DefaultMaxNewGenPercent` setting. This setting is not available in Java HotSpot VM, build 23.
    
- `-XX:ParallelGCThreads=n`  
  
    Sets the value of the STW worker threads. Sets the value of n to the number of logical processors. The value of `n` is the same as the number of logical processors up to a value of 8.
    
    If there are more than eight logical processors, sets the value of `n` to approximately 5/8 of the logical processors. This works in most cases except for larger SPARC systems where the value of `n` can be approximately 5/16 of the logical processors.
    
- `-XX:ConcGCThreads=n`  
  
    Sets the number of parallel marking threads. Sets `n` to approximately 1/4 of the number of parallel garbage collection threads (`ParallelGCThreads`).
    
- `-XX:InitiatingHeapOccupancyPercent=45`  
  
    Sets the Java heap occupancy threshold that triggers a marking cycle. The default occupancy is 45 percent of the entire Java heap.
    
- `-XX:G1MixedGCLiveThresholdPercent=65`  
  
    Sets the occupancy threshold for an old region to be included in a mixed garbage collection cycle. The default occupancy is 65 percent. This is an experimental flag. See "[How to unlock experimental VM flags](https://www.oracle.com/technical-resources/articles/java/g1gc.html#Unlock)" for an example. This setting replaces the `-XX:G1OldCSetRegionLiveThresholdPercent` setting. This setting is not available in Java HotSpot VM, build 23.
    
- `-XX:G1HeapWastePercent=10`  
  
    Sets the percentage of heap that you are willing to waste. The Java HotSpot VM does not initiate the mixed garbage collection cycle when the reclaimable percentage is less than the heap waste percentage. The default is 10 percent. This setting is not available in Java HotSpot VM, build 23.

    设置触发垃圾回收之后剩余可被回收的百分比触发Mixed GC的百分比。如果当存活的垃圾对象百分比小于此设置的百分比时，Java HotSpot VM 不会启动Mixed GC。默认值为 10%。此设置在 Java HotSpot VM 第 23 版中不可用。

- `-XX:G1MixedGCCountTarget=8`  
  
    Sets the target number of mixed garbage collections after a marking cycle to collect old regions with at most `G1MixedGCLIveThresholdPercent` live data. The default is 8 mixed garbage collections. The goal for mixed collections is to be within this target number. This setting is not available in Java HotSpot VM, build 23.

    设置标记周期后混合垃圾收集的目标次数，以收集最多具有 `G1MixedGCLIveThresholdPercent` 实时数据的旧区域。默认为 8 次混合垃圾收集。混合收集的目标是在此目标次数范围内。此设置在 Java HotSpot VM 第 23 版中不可用。
    
- `-XX:G1OldCSetRegionThresholdPercent=10`  
  
    Sets an upper limit on the number of old regions to be collected during a mixed garbage collection cycle. The default is 10 percent of the Java heap. This setting is not available in Java HotSpot VM, build 23.

	**设置在混合垃圾收集周期内收集的旧区域数量上限**。默认值为 Java 堆的 10%。此设置在 Java HotSpot VM 第 23 版中不可用。
    
- `-XX:G1ReservePercent=10`  （保留内存百分比）
  
    Sets the percentage of reserve memory to keep free so as to reduce the risk of to-space overflows. The default is 10 percent. When you increase or decrease the percentage, make sure to adjust the total Java heap by the same amount. This setting is not available in Java HotSpot VM, build 23.
    
    **设置保留空闲内存的百分比**，以降低空间溢出的风险。默认值为 10%。增加或减少该百分比时，请确保**以相同的数量调整 Java 堆总数**。此设置在 **Java HotSpot VM 第 23 版中不可用**。


## About the Author 关于作者

Monica Beckwith, Principal Member of Technical Staff at Oracle, is the performance lead for the Java HotSpot VM's Garbage First Garbage Collector. 

Monica Beckwith 是甲骨文公司的首席技术人员，是 Java HotSpot 虚拟机垃圾优先垃圾收集器的性能负责人。

She has worked in the performance and architecture industry for over 10 years. 

她在性能和架构行业工作了 10 多年。

Prior to Oracle and Sun Microsystems, Monica lead the performance effort at Spansion Inc. 

在加入 Oracle 和 Sun Microsystems 之前，Monica 在 Spansion Inc. 

Monica has worked with many industry standard Java based benchmarks with a constant goal of finding opportunities for improvement in the Java HotSpot VM。

Monica 曾参与过许多基于行业标准 Java 的基准测试，始终致力于寻找 Java HotSpot VM 的改进机会。