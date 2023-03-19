---
title: 【JAVA】JDK11新特性个人分析
subtitle: JDK11新特性个人分析
author: Xander
url_suffix: jdk11update
tags:
  - Java
categories:
  - Java
keywords: JDK11新特性个人分析
description: JDK11
copyright: true
date: 2023-03-19 15:45:58
---
# 历史背景

2018年9 月 26 日，Oracle 官方宣布 Java 11 正式发布。这个版本中一共包含 **17 个 JEP（JDK Enhancement Proposals，JDK 增强提案）**。

**JDK 11 是一个长期支持版本（LTS, Long-Term-Support）**，在编写本文的时间节点下和JDK17一样被用于编写项目代码的主流版本。

本文结合了各方资料整理出JDK11的新特性，工作上使用高版本JDK的机会确实不多，但是自己体验的过程中确实切身体会到JDK升级的好处。

## 升级JDK11的意义

- 大量的新特性、Bug 修复，例如，容器环境支持，GC 等基础领域的增强。
- 安全协议更新，比如废弃TLS1.2中不安全的加密算法，部分支持TLS1.3协议。
- JVM的零成本优化。
- 划时代的**垃圾收集器ZGC**，关键目的是实现GC时间不超过10ms。
- **G1垃圾收集器的免费升级**
	- 并行的 Full GC（JDK11之前Oracle没有考虑过G1需要并行Full GC）
	- 快速的 CardTable 扫描
	- 自适应的堆占用比例调整（IHOP）
	- 在并发标记阶段的类型卸载
- **JVM自带工具增强**
	- JEP 328: **Flight Recorder（JFR）**，商用版工具JFR转为开源。
	- JEP 331: **Low-Overhead Heap Profiling** 实现底层成本堆分析JVM扩展
- TLS协议精简
	- 完善Java标准HTTP类库上的扩展能力。
	- JEP 332: Transport Layer Security (TLS) 1.3，完全支持**TLS 1.3**（协议部分部分支持，加密算法部分基本满足要求）。
- **HTTP/2 Client API**
	- 完善了Java语言自身进行Http Client的能力
	- 等同于把HttpClient官方化（难听点就是搬来了）
	- <s>只要不写作者，谁也不知道自己写的还是抄的谁的</s>
- 废弃Nashorn JavaScript引擎
	- 实际上是全力推动Graal VM虚拟机的开发。

<!-- more -->

# 升级内容

下面就是根据"JEP（JDK Enhancement Proposals，JDK 增强提案）"进行介绍。关于原始的提案名称，这里放到了“附录”部分。

## 新特性

### **JEP 309: Dynamic Class-File Constants 类文件新结构**

属于JVM层面的优化，简单了解即可。下面是搜集到的的一些国外网站的解释：

> Dynamic class-file constants
	JEP 309 extends the existing Java class-file format, creating CONSTANT_Dynamic. This is a JVM feature and doesn't depend on the higher software layers. The loading of CONSTANT_Dynamic delegates the creation of a bootstrap method. When comparing it to the invokedynamic call, you will see how it delegates linking to a bootstrap method.
	One of the main goals of dynamic class-file constants is to make it simple to create new forms of materializable class-file constants, which provides language designers and compiler implementers with broader options for expressivity and performance. This is achieved by creating a new constant pool form that can be parameterized using a bootstrap method with static arguments.

下面是简单概括和了解：

动态类文档常量： JEP 309扩展了现有的Java类文档格式，创建了CONSTANT_Dynamic。这是JVM 功能，不依赖于更高的软件层。加载CONSTANT_Dynamic委派引导方法的创建。将其与动态调用进行比较时，将看到它如何委托链接到引导方法。动态类文档常量的主要目标之一，是使创建可具体化的类文档常量的新形式**变得简单**，这为语言设计者和编译器实现者提供了更广泛的表达性和性能选项。

这是通过创建一个新的常量池形式来实现的，该表单可以使用带有静态参数的引导方法进行参数化。

> 其目标是降低开发新形式的可实现类文件约束带来的成本和干扰

这里的介绍还是不是很懂，所以我又找了一篇英文博客，它阐述了从JDK7出现的invokeDynamic指令到JDK11的新指令出现的含义。

[动手 Java 11 的常量动态 - Java 代码极客 - 2023 (javacodegeeks.com)](https://www.javacodegeeks.com/2018/08/hands-on-java-constantdynamic.html)

Java的指令集新增基本都是大动作，作为对比个人联想到JDK7出现的新指令**invokedynamic**，周志明的《深入理解JVM虚拟机》中介绍了这个指令，并且JDK8以此为基础实现了Lambada语法。

下面是原书中有关invokedynamic的介绍：

> invokedynamic指令是在JDK 7时加入到字节码中的，当时确实只为了做动态语言（如JRuby、 Scala）支持，Java语言本身并不会用到它。而到了JDK 8时代，Java有了Lambda表达式和接口的默认方 法，它们在底层调用时就会用到invokedynamic指令，这时再提动态语言支持其实已不完全切合，我们 就只把它当个代称吧。

所以从JDK11的这一次变动来看，又是一次类似 invokedynamic 的大更新？

### **JEP 323: Local-Variable Syntax for Lambda Parameters**

注意是“语法糖”，实际Class字节码中会被识别为正确的数据类型，并不是变为弱类型语言了，Java依旧是**强类型语言**。

这个改进主要目的实际上是侧重于Lambada的缺陷改善，下面通过实际代码演示进行介绍：

```java

var javastack = "javastack";

System.out.println(javastack);
```

局部变量类型推断指的是左边的类型直接使用 var 定义，而不用写具体的类型，编译器能根据右边的表达式自动推断类型，如上面的 String 。

```java
var javastack = "javastack";
```

就等于：

```java
String javastack = "javastack";
```

var的变量还对于Lambada语法进行了更多改进，**声明隐式类型的lambda表达式的形参时允许使用var**。使用var的好处是在使用lambda表达式时**给参数加上注解**。

比如我们常见的隐式类型的写法如下，JDK11之前都只能是这样的“隐式类型”写法：

```java
(x, y) -> x.process(y)
```

有声明var之后，从表面上看可以让变量“可读性”变强了一点：

```java
(var x, var y) -> x.process(y)
```

实际上有了“变量类型”之后，之前Java版本无法给Lambada表达式的隐式变量进行标记的问题就得到解决：

```java
(@Deprecated var x, @Nullable var y) -> x.process(y)
```

另外这里扩展一下JDK的**非Lambada的声明隐式类型**，这是属于J**ava10**出现的东西，具体含义下面的代码一看便知：

```java
var foo = new Foo(); 
for (var foo : foos) { ... } 
try (var foo = ...) { ... } catch ...
```

> 这样的写法可以使得我们改变对象的时候更小概率改动遍历的代码，比如之前需要使用DtoA做遍历，现在里面的数据包装为Dtob了，这时候只需要动for循环里面的内容即可。

### JEP 329: ChaCha20 and Poly1305 Cryptographic Algorithms 实现RFC7539中指定的ChaCha20和Poly1305两种加密算法, 代替RC4

阅读这一节内容建议读者先有个印象： Java 11（2018年9 月 26 日） 和  TLS1.3 标准发布（2018年12月20日 · TLS1.3 正式版发布）

- **ChaCha20**： 是 Google 设计的另一种加密算法，密钥长度固定为 256 位，纯软件运行性能要超过 AES，在ARMv8引擎升级之后，AES也飞速提升导致ChaCha20被反超。所以现在来看虽然优势不大，但是依然不错的一个算法。  
- **Poly1305**：是由Daniel Bernstein创建的消息身份验证代码（MAC）。Poly1305 采用 32 字节的一次性密钥和一条消息来生成 16 字节的标记。然后使用该标记对消息进行身份验证。每当Poly1305用作MAC算法时，都需要为完整性密钥分配256位。（引自：[Poly1305](https://www.poly1305.com/)）

> 题外话：为什么ChaCha20和Poly1305在TLS1.3中合并为一个算法 ChaCha20-Poly1305？
> 其实很好理解，就好比两个运动员，一个专注长跑，另一个擅长短跑，一合体不就“无敌”了么， ChaCha20-Poly1305就是两个算法取长补短的效果，并且单独分开也是目前非常安全的算法。

RFC7539 也就是IETF制定的TLS1.2的协议标准名称。RC4在很早之前就已经被证实存在漏洞，后面已经被废弃了。

看到这部分的时候，个人联想到之前学习的[[HTTP 面试题 - TLS1.3 初次解读]]的内容。TLS1.3不是把这两个合并成ChaCha20-Poly1305了么，并且它是TLS1.3 IETF指定的AEAD算法，JDK又是如何看待这个问题的呢？

个人抱着这个疑问查阅了JDK官方issue的，最终在下面的连接中找到了答案：

[[JDK-8140466] ChaCha20 and Poly1305 TLS Cipher Suites - Java Bug System (openjdk.org)](https://bugs.openjdk.org/browse/JDK-8140466)

```java
Fix request (11u)

I would like to backport this to 11u because of Chacha20 and Poly1305 cipher suite SHOULD be implemented for TLSv1.3 according to rfc8446

Original patch applies almost clean except for the CipherSuite.java test - the list of cipher suites was reordered by JDK-8210632
Also, CheckCipherSuites.java and CipherSuitesInOrder.java tests are updated to support CHACHA20 cipher suites.

RFR: https://mail.openjdk.java.net/pipermail/jdk-updates-dev/2021-June/006644.html
```

翻译过来：

> 我想将其向后移植到 11u，因为 Chacha20 和 Poly1305 密码套件应该根据 rfc8446 为 TLSv1.3 实现 原始补丁（差不多）可以正常应用，除了 CipherSuite.java 测试 - 密码套件列表由 JDK-8210632 重新排序 此外，CheckCipherSuites.java 和 CipherSuitesInOrder.java 测试也进行了更新，以支持 CHACHA20 密码套件。

当然顺带也查到了JDK代码改动提交地址，感兴趣代码改动细节可以看这一段代码：[https://hg.openjdk.org/jdk/jdk/rev/720fd6544b03](https://hg.openjdk.org/jdk/jdk/rev/720fd6544b03)

ChaCha20-Poly1305 算法，它是TLS 1.3 支持的AEAD算法。

最后是JDK的一段案例程序：

```java
  
/**  
 * XECPublicKey 和 XECPrivateKey  
 * RFC7748定义的秘钥协商方案更高效, 更安全. JDK增加两个新的接口  
 */  
public class XECPublicKeyAndXECPrivateKeyTest {  
    public static void main(String[] args) throws NoSuchAlgorithmException, InvalidAlgorithmParameterException, InvalidKeySpecException, InvalidKeyException {  
        generateKeyPair();  
    }  
  
    private static void generateKeyPair() throws NoSuchAlgorithmException, InvalidAlgorithmParameterException, InvalidKeySpecException, InvalidKeyException {  
        KeyPairGenerator kpg = KeyPairGenerator.getInstance("XDH");  
        NamedParameterSpec paramSpec = new NamedParameterSpec("X25519");  
        kpg.initialize(paramSpec);  
        KeyPair kp = kpg.generateKeyPair();  
        PublicKey publicKey = generatePublic(paramSpec);  
        generateSecret(kp, publicKey);  
    }  
  
    private static void generateSecret(KeyPair kp, PublicKey pubKey) throws InvalidKeyException, NoSuchAlgorithmException {  
        KeyAgreement ka = KeyAgreement.getInstance("XDH");  
  
        ka.init(kp.getPrivate());  
  
        ka.doPhase(pubKey, true);  
  
        byte[] secret = ka.generateSecret();  
        // [B@10a035a0  
        System.out.println(secret);  
    }  
  
    private static PublicKey generatePublic(NamedParameterSpec paramSpec) throws NoSuchAlgorithmException, InvalidKeySpecException {  
        KeyFactory kf = KeyFactory.getInstance("XDH");  
  
        BigInteger u = new BigInteger("111");  
  
        XECPublicKeySpec pubSpec = new XECPublicKeySpec(paramSpec, u);  
  
        PublicKey pubKey = kf.generatePublic(pubSpec);  
        return pubKey;  
    }  
}
```


### JEP 332: Transport Layer Security (TLS) 1.3

实际情况可以看看曾经负责过Java的安全开发工作的大佬文章：

[An example of TLS 1.3 client and server on Java | The blog of a gypsy engineer](https://blog.gypsyengineer.com/en/security/an-example-of-tls-13-client-and-server-on-java.html)。

个人曾经在[[HTTP 面试题 - TLS1.3 初次解读]]对这位大佬做了一些简单阐述，并且建议看这部分内容前对于TLS1.3一定的了解，可能会有更深的印象。

根据作者所说就是Java11实现是实现了TLS1.3，但是是个“**残血版**”，比如不支持下面的特性：

-   0-RTT data 0-RTT 数据
-   Post-handshake authentication 握手后认证
-   Signed certificate timestamps (SCT) 签名证书时间戳 (SCT)
-   ChaCha20/Poly1305 cipher suites ([targeted to Java 12](https://bugs.openjdk.java.net/browse/JDK-8140466)) ChaCha20/Poly1305 密码套件（[针对 Java 12](https://bugs.openjdk.java.net/browse/JDK-8140466)）
- x25519/x448 elliptic curve algorithms x25519/x448 椭圆曲线算法（[针对 Java 12](https://bugs.openjdk.java.net/browse/JDK-8171279)）
- edDSA signature algorithms edDSA 签名算法（[针对 Java 12](https://bugs.openjdk.java.net/browse/JDK-8166596)）

但是从好的方面来看，Java11已经完全具备兼容TLS1.3的条件，因为这些不支持的仅仅是小部分内容，实际上大部分TLS1.3指定的最新加密套件都是支持的，使用者只需要改改“静态参数”，即可完成TLS1.2到TLS1.3的升级兼容。

> 提示：为了防止误解，这里个人有必要补充一下。ChaCha20/Poly1305 并不是说JDK12才真正实现ChaCha20和Poly1305，`ChaCha20/Poly1305`不是ChaCha20+Poly1305，虽然本质上是融合到一块，但是实际上一个全新的算法。

另外这一波也怪不到Oracle，谁让TLS1.3比JDK11晚发布3个月呢。

### JEP 321: HTTP Client (Standard) 标准Java异步HTTP客户端

这是自JDK9被引进孵化，直到JDK11才正式可用的处理 HTTP 请求的的 HTTP Client API，如小节标题所说，它支持同步和异步发送。

因为是跨越多个大版本的改建，Java11标准Java异步HTTP客户端最终被叫做 HTTP/2 Client。它定义了一个全新的实现了HTTP/2和WebSocket的HTTP客户端API，并且可以取代 HttpURLConnection。

过去HttpURLConnection 被人诟病的点如下：

- API 早于HTTP1.1 过于抽象。（HTTP1.0是草稿协议，没有被标准化）
- API 难懂，编写的代码难以维护。
- 过度设计，很多支持的协议到现在基本只剩下HTTP。


> java.net 包中可以找到这些 API，但是我并没有看到作者是谁=-=。

```java
public class JdkHttpClient {  
  
    public static void main(String[] args) throws IOException, InterruptedException {  
        var request = HttpRequest.newBuilder()  
                .uri(URI.create("https://cn.bing.com/?mkt=zh-CN"))  
                .GET()  
                .build();  
        var client = HttpClient.newHttpClient();  
// 同步  
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());  
        System.out.println(response.body());  
// 异步  
        client.sendAsync(request, HttpResponse.BodyHandlers.ofString())  
                .thenApply(HttpResponse::body)  
                .thenAccept(System.out::println);  
    }/**  
        截取部分内容：  
  
     //]]></script><script type="text/rms">//<![CDATA[  
     var sj_appHTML=function(n,t){var f,e,o,r,i,s,h;if(t&&n){var c="innerHTML",l="script",a="appendChild",v="length",y="src",p=sj_ce,u=p("div");if(u[c]="<br>"+t,f=u.childNodes,u.removeChild(f[0]),e=u.getElementsByTagName(l),e)for(o=0;o<e[v];o++)r=p(l),i=e[o],i&&(r.type=i.type=="module"||i.type=="importmap"?
     Process finished with exit code 0  
     */}
```

JDK官方给JDK引入HTTP Client API了，以后还有必要用 Apache 的 HttpClient 工具包么？

### JEP 331: Low-Overhead Heap Profiling 低成本的堆分析

官方说明：[JEP 331: Low-Overhead Heap Profiling (openjdk.org)](https://openjdk.org/jeps/331)

> Provide a low-overhead way of sampling Java heap allocations, accessible via JVMTI.

官方的介绍是说通过**JVMTI**的**sampling**提供低堆内存开销的分析工具。

为什么有了如Java Flight Recorder、YourKit和VisualVM等非常实用的JVM分析工具，JDK官方还需要提供接口扩展堆分析的手段呢？

> One piece of information that is lacking from most of the existing tooling is the call site for **particular allocations**. Heap dumps and heap histograms do not contain this information. 

关键原因在于大多数的分析工具缺少`particular allocations`，这里的特定分配在个人的理解是“堆外”内存的分配，因为这部分是超出JVM管辖范围内直接通过操作系统底层往物理内存分配。

> 现代的消息中间件Kafka、RocketMq等底层大量使用了**页缓存**这个东西，并且进行了大量的直接内存分配，不巧的是这部分内容刚好是JVM堆内存分析工具的痛点。所以JDK开发扩展这些接口也是时代发展的趋势之一。


> This proposal mitigates these problems by providing an extensible JVMTI interface that allows the user to define the sampling interval and returns a set of live stack traces.

通过提供一个可扩展的JVMTI接口来缓解这些问题，该接口允许用户定义采样间隔并返回一组实时堆栈跟踪。

因为用的是JVM对外接口，所以这个接口是C++写的，在`Use-case example`里面有很多的接入案例代码，如果不是要开发JVM堆分析工具，我们了解到这就可以收住了。

低成本的堆分析主要目标：
- 开销低，可以持续默认启用。
- 可以通过一个定义明确的程序化接口访问。
- 可以对所有分配进行采样（即，不限于在某一特定堆区域的分配或以某一特定方式的分配）。
- 可以以**独立**于实现的方式进行定义（即，不依赖于任何特定的GC算法或虚拟机实现），并且可以提供关于存活Java对象和垃圾Java对象的信息。

最终的实现如下：

 - 提供用于生产和消费数据作为事件的API
- 提供缓存机制和二进制数据格式
- 允许事件配置和事件过滤
- 提供OS,JVM和JDK库的事件


## 新增内容


### JEP 330: Launch Single-File Source-Code Programs 启动单一文件的源代码程序

提案愿意是：增强java启动器支持运行单个java源代码文件的程序。这是什么意思？在我们的认知里面，要运行一个 Java 源代码必须先编译，再运行两步执行动作。JDK11简化了允许java程序编译运行的过程。

这个功能是免去了Java初学者需要javac和Java运行class文件的步骤，直接java 类名即可编译运行，但是需要注意这种增强需要满足一定条件：

- 第一点：执行源文件中的第一个类，第一个类必须包含main方法，比如下面的代码main方法在Test2当中就无法使用此特性：

```java
class Test1 {  
    public void otherMethod(){  
          
    }  
}  
  
class Test2 {  
    public static void main(String[] args) throws IOException {  
          
    }  
}
```

- 第二点：不可以使用别源文件中的自定义类, 但是当前文件中的自定义类是可以使用的。

到JDK10为止，Java启动器能以三种方式运行，JDK11补齐了另一块短板：

1.  启动一个class文件；
2.  启动一个JAR中的main方法类；
3.  启动一个模块中的main方法类；
4. 启动java命令直接运行源码级别的Java文件（JDK11）

> 补充：只要是Java程序，无论多复杂，最终入口一定是main()。 

### JEP 327: Unicode 10

Unicode 10 增加了8518个字符， 总计达到了136690个字符，并且增加了4个脚本，同时还有56个新的emoji表情符号。

参考：[http://unicode.org/versions/Unicode10.0.0/](https://link.zhihu.com/?target=http%3A//unicode.org/versions/Unicode10.0.0/)

### JEP 328: Flight Recorder

英文名称直译过来是“飞行记录仪”，没错就是我们日常生活中的黑匣子。

在JDK11之前是一个商业版的特性，在java11当中开源出来，它可以导出事件到文件中，之后配合**Java Mission Control**来分析。

使用Flight Recorder 有两种方式：
- 应用启动时配置java -XX:StartFlightRecording
```java
-XX: +StartFlightRecording=filename=<path>, duration=<time>
```
- 使用jcmd来录制
	- `$ jcmd <pid> JFR.start`
	- `$ jcmd <pid> JFR.dump filename=recording.jfr`
	- `$ jcmd <pid> JFR.stop`
```java
Jcmd <pid> JFR.start filename=<path> duration=<time>
```

扩展：[Java Flight Recorder - Javatpoint](https://www.javatpoint.com/java-flight-recorder)


感兴趣可以看看下面的文章介绍的简单的JFR使用教程，但是注意是JDK8 的：

[工具篇-性能分析工具JFR和JMC - 麦奇 (mikeygithub.github.io)](https://mikeygithub.github.io/2021/05/14/yuque/ligt41/)

#### 实战 JFR

首先是编写一段JFR的测试案例代码：

```java
/**  
 * JFR 的测试程序  
 * 需要使用JDK11以上的版本  
 * <p>  
 * src/main/java/com/zxd/interview/jfr/JfrTestApplication.java  
 */
public class JfrTestApplication {  
  
    public static void main(String[] args) {  
        List<Object> items = new ArrayList<>(1);  
        try {  
            while (true){  
                items.add(new Object());  
            }  
        } catch (OutOfMemoryError e){  
            System.out.println(e.getMessage());  
        }  
        assert items.size() > 0;  
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {  
            System.out.println(e.getMessage());  
        }  
    }  
}
```

为了编译成功需要删除中文内容：

```java
/**  
 * JFR 的测试程序  
 * 需要使用JDK11以上的版本  
 *  
 * src/main/java/com/zxd/interview/jfr/JfrTestApplication.java */
 
```

运行之后发现报错，这是因为中文导致的问题，这里把中文注释掉之后就OK了。

```java
D:\adongstack\project\interview>javac -d out -sourcepath src/main src/main/java/com/zxd/interview/jfr/JfrTestApplication.java
src\main\java\com\zxd\interview\jfr\JfrTestApplication.java:7: error: unmappable character (0x8F) for encoding GBK
 * JFR 鐨勬祴璇曠▼搴?
              ^
src\main\java\com\zxd\interview\jfr\JfrTestApplication.java:8: error: unmappable character (0x80) for encoding GBK
 * 闇?瑕佷娇鐢↗DK11浠ヤ笂鐨勭増鏈?
    ^
src\main\java\com\zxd\interview\jfr\JfrTestApplication.java:8: error: unmappable character (0xAC) for encoding GBK
 * 闇?瑕佷娇鐢↗DK11浠ヤ笂鐨勭増鏈?

```

通过下面的命令执行，编译JfrTestApplication：

```java
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:StartFlightRecording=duration=200s,filename=flight.jfr -cp ./out/ com/zxd/interview/jfr/JfrTestApplication
```

运行结果：

```java
D:\adongstack\project\interview>java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:StartFlightRecording=duration=200s,filename=flight.jfr -cp ./out/ com/zxd/interview/jfr/JfrTestApplication

Java HotSpot(TM) 64-Bit Server VM warning: Ignoring option UnlockCommercialFeatures; support was removed in 11.0
Started recording 1. The result will be written to:
D:\adongstack\project\interview\flight.jfr
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
[70.761s][warning][jfr,system,event] Exception occured during execution of period hook for jdk.ThreadContextSwitchRate(345)

```

因为程序执行的是一个“死循环”，安静等待执行死循环直到OOM即可。执行完成之后，在Idea的项目根路径当中会出现下面的文件内容：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313213708.png)

这个文件要如何阅读？这里要用JDK的另一个工具**JMC**。令人遗憾的是，JFR为JDK自带的工具，但是JMC需要我们自己去Oracle官网下载（JMC被官方从JDK中移除）。

#### JMC安装

下载链接：[https://www.oracle.com/java/technologies/javase/products-jmc8-downloads.html](https://www.oracle.com/java/technologies/javase/products-jmc8-downloads.html)

这里直接下载当前的最新版本即可。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313213906.png)

Windows平台直接打开程序：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313214004.png)

看到下面的界面说明可以正常使用：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313214107.png)


#### Java Mission Control 分析代码

进入主界面，我们直接打开刚刚生成的jfr文件：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313214128.png)

首先是一份报告，简单介绍了jfr的情况：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313215717.png)

```
Application efficiency was affected by halts

The highest ratio of application halts to execution time was 84.7 % during 2023/3/13 下午9:11:53.000 – 下午9:12:53. 2.59 % of the halts were for reasons other than GC. The halts ratio for the entire recording was 86.6 %. 2.21 % of the total halts were for reasons other than GC. Application halts are often caused by garbage collections, but can also be caused by excessive thread dumps or heap dumps. Investigate the VM Operation information and possibly the safepoint specific information.
```

报告主要说明了整个应用程序大部分的时间都是在进行停顿，也就是对象的分配速度远远高于垃圾回收速度，垃圾收集器长时间工作而出现停顿和等待。

因为我们是while的死循环程序，可以明显CPU使用率逐渐飙升。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313220008.png)

我们可以阅读“方法调用的统计信息”来查看整个死循环的过程中哪些方法被大量调用：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313215645.png)

通过“方法”板块我们可以发现示例程序的另一个缺点：方法`java.util.ArrayList.grow（int）`已被调用147次，在每次没有足够的空间添加对象时扩大数组容量。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230313215544.png)

我们可能会看到许多其他有用的信息：

-   有关创建对象的统计信息，当垃圾回收器创建和销毁这些对象时
-   有关线程年表的详细报告，它们何时被锁定或处于活动状态
-   应用程序正在执行哪些 I/O 操作

### JEP 318: Epsilon: A No-Op Garbage Collector Epsilon 垃圾收集器

首先吸引我的是**Epsilon**这个单词，和CMS、G1这种带有含义的垃圾收集器名称一样，Epsilon 也有特殊的含义。

这个希腊字母同时也是英语音标`ɛ`的来源，下面的介绍引自：[Ε - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/zh-cn/%CE%95)

> Epsilon（大写 Ε、小写 ε 或 ϵ；希腊语：έψιλον；中文音译：伊普西龙、厄普西隆、艾普西龙、艾普塞朗），是第五个希腊字母。Epsilon（ἒ ψιλόν）即“e 简单的、e 单一的”的意思，这是为了与中世纪发生单元音化而变为同音的二合字母 ai（αι，古希腊语：[ai̯]）做区别，古典时期本来这个字母读作ei（εἶ，古希腊语：[êː]）；源自腓尼基字母 HeHe，又从 epsilon 发展出了拉丁字母 E 和西里尔字母 Е。在希腊数字系统中，E 表示 5。

Epsilon 垃圾收集器用法非常简单：

```text
-XX:+UseEpsilonGC
```

通常建议搭配参数`-XX:+UnlockExperimentalVMOptions` 使用。

> -XX:+UnlockExperimentalVMOptions ：
> 一般使用在一些低版本jdk想使用高级参数或者可能高版本有的参数情况；
        解锁实验参数，允许使用实验性参数，JVM中有些参数不能通过-XX直接复制需要先解锁，比如要使用某些参数的时候，可能不会生效，需要设置这个参数来解锁；

JDK的描述是开发只负责内存分配，其他事情一概不做处理的垃圾收集器，“不回收垃圾的垃圾收集器”看起来怪怪的，因此用途也比较特殊：

- 性能测试（它可以帮助过滤掉GC引起的性能假象）；
- 内存压力临界点测试（比如知道测试用例应该分配不超过1 GB的内存，我们可以使用-Xmx1g配置`-XX:+UseEpsilonGC`，如果违反了该约束，则会heap dump并崩溃）；
- 非常短的JOB任务（对于这种任务，接受GC清理堆那都是浪费空间）；
- VM接口测试；
- Last-drop 延迟&吞吐改进；

这里直接通过一个程序了解Epsilon垃圾收集器的效果：

```java
  
/**  
 * JDK11 新的垃圾收集器 Epsilon  
 * 需要配置启动参数：  
 * -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC  
 */public class EpsilonGcTest {  
  
    public static void main(String[] args) {  
        boolean flag = true;  
  
        List<Garbage> list = new ArrayList<>();  
  
        long count = 0;  
  
        while (flag){  
            list.add(new Garbage());  
            if(list.size() == 100000 && count == 0){  
                list.clear();  
                count++;  
            }  
        }  
        System.out.println("程序结束");  
    }  
}  
  
class Garbage{  
    int n = (int)(Math.random() * 100);  
  
    @Override  
    protected void finalize() throws Throwable {  
        System.out.println(this + "  : "+ n + "is dying");  
    }  
}
```

启动之前，不要忘了设置VM Option，如果看不到`VM Option`这一行，可以点击"Modify options"中添加VM option。

> PS：下面的环境变量和program arguments设置启动参数是无效的。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230316072908.png)

启动之后一会儿控制台会打印下面的内容并且结束。

```java
Terminating due to java.lang.OutOfMemoryError: Java heap space
```

如果是传统的垃圾收集器，那么输出的结果将会是`System.out.println(this + "  : "+ n + "is dying");`的内容，虽然整个程序依然会继续运行直到OOM，但是要花更多的时间等待。

```java
com.zxd.interview.epsilongctest.Garbage@21abe  : 59is dying
com.zxd.interview.epsilongctest.Garbage@6f6b18fc  : 43is dying
com.zxd.interview.epsilongctest.Garbage@701264d8  : 56is dying
com.zxd.interview.epsilongctest.Garbage@5380a941  : 52is dying
com.zxd.interview.epsilongctest.Garbage@43c78e65  : 72is dying
com.zxd.interview.epsilongctest.Garbage@745cd219  : 11is dying
com.zxd.interview.epsilongctest.Garbage@170b4cee  : 68is dying
com.zxd.interview.epsilongctest.Garbage@411725ef  : 22is dying
com.zxd.interview.epsilongctest.Garbage@3edc2f0e  : 4is dying
com.zxd.interview.epsilongctest.Garbage@2c82eed6  : 58is dying
com.zxd.interview.epsilongctest.Garbage@61a3bb94  : 71is dying
com.zxd.interview.epsilongctest.Garbage@19e3d212  : 70is dying
com.zxd.interview.epsilongctest.Garbage@46182a7f  : 19is dying
com.zxd.interview.epsilongctest.Garbage@7dd78834  : 56is dying
com.zxd.interview.epsilongctest.Garbage@1d4e301  : 68is dying
```

从上面的案例可以看到，有时候不回收垃圾也是有帮助的。

### JEP 333: ZGC: A Scalable Low-Latency Garbage Collector (可伸缩低延迟垃圾收集器)

这应该是JDK11最为瞩目的特性, 没有之一， 但是后面带了Experimental, 说明JDK11这一个版本**不建议用到生产环境**。（JDK13才算是真正完善），ZGC问世对外宣传如下：

-   GC暂停时间不会超过10ms；
-   即能处理几百兆小堆，也能处理几个T的大堆（OMG）；
-   和G1相比，应用吞吐能力不会下降超过15%；
-   为未来的GC功能和利用colord指针以及Load barriers优化奠定基础；
-   初始只支持64位系统；

现代系统可用内存不断增大，GC垃圾收集器同样需要不断进化，现代的应用追求低延迟高吞吐量，ZGC的特点正好切中了GC的痛点。

**ZGC的设计目标是：支持TB级内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于15%。**

美团在2020年有一篇关于ZGC的分析文章质量很高： [# 新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)，如果比较难理解，还可以看看这一篇文章：[# 理解并应用JVM垃圾收集器-ZGC](https://zhuanlan.zhihu.com/p/105921339)

ZGC的了解程度停留在简单理解理论概念即可，下面是从网络整理的资料归档：

#### GC术语

注意下面的并行、串行、并发更偏向GC的概念：

- 并行：可以存在多个GC线程同时进行工作，但是无法确定是否需要暂停用户线程
- 串行：指的是只有单个GC线程进行工作。注意串行也不一定需要暂停用户线程
- STW：用户线程暂停，此时只有GC线程进行工作。
- 并发：GC的并发指的是具备并发性，如果说垃圾收集器某些阶段是并发的，那么可以简单在某些阶段GC线程可以和应用程序线程同时进行，但是这需要十分复杂的设计防止用户线程操作导致GC工作无效。
- 增量阶段指的是某个阶段可以再运行一段时间之后提前中止。

#### 权衡

权衡主要是针对并行和并发阶段带来的问题，并行阶段如果GC占用过多的线程可能导致用户线程性能抖动，而并发阶段如果需要保证能同时处理GC的有效性和用户线程正常工作的问题。

#### 多层堆和压缩

多层堆指的是让JVM管理内存可以像高速缓存一样分为多级缓存存储对象，通过对象分类将频繁访问对象和很少使用对象分开管理的技术。该功能可以通过扩展指针元数据来实现，指针可以实现计数器位并使用该信息来决定是否需要移动对象到较慢的存储上。

压缩是指对象可以以压缩形式保存在内存中，而不是将对象重定位到较慢的存储层。获取压缩对象的时候通过读屏障将其解压并且重新分配。

#### 多重映射

这里不过多介绍计算机的虚拟内存和物理内存概念，简单知道操作系统通过**映射表**实现了物理内存和虚拟内存的映射，最终通过使用页表和处理器的内存管理单元（MMU）和转换查找缓冲器（TLB）来实现这一点。

而多重映射则指的是多个虚拟内存映射到同一块物理内存的技术。ZGC中使用三个映射来完成多重映射操作。

#### 读屏障

读屏障是每当应用程序线程从堆加载引用时运行的代码片段（即访问对象上的非原生字段（non-primitive field））：

> 读屏障是JVM向应用代码插入一小段代码的技术。当应用线程从堆中读取对象引用时，就会执行这段代码。需要注意的是，仅“**从堆中读取对象引用**”才会触发这段代码。

```java
void printName( Person person ) {
	// 这里触发读屏障
	String name = person.name;
	
	// 因为需要从heap读取引用
	
	//
	System.out.println(name); // 这里没有直接触发读屏障

}
```

如注释所说`String name = person.name;`这一行因为访问了对象非原生字段，将引用加载到本地的name变量，此时操作触发了读屏障。

如前面所说，读屏障是在返回对象引用之前“做手脚”，比较能想到的思路是引用中设置某些“标志位”的方式，但是ZGC实际用的是“测试加载的引用”检查是否满足条件，最后根据结果判断是否执行自己需要的特殊操作。

ZGC 最终选择在读屏障上动手脚，一部分原因是源自Shenadoah收集器的挑战，这个非JDK官方设计的垃圾收集器给Oracle不小的冲击。

#### ZGC标记

下面简单了解ZGC垃圾回收截断中的标记截断处理，ZGC的标记分为三个阶段：
- 第一阶段是STW，其中GC roots被标记为活对象。
	- 从roots访问的对象集合称为Live集。GC roots标记步骤非常短，因为roots的总数通常比较小。
- 第二阶段遍历对象图并标记所有可访问的对象。
	- 读屏障针使用掩码测试所有已加载的引用，该掩码确定它们是否已标记或尚未标记，如果尚未标记引用，则将其添加到队列以进行标记。
- 第三阶段是边缘情况的处理，这里会有一个非常短暂的STW操作。也是针对经过前两次标记之后依然没有标记完成的最终检查。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230317150355.png)


> GC roots类似于局部变量，通过它可以访问堆上其他对象。通过Gc root进行可达性分析查找出无法“到达”的对象则被标记为垃圾对象 

#### ZGC 重定位

ZGC在对象清理上采用和G1类似的思路，也就是**标记-复制**把活对象移动到另一处，然后将只存在垃圾对象的内存释放掉，但是实现方法上更为取巧。

ZGC首先将堆分成许多页面，重定位开始的时候，会挑选一组需要重定位对象的页面，为了保证挑选准确此时需要**短暂STW**将该集合中root对象的引用映射到新位置。

移动root后下一阶段是**并发重定位**，此时GC线程遍历重定位集并重新定位其包含的页中所有对象，如果应用线程访问到这些需要重定位对象，则通过转发引用的方式重新定位读取新位置。

这里需要注意读屏障并不能完全保证所有指向旧对象的引用转到重新定位的地址上，在并发的情况下GC需要反复多次检查是否有引用指向旧的位置。

重定位的操作代价十分高昂，为了提高效率会和GC周期的标记阶段一起并行执行，GC周期标记阶段遍历对象对象图的时候，如果发现未重映射的引用则将其重新映射，然后标记为活动状态。

> 这两步可以抽象认为是一个人负责干活，另一个人负责把前面干活遗留的工作进行检查和标记。

> 用户进程读取对象重新定位就是通过**读屏障**实现的。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230317151934.png)

#### ZGC 表现

ZGC的SPECjbb 2015吞吐量与Parallel GC（优化吞吐量）大致相当，但平均暂停时间为1ms，最长为4ms。 与之相比G1和Parallel有很多次超过200ms的GC停顿。

#### ZGC的状态

ZGC 依然有很多亟待解决的问题，以G1为例从发布到支持之间超过3年，ZGC最终到JDK13被正式对外使用。

#### 小结

从整体上看，ZGC的垃圾收集将所有的暂停控制只依赖于GC roots集合上，将停顿控制在一处以达到更好的GC效果。

标记垃圾对象的过程中，标记阶段处理标记终止的最**后一次暂停**是唯一的例外，但是它是增量的（不一定执行），如果超过GC时间预算，那么GC将恢复到并发标记，直到再次尝试。


## 删除内容

### JEP 320: Remove the Java EE and CORBA Modules

Java EE和CORBA两个模块在JDK9中已经标记"deprecated"，在JDK11中正式移除。这部分内容基本上没有Java开发者接触到（连网络搜索都不会碰到）所以**忘记**即可。

JavaEE由4部分组成：

-   JAX-WS (Java API for XML-Based Web Services),
-   JAXB (Java Architecture for XML Binding)
-   JAF (the JavaBeans Activation Framework)
-   Common Annotations.

看起来比较多，但是这个特性和JavaSE关系不大。并且Maven也提供了JavaEE（[http://mvnrepository.com/artifact/javax/javaee-api/8.0](https://link.zhihu.com/?target=http%3A//mvnrepository.com/artifact/javax/javaee-api/8.0) 第三方引用，所以移除的影响并不会很大。

至于CORBA，使用CORBA开发程序没有太大的兴趣。

> CORBA的XML相关模块被移除：
	[java.xml.ws](http://java.xml.ws),
	`java.xml.bind`，
	[java.xml.ws](http://java.xml.ws)，
	`java.xml.ws.annotation`，
	`jdk.xml.bind`，
	`jdk.xml.ws`
	最终保留：`java.xml`，`java.xml.crypto`，`jdk.xml.dom` 模块。

> 除此之外，还有：
> 
> [java.se.ee](http://java.se.ee)，
> `java.activation`，
>` java.transaction`
> 
> 被移除，但是java11新增一个`java.transaction.xa`模块



### JEP 335: Deprecate the Nashorn JavaScript Engine（弃用 Nashorn JavaScript 引擎）

废除**Nashorn javascript**引擎，在后续版本准备移除掉，有需要的可以考虑使用GraalVM

最终：**Nashorn JavaScript Engine 在 Java 15 已经不可用了**。几乎不会用到的东西，为了让读者略微了解一下，这里用JDK8的Nashorn简单做个演示。

```java
public class NashornTest {  
  
    public static void main(String[] args) throws ScriptException, FileNotFoundException {  
        // 直接使用  
        ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");  
        engine.eval("print('Hello World!');");  
  
        // 文件中运行  
        ScriptEngine engine2 = new ScriptEngineManager().getEngineByName("nashorn");  
        engine2.eval(new FileReader("E:\\adongstack\\project\\selfUp\\interview\\src\\main\\resources\\static\\js\\hello.js"));  
  
    }  
}
```

现代项目大多数都是前后端分离的，哪怕没有前后端分离也基本没有人会在后端写很多隐患的JS脚本，个人认为是这些原因导致的**Nashorn javascript**默默无闻的诞生和默默无闻的毁灭。

Nashorn javascript引擎更多了解可以看这篇：[https://mouse0w0.github.io/2018/12/02/Introduction-to-Nashorn/](https://mouse0w0.github.io/2018/12/02/Introduction-to-Nashorn/)

### JEP : 336 : Deprecate the Pack200 Tools and API 废弃Pack200和相关API

Pack200 是自Java5出现的 **压缩工具**，这个工具能对普通的jar文件进行高效压缩。

Pack200 合并原理是根据类的结构设计以及合并常量池的方式，最后去掉无用信息来实现高效压缩，注意这种压缩只能针对Java文件进行压缩，这种压缩方式是很有意义的，能达到jar包的10% - 40%的压缩率。

Java5中还提供了这一技术的API接口，可以将其嵌入到应用程序中使用。使用的方法很简单，下面的短短几行代码即可以实现jar的压缩和解压：

压缩

```java
Packer packer=Pack200.newPacker();
OutputStream output=new BufferedOutputStream(new FileOutputStream(outfile));
packer.pack(new JarFile(jarFile), output);
output.close();
```

解压

```java
Unpacker unpacker=Pack200.newUnpacker();
output=new JarOutputStream(new FileOutputStream(jarFile));
unpacker.unpack(pack200File, output);
output.close();
```

看了上面的介绍，读者可能会跟我有一样的疑问，感觉这**不是挺好的一东西么为啥要抛弃？** 还能结合SpringBoot的jar包进行压缩，为了进一步了解JDK为啥要在Java11移除，这里找到的JDK官方的说明：

[JEP 367: Remove the Pack200 Tools and API (openjdk.org)](https://openjdk.org/jeps/367)

我把JDK官方的说明部分拿过来了：

```
There are three reasons to remove Pack200:

1.  Historically, slow downloads of the JDK over 56k modems were an impediment to Java adoption. The relentless growth in JDK functionality caused the download size to swell, further impeding adoption. Compressing the JDK with Pack200 was a way to mitigate the problem. However, time has moved on: download speeds have improved, and JDK 9 introduced new compression schemes for both the Java runtime ([JEP 220](http://openjdk.java.net/jeps/220)) and the modules used to build the runtime ([JMOD](http://openjdk.java.net/jeps/261#Packaging:-JMOD-files)). Consequently, JDK 9 and later do not rely on Pack200; JDK 8 was the last release compressed with `pack200` at build time and uncompressed with `unpack200` at install time. In summary, a major consumer of Pack200 -- the JDK itself -- no longer needs it.
    
2.  Beyond the JDK, it was attractive to compress client applications, and especially applets, with Pack200. Some deployment technologies, such as Oracle's browser plug-in, would uncompress applet JARs automatically. However, the landscape for client applications has changed, and most browsers have dropped support for plug-ins. Consequently, a major class of consumers of Pack200 -- applets running in browsers -- are no longer a driver for including Pack200 in the JDK.
    
3.  Pack200 is a complex and elaborate technology. Its [file format](https://docs.oracle.com/en/java/javase/13/docs/specs/pack-spec.html) is tightly coupled to the [class file format](https://docs.oracle.com/javase/specs/jvms/se13/html/jvms-4.html#jvms-4.1) and the [JAR file format](https://docs.oracle.com/en/java/javase/13/docs/specs/jar/jar.html), both of which have evolved in ways unforeseen by JSR 200. (For example, [JEP 309](http://openjdk.java.net/jeps/309) added a new kind of constant pool entry to the class file format, and [JEP 238](http://openjdk.java.net/jeps/238) added versioning metadata to the JAR file format.) The implementation in the JDK is split between Java and native code, which makes it hard to maintain. The API in `java.util.jar.Pack200` was detrimental to the modularization of the Java SE Platform, leading to [the removal of four of its methods in Java SE 9](http://cr.openjdk.java.net/~iris/se/9/java-se-9-fr-spec/#APIs-removed). Overall, the cost of maintaining Pack200 is significant, and outweighs the benefit of including it in Java SE and the JDK.
```

一大坨论文一样的英文，这里抽取部分语句的关键点简单概括一下：

第一点：
- 说白了就是有新人干活还有力气，老东西赶紧退休。过去网络带宽资源吃紧加上网速很慢，超过56k的jar包要下半天，压缩处理能在一定程度上**治标**。但是现代的网络环境下个几十K的东西几乎都是瞬间完成了，然后**JDK9还引入了新的压缩方式**（关键）。

第二点：

- Pack200最后一点夹缝：作为浏览器开发插件压缩的生存空间也没了，现代浏览器插件有独有的开发方式，Java在这一块掀不起什么浪花。

第三点：

- `Pack200 is a complex and elaborate technology`。“Pack200是一项复杂而精细的技术”，我愿称之为高情商发言，低情商就是**臃肿又难用**。后面引经据典“批判了一番多么“精细”和“复杂”导致的问题。

总结：**网络下载速度的提升**以及**java9引入模块化系统**之后**不再依赖**Pack200，因此这个版本将其移除掉。


# JDK11 其他内容

## Optional 增强

新增了`empty()`方法来判断指定的 `Optional` 对象是否为空。

```
var op = Optional.empty();
System.out.println(op.isEmpty());//判断指定的 Optional 对象是否为空
```


## String 增强

```java
//判断字符串是否为空  
" ".isBlank();//true  
//去除字符串首尾空格  
" Java ".strip();// "Java"  
//去除字符串首部空格  
" Java ".stripLeading();   // "Java "  
//去除字符串尾部空格  
" Java ".stripTrailing();  // " Java"  
//重复字符串多少次  
"Java".repeat(3);             // "JavaJavaJava"  
//返回由行终止符分隔的字符串集合。  
"A\nB\nC".lines().count();    // 3  
"A\nB\nC".lines().collect(Collectors.toList());
```

## 移除项 

- 移除了`com.sun.awt.AWTUtilities`
- 移除了`sun.misc.Unsafe.defineClass`
- 使用`java.lang.invoke.MethodHandles.Lookup.defineClass`来替代
- 移除了`Thread.destroy()`以及` Thread.stop(Throwable)`方法
- 移除了`sun.nio.ch.disableSystemWideOverlappingFileLockCheck`、`sun.locale.formatasdefault`属性
- 移除了`jdk.snmp`模块
- 移除了 `javafx` ，openjdk 估计是从java10版本就移除了，但是oracle jdk10还尚未移除javafx，而java11版本则oracle的jdk版本也移除了javafx。
- 移除了**Java Mission Control**，从JDK中移除之后，需要自己单独下载

移除了这些**Root Certificates** ：
- Baltimore Cybertrust Code Signing CA
- SECOM 
- AOL and Swisscom

## 废弃项

- `-XX+AggressiveOpts`选项
- `-XX:+UnlockCommercialFeatures`
- `-XX:+LogCommercialFeatures`选项也不再需要

## G1 垃圾收集器升级

JDK11的G1垃圾收集齐相比于 JDK 8可以享受到：
- 并行的 Full GC
- 快速的 CardTable 扫描
- 自适应的堆占用比例调整（IHOP）
- 在并发标记阶段的类型卸载
....等等

这些都是针对 G1 的不断增强，串行 Full GC被常常诟病最终Oracle忍无可忍复用了CMS的并行GC代码给实现了（配置参数可以看出端倪，很多和CMS配置仅仅是换了个名字），最终是减少程序员的调优成本和调优时间。

# 附录

## 17 个 JEP（JDK Enhancement Proposals，JDK 增强提案）

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230311111312.png)


下面包含了这17个JEP的提案：

JEP 181: Nest-Based Access Control

**JEP 309: Dynamic Class-File Constants**

JEP 315: Improve Aarch64 Intrinsics

**JEP 318: Epsilon: A No-Op Garbage Collector**

JEP 320: Remove the Java EE and CORBA Modules

JEP 321: HTTP Client (Standard)

**JEP 323: Local-Variable Syntax for Lambda Parameters**

JEP 324: Key Agreement with Curve25519 and Curve448

JEP 327: Unicode 10

**JEP 328: Flight Recorder**

JEP 329: ChaCha20 and Poly1305 Cryptographic Algorithms

JEP 330: Launch Single-File Source-Code Programs

JEP 331: Low-Overhead Heap Profiling

**JEP 332: Transport Layer Security (TLS) 1.3**

**JEP 333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)**

JEP 335: Deprecate the Nashorn JavaScript Engine

JEP 336: Deprecate the Pack200 Tools and API


### Jshell（JDK9）

和Go和Python这种自带Shell的语言进行对比，Java的Shell直到Java9才出现，Java9引入了jshell这个交互性工具，让Java也可以像脚本语言一样来运行，可以从控制台启动 jshell。

一个编程语言本该有的东西，这里就不多介绍了。Jshell比较适合刚刚入门Java语言的学习者使用。


# 写在最后

这篇文章把大部分JDK11新特性简单分析了一遍，如果有错误的论述欢迎指正。JDK的新版本特性还是非常有意思的，虽然不一定能在工作中用上，但是了解这些特性跟进时代是作为Java开发者的基本操守。