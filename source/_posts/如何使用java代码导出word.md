---
title: 如何使用java代码导出word
subtitle: 参考了网上内容了解如何导出word内容
url_suffix: word-export
author: lazytime
tags:
  - Java
categories:
  - Java
keywords: ['java导出','word导出']
description: 如何使用java代码导出word
copyright: true
date: 2020-11-11 23:03:01
---

# 前言：

导出word的需求其实在日常工作中用到的地方还不少，于是想写一篇文章好好记录一下，在导出之前，需要了解一下关于浏览器如何处理servlet的后台数据。具体可以了解一下http通信下载行为在servlet的实现。

==导出的工具类代码来源于网络，如有侵权可以联系我删除文章==

**个人使用==ftl==作为word导出模板引擎，有很多模板引擎可以选，个人经过查阅资料发现ftl用的比较多，所以选择这一种**

<!-- more -->

## 码云地址：

文章牵扯代码比较多，如果要看具操作可以查看我自己瞎弄的一个码云地址：

https://gitee.com/lazyTimes/interview.git

## 效果演示：

给了一个测试页面，临时写了一些脚本，可以作为参考（后续会贴Html代码进去）

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111220825.png)

点击提交,导出内容, 导出`word`报告

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111220951.png)

导出之后，打开word内容为：

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111222011.png)



## 实现步骤 - 制作word模板

### 第一步 新建word，制作成果样板

将需要导出word的内容，先粘贴到一个新建的word文件里面

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111220620.png)

### 第二步 转存格式 -> xml

选择文件“另存为”，将格式设置为xml格式

![](https://gitee.com/lazyTimes/imageReposity/raw/master/jiuhe/20201106091518.png)

### 第三步 格式化文件

将文件放到`idea`或者支持格式化的软件里面，进行格式化，保存:

注意占位符要匹配

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111222316.png)

### 第四步：模板数据替换占位符

在word页面将需要导入数据的地方，替换占位符

> 需要注意内容处理的时候: ${ filename｝ 有可能被切割为多个部分，我们需要把多个切割部分，改为下面的样式

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20201111222554.png)

**一定记得所有的改动之后，马上打开xml格式的word，确认是不是改崩了**

上面的步骤完成，说明有一个word模板做好了



### 第五步：制作ftl文件，word模板成型

在项目里面新建一个ftl文件，同时需要在工具类中配置，同时把做好站位符操作的xml内容贴进去

## 代码实现 - 导出代码

1. 工具类的配置如下:

`WordGeneratorUtil.java`：

```java
/**
     * 模板常量类配置
     */
public static final class FreemarkerTemplate {
    public static final String REPORT = "report";
    public static final String REC_RECOMMEND = "recRecommend";
    // 增加你的模板文件名称:

}
```

在静态的代码块里面，需要注入对应的模板配置

```java
// 注意初始化要载入对应模板
allTemplates.put(FreemarkerTemplate.REPORT, configuration.getTemplate(FreemarkerTemplate.REPORT + ".ftl"));
allTemplates.put(FreemarkerTemplate.REC_RECOMMEND,configuration.getTemplate(FreemarkerTemplate.REC_RECOMMEND + ".ftl"));
```

2. 在配置完成之后，导出的时候就可以找到对应的文件了
3. 建立一个通用的导出方法:

```java
/**
     * 创建doc 文档
     * dataMap 数据，需要对应模板的占位符，否则会出错
     * @param dataMap 数据
     * @param wordName  word 报表的名称
     * @param freemarkerTemplateName  指定需要使用哪个freemarker模板
     * @return
     */
public static File createDoc(String freemarkerTemplateName, String wordName, Map<String, String> dataMap) {
    try {
        File f = new File(wordName);
        Template t = allTemplates.get(freemarkerTemplateName);
        // 这个地方不能使用FileWriter因为需要指定编码类型否则生成的Word文档会因为有无法识别的编码而无法打开
        Writer w = new OutputStreamWriter(new FileOutputStream(f), StandardCharsets.UTF_8);
        t.process(dataMap, w);
        w.close();
        return f;
    } catch (Exception ex) {
        ex.printStackTrace();
        throw new RuntimeException("生成word文档失败");
    }
}
```



### 工具类完整代码：

```java
package com.zxd.interview.export;


import freemarker.template.Configuration;
import freemarker.template.Template;
import org.springframework.util.CollectionUtils;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;

/**
 * <p>
 * 从网络上根据资料找到的一个工具类
 * 主要以freemarker 为核心的模板生成word文档的工具类
 * 这里默认配置了固定路径
 * 需要根据路径取到对应模板
 * 请求参数需要设置对应的模板名称
 * @author 
 * @className: WordGeneratorUtils
 * @description: 文档生成工具类
 * </p>
 * version: V1.0.0
 */
public final class WordGeneratorUtil {
    private static Configuration configuration = null;
    private static Map<String, Template> allTemplates = null;
    private static final String TEMPLATE_URL = "/templates";

    /**
     * 模板常量类配置
     */
    public static final class FreemarkerTemplate {
        public static final String Test = "test";
        public static final String REPORT = "report";
        public static final String REC_RECOMMEND = "recRecommend";

    }

    static {
        configuration = new Configuration(Configuration.VERSION_2_3_28);
        configuration.setDefaultEncoding("utf-8");
        configuration.setClassForTemplateLoading(WordGeneratorUtil.class, TEMPLATE_URL);
        allTemplates = new HashMap(4);
        try {
            // 注意初始化要载入对应模板
            allTemplates.put(FreemarkerTemplate.Test, configuration.getTemplate(FreemarkerTemplate.Test + ".ftl"));
            allTemplates.put(FreemarkerTemplate.REPORT, configuration.getTemplate(FreemarkerTemplate.REPORT + ".ftl"));
            allTemplates.put(FreemarkerTemplate.REC_RECOMMEND, configuration.getTemplate(FreemarkerTemplate.REC_RECOMMEND + ".ftl"));
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    private WordGeneratorUtil() {
        throw new AssertionError();
    }

    /**
     * 创建doc 文档
     * dataMap 数据，需要对应模板的占位符，否则会出错
     * @param dataMap 数据
     * @param wordName  word 报表的名称
     * @param freemarkerTemplateName  指定需要使用哪个freemarker模板
     * @return
     */
    public static File createDoc(String freemarkerTemplateName, String wordName, Map<String, String> dataMap) {
        try {
            File f = new File(wordName);
            Template t = allTemplates.get(freemarkerTemplateName);
            // 这个地方不能使用FileWriter因为需要指定编码类型否则生成的Word文档会因为有无法识别的编码而无法打开
            Writer w = new OutputStreamWriter(new FileOutputStream(f), StandardCharsets.UTF_8);
            t.process(dataMap, w);
            w.close();
            return f;
        } catch (Exception ex) {
            ex.printStackTrace();
            throw new RuntimeException("生成word文档失败");
        }
    }


}

```

### 调用层：

1. 在业务层，将需要导出的数据，根据占位符的i信息进行赋值，注意不能漏，否则导出之后的文件会打不开

```java
@Override
public File exportQualityStep4Word(WordReportDTO exportWordRequest) {
    Map<String, String> datas = new HashMap(QualityConstants.HASH_MAP_INIT_VALUE);
    //主标题
    datas.put("schoolName", exportWordRequest.getSchoolName());
    datas.put("title1", exportWordRequest.getBaseSituation());
    datas.put("title2", exportWordRequest.getLearningEnvRec());
    datas.put("title3", exportWordRequest.getLearningEnvPro());
    datas.put("title4", exportWordRequest.getDayLifeRec());
    datas.put("title5", exportWordRequest.getDayLifePro());
    datas.put("title6", exportWordRequest.getLearningActivityRec());
    datas.put("title7", exportWordRequest.getLearningActivityPro());
    datas.put("title8", exportWordRequest.getDevRecommend());

    datas.put("base64_1", exportWordRequest.getBase64_1());
    datas.put("base64_2", exportWordRequest.getBase64_2());
    datas.put("base64_3", exportWordRequest.getBase64_3());
    datas.put("base64_4", exportWordRequest.getBase64_4());
    datas.put("base64_5", exportWordRequest.getBase64_5());
    datas.put("base64_6", exportWordRequest.getBase64_6());


    //导出
    return WordGeneratorUtil.createDoc(WordGeneratorUtil.FreemarkerTemplate.REPORT,
                                       exportWordRequest.getWordName(),
                                       datas);
}
```

2. 下面是生成报表导出的基本操作，可以在用到的地方复制过去改动即可

```java
/**
 * 生成报告的导出报表操作
 *
 * @param request           request
 * @param response          响应数据
 * @param exportWordRequest 导出dto
 */
@PostMapping("/quality/exportword")
@ResponseBody
public void povertyExportWord(HttpServletRequest request, HttpServletResponse response,
                              WordReportDTO exportWordRequest) {

    File file = qualityReportService.exportQualityStep4Word(exportWordRequest);

    InputStream fin = null;
    OutputStream out = null;
    try {
        // 调用工具类WordGeneratorUtils的createDoc方法生成Word文档
        fin = new FileInputStream(file);

        response.setCharacterEncoding(QualityConstants.UTF_8);
        response.setContentType(QualityConstants.CONTENT_TYPE_WORD);
        // 设置浏览器以下载的方式处理该文件
        // 设置文件名编码解决文件名乱码问题
        //获得请求头中的User-Agent
        String filename = exportWordRequest.getWordName();
        String agent = request.getHeader(QualityConstants.USER_AGENT);
        String filenameEncoder = "";
        // 根据不同的浏览器进行不同的判断
        if (agent.contains(QualityConstants.MSIE)) {
            // IE浏览器
            filenameEncoder = URLEncoder.encode(filename, QualityConstants.UTF_8);
            filenameEncoder = filenameEncoder.replace("+", " ");
        } else if (agent.contains(QualityConstants.FIREFOX)) {
            // 火狐浏览器
            BASE64Encoder base64Encoder = new BASE64Encoder();
            filenameEncoder = "=?utf-8?B?" + base64Encoder.encode(filename.getBytes(QualityConstants.UTF_8)) + "?=";
        } else {
            // 其它浏览器
            filenameEncoder = URLEncoder.encode(filename, QualityConstants.UTF_8);
        }
        response.setHeader(QualityConstants.ACCESS_CONTROL_ALLOW_ORIGIN, "*");//所有域都可以跨
        response.setHeader(QualityConstants.CONTENT_TYPE, QualityConstants.CONTENT_TYPE_STEAM);//二进制  流文件
        response.setHeader(QualityConstants.CONTENT_DISPOSITION, "attachment;filename=" + filenameEncoder + ".doc");//下载及其文件名
        response.setHeader(QualityConstants.CONNECTION, QualityConstants.CLOSE);//关闭请求头连接
        //设置文件在浏览器打开还是下载
        response.setContentType(QualityConstants.CONTENT_TYPE_DOWNLOAD);
        out = response.getOutputStream();
        byte[] buffer = new byte[QualityConstants.BYTE_512];
        int bytesToRead = QualityConstants.NUM_MINUS_1;
        // 通过循环将读入的Word文件的内容输出到浏览器中
        while ((bytesToRead = fin.read(buffer)) != QualityConstants.NUM_MINUS_1) {
            out.write(buffer, QualityConstants.NUM_ZERO, bytesToRead);
        }

    } catch (Exception e) {
        throw new RuntimeException(QualityConstants.FARIURE_EXPORT, e);
    } finally {
        try {
            if (fin != null) {
                fin.close();
            }
            if (out != null) {
                out.close();
            }
            if (file != null) {
                file.delete();
            }
        } catch (IOException e) {
            throw new RuntimeException(QualityConstants.FARIURE_EXPORT, e);
        }
    }

}
```

### 导出实体dto

下面写了一个导出的实体dto，实体对象可以自己定制：

```java
package com.zxd.interview.dto;

/**
 * 测试使用的dto，用于封装导出word的对象
 *
 * @author zhaoxudong
 * @version 1.0
 * @date 2020/11/7 23:37
 */
public class TestReportDTO {

    /**
     * 测试
     */
    private String test0;

    /**
     * 测试
     */
    private String test1;

    /**
     * 测试
     */
    private String test2;

    /**
     * 测试
     */
    private String test4;

    /**
     * 测试
     */
    private String test5;

    /**
     * 测试
     */
    private String test6;

    /**
     * 报告名称
     */
    private String wordName;

    public String getTest0() {
        return test0;
    }

    public void setTest0(String test0) {
        this.test0 = test0;
    }

    public String getTest1() {
        return test1;
    }

    public void setTest1(String test1) {
        this.test1 = test1;
    }

    public String getTest2() {
        return test2;
    }

    public void setTest2(String test2) {
        this.test2 = test2;
    }

    public String getTest4() {
        return test4;
    }

    public void setTest4(String test4) {
        this.test4 = test4;
    }

    public String getTest5() {
        return test5;
    }

    public void setTest5(String test5) {
        this.test5 = test5;
    }

    public String getTest6() {
        return test6;
    }

    public void setTest6(String test6) {
        this.test6 = test6;
    }

    public String getWordName() {
        return wordName;
    }

    public void setWordName(String wordName) {
        this.wordName = wordName;
    }


    @Override
    public String toString() {
        return "TestReportDTO{" +
                "test0='" + test0 + '\'' +
                ", test1='" + test1 + '\'' +
                ", test2='" + test2 + '\'' +
                ", test4='" + test4 + '\'' +
                ", test5='" + test5 + '\'' +
                ", test6='" + test6 + '\'' +
                '}';
    }
}

```



### 常量配置模块:

个人很不喜欢硬编码这东西，又丑又难看，所以很多东西会用不可变对象替代.

```java
package com.zxd.interview.constant;

/**
 * 常量配置类
 *
 * @author zhouhui
 */
public class QualityConstants {

    /**
     * 质量检测 的督导事项id
     */
    public static final int EVENTID = 12;

    /**
     * 数字0
     */
    public static final int NUM_ZERO = 0;
    /**
     * 数字1
     */
    public static final int NUM_ONE = 1;

    /**
     * 数字2
     */
    public static final int NUM_TWO = 2;
    /**
     * 数字-1
     */
    public static final int NUM_MINUS_1 = -1;
    /**
     * 字节大小512
     */
    public static final int BYTE_512 = 512;
    /**
     * 500错误编码
     */
    public static final int CODE_500 = 500;
    /**
     * 500错误提示信息 - 状态非法
     */
    public static final String CODE_500_MSG_1 = "状态非法!";
    /**
     * 500错误提示信息 - 非督导用户不允许查看质量检测记录
     */
    public static final String CODE_500_MSG_2 = "非督导用户不允许查看质量检测记录!";
    /**
     * 500错误提示信息 - 这条质量监测已经完成！无法修改
     */
    public static final String CODE_500_MSG_3 = "这条质量监测已经完成！无法修改！";
    /**
     * 500错误提示信息 - 提交失败，材料上传不能为空
     */
    public static final String CODE_500_MSG_4 = "提交失败，材料上传不能为空";
    /**
     * 500错误提示信息 - 提交失败，请稍后重试或联系管理员
     */
    public static final String CODE_500_MSG_5 = "提交失败，请稍后重试或联系管理员！";
    /**
     * 500错误提示信息 - 提交失败，意见反馈不能为空
     */
    public static final String CODE_500_MSG_6 = "提交失败，意见反馈不能为空!";
    /**
     * 405错误编码
     */
    public static final int CODE_405 = 405;
    /**
     * 405错误提示信息 - 该信息只允许督导查看
     */
    public static final String CODE_405_MSG_1 = "该信息只允许督导查看!";
    /**
     * 200成功编码
     */
    public static final int CODE_200 = 200;
    /**
     * 200成功提示信息 - 该信息只允许督导查看
     */
    public static final String CODE_200_MSG_1 = "提交成功!";

    /**
     * 错误提示信息 - 尚未选择记录
     */
    public static final String DELETE_FAIRURE_MSG = "删除失败，尚未选择记录！";
    /**
     * 错误提示信息 - 尚未选择记录
     */
    public static final String NO_RECORD_SELECTED = "尚未选择记录！";
    /**
     * 字符编码utf-8
     */
    public static final String UTF_8 = "utf-8";
    /**
     * 默认pid
     */
    public static final int PID = 0;
    /**
     * 默认层级
     */
    public static final int DEFUALT_LAYER = 1;

    /**
     * 不适当最低得分
     */
    public static final Integer MIN_SCORE = 1;
    /**
     * 优秀最高得分
     */
    public static final Integer MAX_SCORE = 7;

    /**
     * map的hash初始值
     */
    public static final int HASH_MAP_INIT_VALUE = 32;

    /**
     * 全园平均分
     */
    public final static String WHOLE_AVERAGE = "全园平均分";
    /**
     * 查询失败
     */
    public final static String QUERY_FAIRURE = "查询失败";
    /**
     * 操作成功
     */
    public final static String SUCCESS_MSG = "操作成功！";
    /**
     * 操作失败
     */
    public final static String FARIURE_MSG = "操作失败！";
    /**
     * 导出失败
     */
    public final static String FARIURE_EXPORT = "导出失败！";

    /**
     * 请求头 - 文档
     */
    public final static String CONTENT_TYPE_WORD = "application/msword";
    /**
     * 请求头 - 下载
     */
    public final static String CONTENT_TYPE_DOWNLOAD = "application/x-download";
    /**
     * 请求头 - 二进制文件
     */
    public final static String CONTENT_TYPE_STEAM = "application/octet-stream;charset=UTF-8";
    /**
     * 请求头
     */
    public final static String USER_AGENT = "User-Agent";
    /**
     * 请求头
     */
    public final static String CONTENT_TYPE = "Content-Type";
    /**
     * 连接
     */
    public final static String CONNECTION = "Connection";
    /**
     * 关闭连接
     */
    public final static String CLOSE = "close";
    /**
     * 连接
     */
    public final static String ACCESS_CONTROL_ALLOW_ORIGIN = "Access-Control-Allow-Origin";
    /**
     * 连接
     */
    public final static String CONTENT_DISPOSITION = "Content-Disposition";

    /**
     * 浏览器 - ie
     */
    public final static String MSIE = "MSIE";
    /**
     * 浏览器 - Firefox
     */
    public final static String FIREFOX = "Firefox";

    /**
     * 填写报告的step
     */
    public final static String MODULE_STEP3_REPORT = "qualityreport";

    /**
     * 督导下园核实的材料
     */
    public final static String MODULE_STEP1_MATERIAL = "qualitymetrail";

    /**
     * 数字3
     */
    public final static int NUM_3 = 3;
    /**
     * 数字4
     */
    public final static int NUM_4 = 4;

    /**
     * 数字5
     */
    public final static int NUM_5 = 5;
    /**
     * 数字6
     */
    public final static int NUM_6 = 6;
    /**
     * 数字7
     */
    public final static int NUM_7 = 7;
    /**
     * 数字8
     */
    public final static int NUM_8 = 8;
    /**
     * 数字9
     */
    public final static int NUM_9 = 9;
    /**
     * 数字10
     */
    public final static int NUM_10 = 10;
    /**
     * 数字11
     */
    public final static int NUM_11 = 11;

    /**
     * 数字12
     */
    public final static int NUM_12 = 12;
    /**
     * 数字13
     */
    public final static int NUM_13 = 13;
    /**
     * 数字14
     */
    public final static int NUM_14 = 14;
    /**
     * 数字15
     */
    public final static int NUM_15 = 15;
    /**
     * 数字16
     */
    public final static int NUM_16 = 16;
    /**
     * 数字17
     */
    public final static int NUM_17 = 17;
    /**
     * 数字18
     */
    public final static int NUM_18 = 18;
    /**
     * 数字19
     */
    public final static int NUM_19 = 19;
    /**
     * 数字20
     */
    public final static int NUM_20 = 20;

    /**
     * 格式化数字
     */
    public final static String DECIMAL_Format = "######.00";

}

```



### 页面层处理：

前端增加一个form提交，使用form提交表单数据，实现word导出功能:

(注意使用的模板引擎是thymeleaf)

#### html代码:

```html
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
<!-- 最新版本的 Bootstrap 核心 CSS 文件 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<div class="container">
    <div id="vue1" class="row">
<!--        <form @submit.prevent="submit">-->
<!--            <div>-->
<!--                导出word名称：<input class="form-control" type="text" v-model="student.wordName">-->
<!--            </div>-->

<!--            <div>-->
<!--                test0：<input type="text" class="form-control" v-model="student.test0">-->
<!--                test1：<input type="text" class="form-control" v-model="student.test1">-->
<!--                test2：<input type="text" class="form-control" v-model="student.test2">-->
<!--                test4：<input type="text" class="form-control" v-model="student.test4">-->
<!--                test5：<input type="text" class="form-control" v-model="student.test5">-->
<!--                test6：<input type="text" class="form-control" v-model="student.test6">-->
<!--            </div>-->

<!--            <input type="submit" class="btn btn-danger" value="提交">-->
<!--        </form>-->


        <form method="post" action="/quality/exportword">
            <div>
                导出word名称：<input class="form-control" type="text" name="wordName">
            </div>

            <div>
                test0：<input type="text" class="form-control" name="test0">
                test1：<input type="text" class="form-control" name="test1">
                test2：<input type="text" class="form-control" name="test2">
                test4：<input type="text" class="form-control" name="test4">
                test5：<input type="text" class="form-control" name="test5">
                test6：<input type="text" class="form-control" name="test6">
            </div>

            <input type="submit" class="btn btn-danger" value="提交">
        </form>

    </div>
</div>

</body>
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="https://cdn.staticfile.org/vue-resource/1.5.1/vue-resource.min.js"></script>
<script th:src="@{/js/exportword.js}"></script>
</html>
```

#### js代码

使用js代码处理form表单提交，使用了jquery进行导出，其实一直不太懂前端怎么导出后台产生的二进制流，做法挺多，下次写一篇文章好好汇总一下几种用法。

```javascript
var v1 = new Vue({
    el: '#vue1',
    data: {
        counter: 0,
        student: {
            test0:'',
            test1:'',
            test2:'',
            test3:'',
            test4:'',
            test5:'',
            test6:'',
            wordName: '',

        }
    },
    methods: {
        test: function () {
            console.log(this.counter);
        },
        submit() {
            console.log(this.student);
            var url = '/quality/exportword';
            var formData = JSON.stringify(this.student); // this指向这个VUE实例 data默认绑定在实例下的。所以直接this.student就是要提交的数据
            this.$http.post(url, formData).then(function (data) {
                console.log(data);
                let blob = new Blob([data.data],{ type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document;charset=utf-8'});
                let objectUrl = URL.createObjectURL(blob);
                window.location.href = objectUrl;
            }).catch(function () {
                console.log('test');
            });
        }
    }
})
```





# 结尾：

个人水平一般，希望通过这篇文章可以帮到读者，有错误的地方欢迎指点，看到会及时改成，谢谢！

前段时间忙于面试找到新的地方工作了，等工作安定之后，会继续深耕博客和技术栈。

