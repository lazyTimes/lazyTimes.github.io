---
title: 简单分析Java的HashMap.entrySet()的实现
subtitle: 简单分析Java的HashMap.entrySet()的实现,底层实现的原理
author: lazytime
tags:
  - Map
categories:
  - 集合
keywords: hashmap
description: hashmap的entrySet是如何实现的
copyright: true
date: 2020-08-29 22:34:22
---

# 简单分析Java的HashMap.entrySet()的实现

转载自：https://blog.csdn.net/weixin_30807779/article/details/99566016

<!-- more -->

## 前言：

map通过Entry 实现一个 <K,V> 的存储，通过key 的哈希值确定哈希表的数组下标，通过下标找到桶位之后，通过遍历找到相对应的Value

map是用来存储key-value类型数据的，一个<k, v>对在Map的接口定义中被定义为Entry，HashMap内部实现了`Entry`接口。HashMap内部维护一个`Entry`数组。



## 解析

`transient Entry[] table;` 

注意关键字：`transient` 该关键字，对于此属性是不能进行序列化的

```
Entry[] table
    +---+
    | 0 | -> entry_0_0 -> entry_0_1 -> null
    +---+
    | 1 | -> null
    +---+
    |   |

     ...

    |n-1| -> entry_n-1_0 -> null
    +---+
```

使用map 的 entrySet() 方法，就可以获得一个`EntryIterator`类型的实例，该实例调用 `nextEntry()`方法，来获取迭代器的下一个元素。`EntryIterator`类型是泛型`HashIterator<T>`的一个子类，这个类的内容很简单，唯一的代码是在`next()`函数中调用了`HashIterator`的`nextEntry()`方法。

```java
HashMap
    |- table <------------------------------------\
    \+ entrySet()                                 |iterates
        |              HashMap.HashIterator<T>    |
        |returns                ^       \- nextEntry()
        V                       -                 ^
HashMap.EntrySet                |                 |
    \- iterator()               |extends          |
            |                   |                 |
            |  instantiats      |                 |calls
            \----------> HashMap.EntryIterator    |
                                        \- next() /
```

`entrySet()`方法返回的是一个特殊的Set，定义为HashMap的内部私有类 `EntryIterator` ,它继承自 `HashIterator` 

`HashIterator`通过遍历`table`数组，实现对HashMap的遍历。内部维护几个变量：`index`记录当前在`table`数组中的下标，`current`用来记录当前在`table[index]`这个链表中的位置，`next`指向current的下一个元素。`nextEntry()`的完整代码如下：

```java
final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    Entry<K,V> e = next;
    if (e == null)
        throw new NoSuchElementException();
 
    if ((next = e.next) == null) {
        Entry[] t = table;
        while (index < t.length && (next = t[index++]) == null)
            ;
    }
    current = e;
    return e;
}
```



## 总结

从调用链路可以看出， entrySet() 最后得到的其实是一个关于 map 的一个`视图`，当他在迭代的时候，其实就是相当于对于map的元素进行查找，不会创建任何额外的空间。

同时，`entrySet` 本身也是迭代器的思想，在多线程的环境下其实是线程不安全的，这点需要注意

## 其他

> 为什么要对于table 进行非串行化？
>
> [Java中HashMap关键字transient的疑惑](https://developer.aliyun.com/ask/62542?spm=a2c6h.13159736)
>
> 怎么理解? 看一下HashMap.get()/put()知道, 读写Map是根据Object.hashcode()来确定从哪个bucket读/写. 而Object.hashcode()是native方法, 不同的JVM里可能是不一样的.
>
> 打个比方说, 向HashMap存一个entry, key为 字符串"STRING", 在第一个java程序里, "STRING"的hashcode()为1, 存入第1号bucket; 在第二个java程序里, "STRING"的hashcode()有可能就是2, 存入第2号bucket. 如果用默认的串行化(Entry[] table不用transient), 那么这个HashMap从第一个java程序里通过串行化导入第二个java程序后, 其内存分布是一样的. 这就不对了. HashMap现在的readObject和writeObject是把内容 输出/输入, 把HashMap重新生成出来.



> 关于JDK官方吧Entry 换为了Node 的讨论
>
> 1. 为什么要换为Node
> 2. 更换之后和原来的设计有什么区别
>
> 回答：JDK1.8 改动： 在`java1.8`中,`Entry`被`Node`替代(换了一个马甲)。

