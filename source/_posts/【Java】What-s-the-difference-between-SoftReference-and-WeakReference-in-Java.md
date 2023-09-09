---
title: 【Java】What s the difference between SoftReference and WeakReference in Java
subtitle: What s the difference between SoftReference and WeakR eference in Java
author: lazytime
url_suffix: differencebetween
tags:
  - Java
categories:
  - Java
keywords: SoftReference,WeakReference
description: SoftReference和WeakReference
copyright: true
date: 2023-06-16 15:13:36
---
# 原文

[Whats the difference between SoftReference and WeakReference in Java?](https://stackoverflow.com/questions/299659/whats-the-difference-between-softreference-and-weakreference-in-java)

From [Understanding Weak References](https://web.archive.org/web/20061130103858/http://weblogs.java.net/blog/enicholas/archive/2006/05/understanding_w.html), by Ethan Nicholas:

# 引言

在学习JVM的过程中大概率会看到类似 **SoftReference** 和 **WeakReference**的字样，本部分挑选了Stack Flow 上的高赞回答进行整理。

# Understanding Weak References

[Understanding Weak References](https://web.archive.org/web/20061130103858/http://weblogs.java.net/blog/enicholas/archive/2006/05/understanding_w.html)

虽然是一篇2006年的博客，但是写的非常好，并且至今也依然有不小的价值，这里简单翻译和提取一下这篇文章的内容。

前面一大段说明写这篇文章的缘由是面试的时候发现很多面试者甚至不知道这个东西，相比之下自己不仅聊了作用和一些细节反差很大。

> Now, Im not suggesting you need to be a weak reference expert to qualify as a decent Java engineer. But I humbly submit that you should at least _know what they are_ -- otherwise how will you know when you should be using them? Since they seem to be a little-known feature, here is a brief overview of what weak references are, how to use them, and when to use them.

现在，我并不是说你需要成为一个薄弱的参考资料专家，才有资格成为一个体面的Java工程师。但我谦虚地认为，你至少应该知道它们是什么 -- 否则你怎么会知道什么时候应该使用它们？由于它们似乎是一个鲜为人知的特性，这里简要介绍一下什么是弱引用，如何使用它们，以及何时使用它们。

<!-- more -->

## Strong references

首先了解一下强引用，任何的`new`操作都是一个强引用：

```java
StringBuffer buffer = new StringBuffer();
```

> If an object is reachable via a chain of strong references (strongly reachable), it is not eligible for garbage collection.

如果一个对象具备强引用，并且引用链可到达，那么垃圾回收器就没有资格进行回收。

## When strong references are too strong

这部分大段文字描述的含义表明，有时候设计系统会把一些强引用对象放到HashMap这样的集合容器当中，它长的像这样：

```java
serialNumberMap.put(widget, widgetSerialNumber);
```

表面上看这似乎工作的很好，但是我们必须知道（**有100%的把握**）何时不再需要这个对象，一旦把这个对象溢出，很有可能存在隐藏的内存泄漏或者强引用无法释放导致无法垃圾回收的问题。

强引用的另外一个问题是缓存，假设你有一个需要处理用户提供的图片的应用程序，你想要利用缓存来减少图像加载带来的磁盘开销，同时要避免两份一模一样的图像同时存在内存当。

对于普通的强引用来说，这个引用本身就会迫使图像留在内存中，这就要求你（就像上面一样）以某种方式确定何时不再需要内存中的图像，并将其从缓存中删除，这样它就有资格被垃圾回收。

最要命的是这种情况下甚至需要考虑垃圾回收器的工作行为。

## Weak references

弱引用：简单来说是一个不够强大的引用，不足以迫使一个对象留在内存中。弱引用可以让程序员根据垃圾回收器的行为大致估算引用的生命周期。换句话说，弱引用肯定会在下一次垃圾收集器给回收掉，程序员无需担心类似强引用传递的问题。

在Java当中，通常用`java.lang.ref.WeakReference`类来表示。

```java
public class Main {
    public static void main(String[] args) {
     
        WeakReference<String> sr = new WeakReference<String>(new String("hello"));
         
        System.out.println(sr.get());
        System.gc();                //通知JVM的gc进行垃圾回收
        System.out.println(sr.get());
    }
}/**输出结果：
hello
null
**/

```

注意**被弱引用关联的对象是指只有弱引用与之关联**，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）

### ReferenceQueue 应用

此外，弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被JVM回收，这个引用就会被加入到与之关联的引用队列中。

```java
 @Test  
    public void referenceQueueUse() throws InterruptedException {  
        Object value = new Object();  
        Map<Object, Object> map = new HashMap<>();  
        ReferenceQueue<byte[]> referenceQueue = new ReferenceQueue<>();  
        for(int i = 0;i < 10000;i++) {  
            byte[] bytes = new byte[1024 * 1024];  
            WeakReference<byte[]> weakReference = new WeakReference<byte[]>(bytes, referenceQueue);  
            map.put(weakReference, value);  
        }  
        // 使用主线程观察 referenceQueue 的对象回收比较麻烦  
        Thread thread = new Thread(() -> {  
            try {  
                int cnt = 0;  
                WeakReference<byte[]> k;  
                // 等待垃圾回收  
                while((k = (WeakReference) referenceQueue.remove()) != null) {  
                    System.out.println((cnt++) + " 被回收了:" + k);  
                }  
            } catch(InterruptedException e) {  
                //结束循环  
            }  
        });  
//        thread.setDaemon(true);  
        thread.start();  
        thread.join();  
    }/**  
     0 被回收了:java.lang.ref.WeakReference@138caeca  
     1 被回收了:java.lang.ref.WeakReference@3402b4c9  
     2 被回收了:java.lang.ref.WeakReference@75769ab0  
     3 被回收了:java.lang.ref.WeakReference@2e23c180  
     4 被回收了:java.lang.ref.WeakReference@4aaae508  
     5 被回收了:java.lang.ref.WeakReference@3e84111a  
     6 被回收了:java.lang.ref.WeakReference@2cc04358  
     7 被回收了:java.lang.ref.WeakReference@45bb2aa1  
     8 被回收了:java.lang.ref.WeakReference@1639f93a  
     9 被回收了:java.lang.ref.WeakReference@f3021cb  
     */
```

上面的代码执行之后会触发垃圾回收，中的对象引用变为null就意味着对象是小并且将会被垃圾回收，WeakHashMap必须删除这种失效的条目，以避免持有不断增加的死的WeakReference。

在 WeakReference 中传入 ReferenceQueue，在引用失效的时候会被推入到ReferenceQueue当中。

### 在类weakHashMap中的使用

实际上weakHashMap就是使用weakReference当作key来进行数据的存储，所以key被gc之后响应的gc也可以移除掉。

**WeakHashMap Weak在于Key而非Value**，WeakHashMap的Key没有其他强引用时，这些key就可能被回收，且WeakHashMap可能会回收这些Key对应的键值对。

这里要举比较容易出错的例子，那就是String 本身为对象，但是如果是 `String xx = "xxx"`这样的定义方式，由于key会被加载常量池当中自动获得强引用，实际上不会被垃圾回收。

```java
@Test  
    public void weakHashMap(){  
        WeakHashMap<String, String> map = new WeakHashMap<>();  
        // 下面的注释说明 value 对于 WeakHashMap 过期没有任何影响，不管它是否在常量池当中有强引用。  
        String value = new String("value");  
//        String value = "value";  
  
        // new String 此时编译器无法感知，编译阶段无法提前感知，没有强引用保留。  
//        map.put(new String("key"), value); // {key=value}、  
  
        // 对于可在编译阶段确定的字符串，系统的字符串常量池会直接记录它，自动保留对它的强引用  
        map.put("key", value); // {key=value}  
        System.out.println(map);  
  
        System.gc();  
        System.runFinalization();  
  
        System.out.println(map);  
    }/**运行结果：  
     如果使用 new String("key") 则为下面的结果  
     {key=value}  
     {}  
     由于直接使用了字符串字面量“key”，造成了系统对“key”字符串的缓存，对其施加了强引用，因此GC未能销毁此实例  
     {key=value}  
     {key=value}     */
```

再看看下面的方法，可以看到如果让key，value 的引用同时使用一个引用，value实际使用的还是强引用，由于Key和Value都存在强引用，所以这种情况下也会出现key无法回收的问题：

```java  
/**  
 * 模拟 Intern 方法  
 * @param <T>  
 */  
class SimulaIntern<T> {  
    private WeakHashMap<T, T> weakHashMap = new WeakHashMap<T, T>();  
  
    /**  
     * 修复之后的对象使用  
     */  
    private WeakHashMap<String, MyObject> weakHashMapFix = new WeakHashMap<>();  
  
    /**  
     * 此方法存在问题  
     * @description 此方法存在问题  
     * @param item  
     * @return T  
     * @author xander  
     * @date 2023/6/16 14:48  
     */    public T intern(T item){  
        if(weakHashMap.get(item) != null){  
            return weakHashMap.get(item);  
        }  
        // 根源在于这里的 put， 对于 key 来说是弱引用，但是 value 依旧是强引用保留  
        weakHashMap.put(item, item);  
        return item;  
    }  
  
    public MyObject intern(String key, MyObject value) {  
        MyObject object = weakHashMapFix.get(key);  
        if (object != null) {  
            return object;  
        }  
        weakHashMapFix.put(key, value);  
        return value;  
    }  
  
    public void print(){  
        System.out.println("weakHashMap => "+ weakHashMap);  
        System.out.println("weakHashMapFix => "+weakHashMapFix);  
    }  
  
}  
  
class MyObject{  
  
}  
  
@Test  
public void testWeakMap(){  
    // intern 存在问题  
    SimulaIntern<String> testPool = new SimulaIntern<>();  
    testPool.intern(new String("testPool"));  
    testPool.print();  
    System.gc();  
    System.runFinalization();  
    testPool.print();  
  
    // 可以正常清理，实际上就是把 k 和 value 两者进行拆分  
    SimulaIntern simulaIntern = new SimulaIntern();  
    simulaIntern.intern(new String("name"), new MyObject());  
    simulaIntern.print();  
    System.gc();  
    System.runFinalization();  
    simulaIntern.print();  
}/**运行结果：  
 // 无法正常清理  
 weakHashMap => {testPool=testPool}  
 weakHashMap => {testPool=testPool}  
 // 修改之后， 可以正常清理，实际上就是把 k 和 value 两者进行拆分  
 weakHashMapFix => {name=com.zxd.interview.weakref.WeakReferenceTest$MyObject@1936f0f5}  
 weakHashMapFix => {}*/
```

## Soft references

软引用和弱引用的区别是，弱引用在没有强引用加持的情况下，通常必然会在下一次垃圾回收给清理掉，但软引用则稍微强一些，一般会坚持更长的时间，一般情况下如果内存足够，软引用基本会被保留。

所以软银用的特征是：**只要内存供应充足，Soft reachable对象通常会被保留**。如果既担心垃圾回收提前处理掉对象，又担心内存消耗的需求，就可以考虑软引用。

## Phantom references

虚引用和软引用和弱引用完全不一样，它的 get 方法经常返回 Null，虚引用的唯一用途是跟踪什么时候对象会被放入到引用队列，那么这种放入引用队列的方式和弱引用有什么区别？

主要的区别是入队的时间，WeakReference在它所指向的对象变为弱引用（无强引用）的时候会被推入到队列，因为可以在queue 中获取到弱引用对象，该对象甚至可以通过非正统的finalize()方法 "复活"，但WeakReference仍然是死的。

**PhantomReference**只有在**对象被从内存中物理删除时才会被排队**，而且get()方法总是返回null，主要是为了防止你能够 "复活 "一个几乎死去的对象。所以 PhantomReference 会有什么用：

- 它们允许你准确地确定一个对象何时从内存中删除。 
- PhantomReferences避免了finalize()的一个基本问题。

第一点准确的确定一个对象何时删除，造某些大内存占用的情况下可以实现等待垃圾回收再加载下一个图片防止OOM。

第二点则finalize()方法可以通过为对象创建新的强引用来 "复活 "它们，但是虚引用可以很大概率避免。当一个PhantomReference被排队时，绝对没有办法获得现在已经死亡的对象的指针，该对象可以在第一个垃圾收集周期中被发现可以幻象地到达时被立即清理掉。

# Stack Flow 的答案

这些答案基本都来自上面提到的博客相同，这里不再过多赘述。

**Weak references**

 A _weak reference_, simply put, is a reference that isnt strong enough to force an object to remain in memory. Weak references allow you to leverage the garbage collectors ability to determine reachability for you, so you dont have to do it yourself. You create a weak reference like this

```java
WeakReference weakWidget = new WeakReference(widget);
```

and then elsewhere in the code you can use `weakWidget.get()` to get the actual `Widget` object. Of course the weak reference isnt strong enough to prevent garbage collection, so you may find (if there are no strong references to the widget) that `weakWidget.get()` suddenly starts returning `null`.

...

**Soft references**

A _soft reference_ is exactly like a weak reference, except that it is less eager to throw away the object to which it refers. An object which is only weakly reachable (the strongest references to it are `WeakReferences`) will be discarded at the next garbage collection cycle, but an object which is softly reachable will generally stick around for a while.

`SoftReferences` arent _required_ to behave any differently than `WeakReferences`, but in practice softly reachable objects are generally retained as long as memory is in plentiful supply. This makes them an excellent foundation for a cache, such as the image cache described above, since you can let the garbage collector worry about both how reachable the objects are (a strongly reachable object will _never_ be removed from the cache) and how badly it needs the memory they are consuming.

最后的补充部分简单翻译一下

> And Peter Kessler added in a comment:

下面是评论中的一些补充

> The Sun JRE does treat SoftReferences differently from WeakReferences. We attempt to hold on to object referenced by a SoftReference if there isnt pressure on the available memory. 

Sun JRE对待SoftReferences的方式与WeakReferences不同。如果可用内存并且此时没有使用压力，会尝试保留被SoftReference引用的对象。

> One detail: the policy for the "-client" and "-server" JREs are different: the -client JRE tries to keep your footprint small by preferring to clear SoftReferences rather than expand the heap, whereas the -server JRE tries to keep your performance high by preferring to expand the heap (if possible) rather than clear SoftReferences. One size does not fit all.

"-client "和"-server "JRE的策略是不同的：
- `-client`：JRE试图通过**优先清除SoftReferences**而不是扩展堆来保持你的内存。
- `-server`：JRE则试图通过优先扩展堆（如果可能）而不是清除SoftReferences来保持你的性能