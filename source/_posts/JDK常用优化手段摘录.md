---
title: JDK常用优化手段摘录
subtitle: 临时笔记摘录
author: lazytime
url_suffix: jdkyouhua
tags:
  - Java
categories:
  - Java
keywords: ['jdk']
description: Java JDK 一些优化的手段处理
copyright: true
date: 2022-01-28 23:38:26
---

# JDK常用优化手段	

# if/else嵌套问题：

问题诊断：

+ 利用`if/else`和缩进进行大量的逻辑判断
+ 逻辑重复，臃肿

反例：`if`条件仅仅包含一条，但是else内容很多，导致过多的缩进，比如像狭隘难这种代码，一旦多层嵌套很容易让逻辑变得复杂

```java
if (ArrayUtil.isEmpty(values)) {
    return BigDecimal.ZERO;
} else {
    String value = values[0];
    BigDecimal result = new BigDecimal(null == value ? "0" : value);

    for(int i = 1; i < values.length; ++i) {
        value = values[i];
        if (null != value) {
            result = result.subtract(new BigDecimal(value));
        }
    }

    return result;
}
```

<!-- more -->

优化手段：

​	从上面的代码我们可以发现，这个else完全可以忽略，多数时候时候缩进十分影响阅读体验的，然后我们可以发现



优化后：

```java
if (ArrayUtil.isEmpty(values)) {
    return BigDecimal.ZERO;
} 
// 上面return之后，下面的else显得十分多余
String value = values[0];
BigDecimal result = new BigDecimal(null == value ? "0" : value);

for(int i = 1; i < values.length; ++i) {
    value = values[i];
    if (null != value) {
        result = result.subtract(new BigDecimal(value));
    }
}

return result;

```

优化方式：

+ 一个if必定携带一个else，否则逻辑可能会出现漏洞
+ 能少用else就少用else，最好是使用if+return进行提前返回。
+ 注意if中如果存在对象的参数判断，需要确保对象是否会为**null**



# 复合操作的方法拆分：

​	复合操作常常是一种人脑无意识的操作，比如操作文件的时候，需要根据某个资源地址下载文件，或者从某一个网址下载资源到本地，程序员通常会在业务操作的时候带入相关的业务细节。导致后续的类似业务出现大量的相似代码：

常见误区：

1. 将读文件和操作文件内容混为一谈
2. 复合操作却起名为单一操作的起名，造成误解



反例：

​	下面的这段代码看似是一个操作，其实算是两个操作：解密和判断

```java
ExecuteResult decryptRes = JdkCommandUtil.decryptCmd(dccConfig, downloadFile, decAfterFile);
//解密不成功直接return
if (0 != decryptRes.getExitCode() && Ognl.isNotEmpty(decryptRes.getExecuteOut())) {
    logger.error("==>>==解密失败==");
    return;
}
```



​	现在将方法改为如下的方式：

```java
 protected boolean decAndCheckFile(TaskExceptionRequest taskExceptionRequest, String encFileName, String decAfterFileName) throws Exception {
        //文件解密
        ExecuteResult decryptRes = null;
        decryptRes = decryptCmdByYYYYMMDD(taskExceptionRequest, encFileName, decAfterFileName);
        //解密不成功直接return
        if (0 != decryptRes.getExitCode() && Ognl.isNotEmpty(decryptRes.getExecuteOut())) {
            logger.error("当前文件解密失败!");
            throw new DecodeFileException("当前文件解密失败");
        }
        return true;
    }
```

​	由于文件解密操作在多数情况下是不会出现问题的，而且一旦出现问题，基本上都是无法通过人工来进行手动修复的，所以我们修改为在解密失败之后，抛出我们提前定义好的自定义异常，通知客户端进行异常处理，当然这是一个运行时的异常，在多数情况下不会报错。同时将方法的名称改为了符合操作的名称。



# 变量定义：

基本遵循如下规则：

+ 如果可以，方法尽量定义为private，尽量少的暴露无用方法或者细节处理方法
+ 如果是常量，建议直接定义为 private static final 类型，一方面是静态final类型在虚拟机的编译阶段就可以知道静态final变量的值，无须进行初始化。另一方面，jvm本身会对final变量进行常量池的优化操作。



# 硬编码和魔法值：

​	和上面的变量定义一样，硬编码的问题更多一些，比如很容易在代码里面写上数字等内容。

​	而魔法值最常见的就是偷懒使用Map作为全局参数传递，见过不少项目都是这样使用的。



# 枚举使用：

枚举的基本用法可以用如下的套版

枚举建议如下：

+ 尽量使用基本类型，少使用包装类型（自动拆装箱的问题）
+ 尽量使用final定义，如果可以只提供get方法

```java
public enum RealRateEnum {


    VISA("1", "Visa"),
    MASTER("2", "Master"),
    JCB("3", "Jcb"),
    DC("4", "Dc"),
    AE("5", "Ae"),
    NONE("-1", "None");

    private final String code;
    private final String name;

    RealRateEnum(String code, String name) {
        this.code = code;
        this.name = name;
    }

    public String getCode() {
        return code;
    }

    public String getName() {
        return name;
    }
}
```

另外可以根据实际需要添加静态方法，比如根据code匹配获取name等



# 空指针规避

​	空指针是程序员经常犯的问题，下面就几个非常常见的场景防范方法

 + 字符串`equals`

   建议：使用`Objects.equals()`或者使用其他工具类方法替代，或者确保obj.equals(target)的obj对象不会为null

   ```java
   SftpExceptionEnum enumByCode = SftpExceptionEnum.getEnumByCode(exceptionRequest.getFileType());
   if (Objects.equals(enumByCode, SftpExceptionEnum.MERCHANT_INC_SERVICE)) {
       executeMerchIncTask(exceptionRequest, selectDefineFieldJson, merchIncEncFileName, merchIncDecAfterFileName);
   } else if (Objects.equals(enumByCode, SftpExceptionEnum.MERCHANT_FULL_SERVICE)) {
       executeMerchFullTask(exceptionRequest, selectDefineFieldJson, merchFullEncFileName, merchFullDecAfterFileName);
   }else{
       throw new UnsupportedOperationException("异常");
   }
   ```

   

 + 对象属性 == 操作

   建议：

    	1. 确保比较类型一致，比如最经典的Integer和int比较在超过127的时候为false的问题。
    	2. 使用框架工具类的equals() 进行替代
    	3. 使用`Objects.equals()`方法替代



 + 集合元素为null，或者框架查询返回空集合

   建议：

   	1. 元素null多数情况不常见，但是null的集合对象比较常见
   	2. 可以编写工具类方法对于集合的内容进行null排除，或者使用lambada表达式处理



 + map的元素值为null

   1. 使用mapUtil获取元素，不过需要注意mapUtil里面存在不少的"副作用"
   2. 
   
 + 类型强转

 + json转换为null

   