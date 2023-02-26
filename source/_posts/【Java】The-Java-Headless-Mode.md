---
title: 【Java】The Java Headless Mode
subtitle: 被忽略的优化参数
author: lazytime
url_suffix: headlessmode
tags:
  - Java
categories:
  - Java
keywords: Java被忽略的参数
description: Java被忽略的参数
copyright: true
date: 2023-02-27 07:37:31
---

# 原文

[https://www.baeldung.com/java-headless-mode](https://www.baeldung.com/java-headless-mode)
# 引言

这篇文章源自个人看到了Kafka的启动脚本中一个“奇怪”的参数：

```java
-Djava.awt.headless=true
```

拿去谷歌一下发现网上的描述都大差不差，这里找了baeldung（类似国外的菜鸟教程）中的一篇文章，本文内容来自于英文博客原文。

这篇文章介绍了 **-Djava.awt.headless** 参数的作用，网上大部分的资料都是说“为了**提高计算效率和适配性**我们可以使用这种模式，关闭图形显示等功能可以大大节省设备的计算能力，而且对一些本身没有相关显示设备的机器也能适配，程序也可以正常运行。”，个人认为这些理论内容不太能理解。

当然也有诸如**服务器没有显示屏什么的，你得告诉程序一声，你工作的地方没有这些设备**这种说法 ，为此找了一篇国外的博客介绍。

<!-- more -->

# 如何设置？

设置方式如下：

- 在system property中设置 _java.awt.headless_ 为 _true_。

SpringBoot的源码中可以找到类似的代码：

```java
private void configureHeadlessProperty() {
        System.setProperty("java.awt.headless", System.getProperty("java.awt.headless", Boolean.toString(this.headless)));
    }

```

- 启动脚本中进行设置`-Djava.awt.headless=true`：在Kafka的脚本当中存在类似的启动脚本。

```sh
KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true"
```

最后一个参数显示它使用headless模式。

- 在执行命令的时候动态添加`-Djava.awt.headless=true`，这种方式和脚本设置启动的方式类似。

# Headless 绕过重量级组件

如果一个带有GUI组件的代码在开和关Headless模式下运行分别会有什么不同的效果？

```java
@Test  
public void FlexibleApp() {  
    if (GraphicsEnvironment.isHeadless()) {  
        System.out.println("Hello World");  
    } else {  
        JOptionPane.showMessageDialog(null, " showMessageDialog Hello World");  
    }  
}
```

上面的代码如果关闭了Headless模式，则打印Hello World会变为图形化界面。

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230223072911.png)

如果开启Headless，则会打印在控制台。

# Headless Mode 在UI组件的应用案例

Java Headless Mode 的典型案例可能是使用图形转化器，我们有时候可能需要图形数据进行图像处理，但是不一定需要实际显示。

下面通过一个单元测试来模拟这些情况：

```java
  @Before  
    public void setUpHeadlessMode() {  
        // 通过注释掉下面的代码测试不同的效果  
//        System.setProperty("java.awt.headless", "true");  
    }  
  
    @Test  
    public void whenSetUpSuccessful_thenHeadlessIsTrue() {  
        boolean headless = GraphicsEnvironment.isHeadless();  
        Assert.assertTrue(headless);  
    }/*  
    测试通过  
    注释下面的代码之后，单元测试不通过  
    //        System.setProperty("java.awt.headless", "true");    */
```

使用awt的组件`java.awt.GraphicsEnvironment#isHeadless`，注意较高版本的JDK（例如 JDK11）中awk被直接干掉了，需要下载外部依赖导入才可以使用，建议选择JDK8以及以下的版本测试上面的程序。

上面的代码如果注释掉 headless模式，单元测试会直接不通过。下面简单构建了一个图形转化器：

```java
@Test  
public void whenHeadlessMode_thenImagesWork() {  
    boolean result = false;  
    try (InputStream inStream = HeadlessModeUnitTest.class.getResourceAsStream(IN_FILE);  
         FileOutputStream outStream = new FileOutputStream(OUT_FILE)) {  
        BufferedImage inputImage = ImageIO.read(inStream);  
        result = ImageIO.write(inputImage, FORMAT, outStream);  
    }  
  
    assertThat(result).isTrue();  
}
```

在接下来的这个例子中，我们可以看到所有字体的信息，包括字体的度量，也可以让我们使用。

```java
  @Test  
    public void whenHeadless_thenFontsWork() {  
        GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();  
        String fonts[] = ge.getAvailableFontFamilyNames();  
  
//        assertThat(fonts).isNotEmpty();  
  
        Font font = new Font(fonts[0], Font.BOLD, 14);  
        FontMetrics fm = (new Canvas()).getFontMetrics(font);  
  
//        assertThat(fm.getHeight()).isGreaterThan(0);  
//        assertThat(fm.getAscent()).isGreaterThan(0);  
//        assertThat(fm.getDescent()).isGreaterThan(0);  
    }
```

# HeadlessException

有些设备是需要外部设备支持的，否则会抛出下面的异常：

```java
Exception in thread "main" java.awt.HeadlessException
	at java.awt.GraphicsEnvironment.checkHeadless(GraphicsEnvironment.java:204)
	at java.awt.Window.<init>(Window.java:536)
	at java.awt.Frame.<init>(Frame.java:420)
```

可以使用Frame来进行验证：

```java
  
   @Test  
   public void whenHeadlessmode_thenFrameThrowsHeadlessException() {  
       Frame frame = new Frame();  
       frame.setVisible(true);  
       frame.setSize(120, 120);  
   }/*  
   在开关Headless模式后会有不同的结果  
   开启：通过      
   
   关闭  
   ava.awt.HeadlessExceptionat java.awt.GraphicsEnvironment.checkHeadless(GraphicsEnvironment.java:204)  
at java.awt.Window.<init>(Window.java:536)  
at java.awt.Frame.<init>(Frame.java:420)  
at java.awt.Frame.<init>(Frame.java:385)  
  
   */
```

作为一个经验法则，请记住，像Frame和Button这样的顶级组件总是需要一个交互式的环境，并且会抛出这个异常。然而，如果没有明确设置无头模式，它将被抛出，成为一个不可恢复的错误。

# 总结

通过代码和案例分析，我们大致了解Java Headless Mode模式是怎么一回事，说白了就是屏蔽掉外置设备比如GUI的额外开销，转而用程序自己去进行模拟。

Kafka设置这样的参数就是把性能发挥到机制，摈弃一切外部设备干扰，让服务器尽可能的通过自身程序模拟外部设备。

比如重量级组件控制台打印，在外部设计可以通过`JOptionPane`的GUI组件实现可视化效果，而Headless则是利用我们熟知的`System.out`控制台输入输出流完成打印功能的模拟。

以上就是关于 Java Headless Mode 的理解。


# 程序demo

本文的个人实验代码放到下面部分，文章提到的部分代码可能会无法编译通过（图形转化器的代码），个人理解代码意图之后就没有深究了，读者碰到报错问题忽略删除即可。

> PS：建议使用JDK8之前的版本，可以直接引入awt和swing的相关组件。

```java
  
import jdk.nashorn.internal.ir.LiteralNode;  
import org.junit.Assert;  
import org.junit.Before;  
import org.junit.Test;  
  
import javax.imageio.ImageIO;  
import javax.swing.*;  
import java.awt.*;  
import java.awt.image.BufferedImage;  
import java.io.FileOutputStream;  
import java.io.IOException;  
import java.io.InputStream;  
  
import static org.junit.Assert.assertThat;  
  
  
public class HandlessTest {  
  
    @Before  
    public void setUpHeadlessMode() {  
        // 通过注释掉下面的代码测试不同的效果  
        System.setProperty("java.awt.headless", "true");  
    }  
  
    @Test  
    public void whenSetUpSuccessful_thenHeadlessIsTrue() {  
        boolean headless = GraphicsEnvironment.isHeadless();  
        Assert.assertTrue(headless);  
    }/*  
    测试通过  
    注释下面的代码之后，单元测试不通过  
    //        System.setProperty("java.awt.headless", "true");    */  
  
    //@Test  
    //public void whenHeadlessMode_thenImagesWork() throws IOException {  
  
    //    boolean result = false;  
    //    try (InputStream inStream = HandlessTest.class.getResourceAsStream(IN_FILE);  
    //         FileOutputStream outStream = new FileOutputStream(OUT_FILE)) {  
    //        BufferedImage inputImage = ImageIO.read(inStream);  
    //        result = ImageIO.write(inputImage, FORMAT, outStream);  
    //   }  
    //    Assert.assertTrue(result);  
    //}  
  
//    @Test  
//    public void whenHeadlessMode_thenImagesWork() {  
//        boolean result = false;  
//        try (InputStream inStream = HeadlessModeUnitTest.class.getResourceAsStream(IN_FILE);  
//             FileOutputStream outStream = new FileOutputStream(OUT_FILE)) {  
//            BufferedImage inputImage = ImageIO.read(inStream);  
//            result = ImageIO.write(inputImage, FORMAT, outStream);  
//        }  
//  
//        assertThat(result).isTrue();  
//    }  
  
    @Test  
    public void whenHeadless_thenFontsWork() {  
        GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();  
        String fonts[] = ge.getAvailableFontFamilyNames();  
  
//        assertThat(fonts).isNotEmpty();  
  
        Font font = new Font(fonts[0], Font.BOLD, 14);  
        FontMetrics fm = (new Canvas()).getFontMetrics(font);  
  
//        assertThat(fm.getHeight()).isGreaterThan(0);  
//        assertThat(fm.getAscent()).isGreaterThan(0);  
//        assertThat(fm.getDescent()).isGreaterThan(0);  
    }  
  
    @Test  
    public void whenHeadlessmode_thenFrameThrowsHeadlessException() {  
        Frame frame = new Frame();  
        frame.setVisible(true);  
        frame.setSize(120, 120);  
    }/*  
    在开关Headless模式后会有不同的结果  
    开启：通过  
    关闭  
    ava.awt.HeadlessException   at java.awt.GraphicsEnvironment.checkHeadless(GraphicsEnvironment.java:204)   at java.awt.Window.<init>(Window.java:536)   at java.awt.Frame.<init>(Frame.java:420)   at java.awt.Frame.<init>(Frame.java:385)  
    */  
  
  
    @Test  
    public void FlexibleApp() {  
        if (GraphicsEnvironment.isHeadless()) {  
            System.out.println("Hello World");  
        } else {  
            JOptionPane.showMessageDialog(null, "showMessageDialog Hello World");  
        }  
    }  
  
  
  
}
```

# 参考资料

[https://www.jianshu.com/p/7248b3ff5ca7](https://www.jianshu.com/p/7248b3ff5ca7)

[https://www.baeldung.com/java-headless-mode](https://www.baeldung.com/java-headless-mode)](https://www.baeldung.com/java-headless-mode](https://www.baeldung.com/java-headless-mode))