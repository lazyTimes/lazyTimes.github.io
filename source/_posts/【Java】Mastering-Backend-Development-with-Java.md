---
title: 【Java】Mastering Backend Development with Java
subtitle: SpringBoot 开发的一些建议
author: lazytime
url_suffix: masteringjava
tags:
  - Java
categories:
  - Java
keywords: 请输入关键字（英文逗号分隔多个关键字）
description: 请输入描述信息
copyright: true
date: 2023-09-07 14:03:14
---
#springboot
# Source

[Mastering Backend Development with Java Spring Boot: Best Practices and Pro Tips | by Nihal Parmar | Medium](https://medium.com/@itznihal/mastering-backend-development-with-java-spring-boot-best-practices-and-pro-tips-3fc0f501418e)

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131707.png)

Spring Boot is a widely used and very popular enterprise-level high-performance framework. 

Spring Boot 是一种广泛使用且非常流行的企业级高性能框架。

Here are some best practices and a few tips you can use to improve your Spring Boot application and make it more efficient. 

以下是一些最佳实践和技巧，您可以用来改进 Spring Boot 应用程序，使其更加高效。

This article will be a little longer, and it will take some time to completely read the article.

本文篇幅稍长，完整阅读需要耗费不少时间。

<!-- more -->

# Proper packaging style  适当的包装风格

- Proper packaging will help to understand the code and the flow of the application easily.

- 适当的包装有助于轻松理解代码和应用程序的流程。

- You can structure your application with meaningful packaging.

- 您可以通过有意义的打包来构建应用程序。

- You can include all your controllers into a separate package, services in a separate package, util classes into a separate package…etc. This style is very convenient in small-size microservices.

- 您可以将所有`controller`打包成一个单独的包，将服务打包成一个单独的包，将 `util` 类打包成一个单独的包......等等。这种风格在小型微服务中非常方便。

- If you are working on a huge code base, a feature-based approach can be used. You can decide on your requirement.

- 如果您正在处理庞大的代码库，则可以使用基于功能的方法。您可以根据自己的需求来决定。

**Based on type**

**基于类型**

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131717.png)


**Based on feature**

**基于功能**

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131741.png)

# Use design patterns 使用设计模式

- No complaints. Design patterns are already best practices.
- But you must identify the place where you can use them.
- Please check this article to understand “[**_how to use the Builder design pattern_**](https://www.javastackflow.com/2022/09/how-to-use-builder-design-pattern-with.html)” in our Spring Boot applications.

- 不要埋怨。设计模式已经是最佳实践。
- 但您必须确定可以使用它们的地方。
- 请查看本文以了解："[**_如何在 Spring Boot 应用程序中使用生成器设计模式_**](https://www.javastackflow.com/2022/09/how-to-use-builder-design-pattern-with.html)"。

# Use Spring Boot starters 使用 Spring Boot 启动器

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131756.png)


Image: [https://www.merriam-webster.com/words-at-play/names-of-appetizers](https://www.merriam-webster.com/words-at-play/names-of-appetizers)

- This is a cool feature of Spring Boot.

- 这是 Spring Boot 的一个很酷的功能。

- We can very easily use starter dependencies without adding single dependencies one by one. These starter dependencies are already bundled with the required dependencies.

- 我们可以非常轻松地使用启动依赖，而无需逐个添加单个依赖。这些启动依赖已经与所需的依赖捆绑在一起。

- For example, if we add **spring-boot-starter-web** dependency，by default,  it is bundled with **jackson, spring-core, spring-mvc, and spring-boot-starter-tomcat dependencies**.

- 例如，如果我们添加了**spring-boot-starter-web**依赖，默认情况下，它与 **jackson、spring-core、spring-mvc 和 spring-boot-starter-tomcat 依赖**捆绑在一起。

- So we don’t need to care about adding dependencies separately.

- 因此，我们无需单独添加依赖。

- And also it helps us to avoid version mismatches.

- 此外，它还能帮助我们避免版本不匹配。
# Use proper versions of the dependencies 使用正确的依赖版本

- It is always recommended to use the latest stable **GA** versions.

- 建议始终使用最新的稳定**GA**版本。

- Sometimes it may vary with the Java version, server versions, the type of the application…etc.

- 有时可能会因 Java 版本、服务器版本、应用程序类型......等而有所不同。

- Do not use different versions of the same package and always use < properties> to specify the version if there are multiple dependencies.

- 不要使用同一软件包的不同版本，如果存在多个依赖关系，应始终使用 < properties> 来指定版本。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131843.png)


# Use Lombok 使用 Lombok

- As a Java developer, you have probably heard of the **Lombok project**.
- Lombok is a Java library that can be used to reduce your codes and allow you to write clean code using its annotations.
- For example, you may use plenty of lines for getters and setters in some classes like entities, request/response objects, dtos…etc.
- But if you use Lombok, it is just one line, you can use **@Data, @Getter** or **@Setter** as per your requirement.
- You can use **_Lombok_** logger annotations as well. **_@Slf4j_** is recommended.
- Check this [**_file_**](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/entity/InputFile.java) for your reference.

- 作为一名 Java 开发人员，您可能听说过**Lombok 项目**。
- `Lombok` 是一个 Java 库，可用于减少代码，并允许您使用其注解编写简洁的代码。
- 例如，在一些类（如实体、请求/响应对象、dtos......等）中，您可能会使用大量的行来表示 `getters` 和 `setters`。
- 但如果使用 **Lombok**，则只需一行，您可以根据需要使用 **@Data**、**@Getter** 或 **@Setter**。
- 您还可以使用 **_Lombok_** 日志注释。 建议使用 **_@Slf4j_**。
- 请查看此 [**_file_**](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/entity/InputFile.java)，以供参考。
# Use constructor injection with Lombok 使用 Lombok 注入构造函数

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131901.png)


- When we talk about dependency injection, there are two types.

- 当我们谈论依赖注入时，有两种类型。

- One is “**constructor injection**” and the other is “**setter injection**”. Apart from that, you can also use “**field injection**” using the very popular **@Autowired** annotation.

- 一种是 "**构造器注入**"，另一种是 "**设置器注入**"。除此之外，你还可以使用非常流行的**@Autowired**注解来进行 " **字段注入** "。

- But we highly recommend using **Constructor injection** over other types. Because it allows the application to initialize all required dependencies at the initialization time.

- 但我们强烈建议使用**构造器注入**，而不是其他类型。因为**它允许应用程序在初始化时初始化所有需要的依赖关系**。

- This is very useful for unit testing.

- 这对**单元测试**非常有用。

- The important thing is, that we can use the **@RequiredArgsConstructor** annotation by Lombok to use constructor injection.

- 重要的是，我们可以使用 Lombok 的 **@RequiredArgsConstructor** 注解来使用构造器注入。

- Check this [_sample controller_](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/controller/FileController.java) for your reference.

- 请查看 [_sample controller_](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/controller/FileController.java) 以供参考。

```java
package com.app.controller;

import com.app.entity.InputFile;
import com.app.service.FileService;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@RestController
@RequestMapping("/api/v1/files")
@RequiredArgsConstructor
public class FileController {

    private static final Logger LOGGER = LoggerFactory.getLogger(FileController.class);

    private final FileService fileService;

    @PostMapping(consumes = {MediaType.MULTIPART_FORM_DATA_VALUE})
    public List<InputFile> addFile(@RequestParam("files")MultipartFile[] files){
        LOGGER.debug("Call addFile API");
        return fileService.uploadFiles(files);
    }
}
```

# Use slf4j logging 使用 slf4j 日志

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907131917.png)

- Logging is very important.
- If a problem occurs while your application is in production, logging is the only way to find out the root cause.
- Therefore, you should think carefully before adding loggers, log message types, logger levels, and logger messages.
- Do not use System.out.print()
- Slf4j is recommended to use along with logback which is the default logging framework in Spring Boot.
- Always use slf4j { } and avoid using String interpolation in logger messages. Because string interpolation consumes more memory.
- Please check [_this file_](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/service/FileServiceImpl.java) for your reference to get an idea about, implementing a logger.
- ==You can use Lombok== ==**_@Slf4j_**== ==_annotation to create a logger very easily._==
- If you are in a micro-services environment, you can use the ELK stack.

- 日志记录非常重要。  
- 如果应用程序在生产过程中出现问题，日志记录是找出根本原因的唯一途径。  
- 因此，在添加日志记录器、日志消息类型、日志记录器级别和日志记录器消息之前，应仔细考虑。  
- **不要使用 System.out.print()**
- 建议使用 `Slf4j` 和 `logback`，后者是 `Spring Boot` 的默认日志框架。  
- 始终使用 slf4j { }，避免在日志记录器消息中使用字符串插值。因为字符串插值会消耗更多内存。  
- 请查看 [_此文件_](https://github.com/raviyasas/springboot-gcs-demo/blob/master/src/main/java/com/app/service/FileServiceImpl.java) 以了解如何实现日志记录器，供您参考。  
- ===你可以使用 Lombok== ==**_@Slf4j_**== ==_annotation 来轻松创建日志记录器。  
- 如果你身处微服务环境，可以使用 ELK 栈。  

# Use Controllers only for routing 仅在路由时使用控制器

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907133704.png)

- Controllers are dedicated to routing.
- It is **stateless** and **singleton**.
- The DispatcherServlet will check the _@RequestMapping_ on _Controllers_
- Controllers are the ultimate target of requests, then requests will be handed over to the service layer and processed by the service layer.
- The business logic **should not be** in the controllers.

- 控制器专用于路由。
- 它是**无状态**和**单例**。
- DispatcherServlet 会检查控制器上的 _@RequestMapping_ 。
- 控制器是请求的最终目标，然后请求将被移交给服务层并由服务层处理。
- 业务逻辑**不应该**在控制器中。
# Use Services for business logic 业务逻辑使用服务

- The **complete business logic goes here** with validations, caching…etc.
- Services communicate with the persistence layer and receive the results.
- Services are also singleton.

- 这里***完整的业务逻辑，包括验证、缓存......等。
- 服务与持久层通信并接收结果。
- 服务也是单例的。

> _Bonus article:_ [_Manage stress as a Software Engineer_](https://medium.com/@raviyasas/why-stress-and-how-to-manage-it-as-a-software-engineer-204b4e21ed9)
> 
> _Bonus article:_ [_Manage stress as a Software Engineer_](https://medium.com/@raviyasas/why-stress-and-how-to-manage-it-as-a-software-engineer-204b4e21ed9)

# Avoid NullPointerException 避免 NullPointerException

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907134658.png)

- To avoid **NullPointerException** you can use **Optional** from java.util package.
- You can also use null-safe libraries. Ex: **Apache Commons StringUtils**
- Call **equals()** and **equalsIgnoreCase()** methods on known objects.
- Use **valueOf()** over **toString()**
- Use IDE-based **@NotNull** and **@Nullable** annotations.

- 避免**NullPointerException**，您可以使用 java.util 包中的**Optional**。
- 您还可以使用空安全库。例如：**Apache Commons StringUtils**
- 在已知对象上调用 **equals()** 和 **equalsIgnoreCase()** 方法。
- 使用 **valueOf()** 而非 **toString()**
- 使用基于 IDE 的 **@NotNull** 和 **@Nullable** 注解。
# Use best practices for the Collection framework 使用Collection框架的最佳实践

- Use appropriate collection for your data set.
- Use **forEach** with Java 8 features and avoid using legacy for loops.
- Use **interface type** instead of the implementation.
- Use **isEmpty()** over **size()** for better readability.
- Do not return null values, you can return an empty collection.
- If you are using objects as data to be stored in a hash-based collection, you should override equals() and hashCode() methods. Please check this article “[_How does a HashMap internally work_](https://medium.com/@raviyasas/how-a-hashmap-internally-works-93e3887978f3)”.

- 针对数据集使用适当的集合。
- 使用 Java 8 Lambda 的 **forEach**，避免使用传统的 for 循环。
- 使用 **接口类型** 而不是具体实现。
- 使用 **isEmpty()**，而不是 **size()**，以获得更好的可读性。
- 不要返回空值，可以返回空集合。
- 如果使用对象作为数据存储在基于Hash的集合中，则应覆盖 equals() 和 hashCode() 方法。请查看本文"[_HashMap 内部如何工作_](https://medium.com/@raviyasas/how-a-hashmap-internally-works-93e3887978f3)"。

# Use pagination 使用分页

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230907134647.png)


- This will improve the performance of the application.
- If you’re using **Spring Data JPA**, the **_PagingAndSortingRepository_** makes using pagination very easy and with little effort.

- 这将提高应用程序的性能。
- 如果你使用的是**Spring Data JPA**，那么 **_PagingAndSortingRepository_** 会让分页功能的使用变得非常简单，而且几乎不费吹灰之力。
# Use caching 使用缓存

- Caching is another important factor when talking about application performance.
- By default Spring Boot provides caching with **ConcurrentHashMap** and you can achieve this by **@EnableCaching** annotation.
- If you are not satisfied with default caching, you can use **Redis**, **Hazelcast,** or any other distributed caching implementations.
- Redis and Hazelcast are **in-memory** caching methods. You also can use database cache implementations as well.

- 缓存是影响应用程序性能的另一个重要因素。
- 默认情况下，Spring Boot 通过 **ConcurrentHashMap** 提供缓存，您可以通过 **@EnableCaching** 注解实现缓存。
- 如果对默认缓存不满意，可以使用 **Redis**、**Hazelcast** 或其他分布式缓存实现。
- **Redis** 和 **Hazelcast** 是**内存**缓存方法。您也可以使用数据库缓存实现。
# Use custom exception handler with global exception handling 使用自定义异常处理程序和全局异常处理

![](https://miro.medium.com/v2/resize:fit:875/0*ErfHekfADhtYb8e9.png)

- This is very important when working with large enterprise-level applications.
- Apart from the general exceptions, we may have some scenarios to identify some specific error cases.
- Exception adviser can be created with **@ControllerAdvice** and we can create separate exceptions with meaningful details.
- It will make it much easier to identify and debug errors in the future.
- Please check [_this_](https://github.com/raviyasas/springboot-exceptions-demo/tree/master/src/main/java/com/app/exception) and [_this_](https://github.com/raviyasas/springboot-gcs-demo/tree/master/src/main/java/com/app/exception) for your reference.

- 在处理大型企业级应用程序时，这一点非常重要。
- 除了一般的异常情况外，我们可能还需要在某些情况下识别一些特定的错误案例。
- 可以使用 **@ControllerAdvice** 创建异常切面，我们可以创建具有有意义细节的单独异常。
- 这将大大方便今后识别和调试错误。
- 请查看 [_this_](https://github.com/raviyasas/springboot-exceptions-demo/tree/master/src/main/java/com/app/exception) 和 [_this_](https://github.com/raviyasas/springboot-gcs-demo/tree/master/src/main/java/com/app/exception)，以供参考。


> _Bonus article:_ [_What is serverless Architecture?_](https://www.javastackflow.com/2022/09/what-is-serverless-architecture.html)
> 
> _Bonus article:_ [_什么是无服务架构？_](https://www.javastackflow.com/2022/09/what-is-serverless-architecture.html)

# Use custom response object 使用自定义响应对象

![](https://miro.medium.com/v2/resize:fit:875/0*HsR7UP64pwDENvq8.png)

- A custom response object can be used to return an object with some specific data with the requirements like HTTP status code, API code, message…etc.
- We can use the **builder design pattern** to create a custom response object with custom attributes.
- Please check [**_this article_**](https://www.javastackflow.com/2022/09/how-to-use-builder-design-pattern-with.html) for your reference.

- 自定义响应对象可用于返回带有某些特定数据的对象，这些数据应符合 HTTP 状态代码、API 代码、消息等要求。
- 我们可以使用 ** 生成器设计模式** 来创建带有自定义属性的自定义响应对象。
- 请查阅[**_本文_**](https://www.javastackflow.com/2022/09/how-to-use-builder-design-pattern-with.html)以供参考。
# Remove unnecessary codes, variables, methods, and classes.  删除不必要的代码、变量、方法和类


![](https://miro.medium.com/v2/resize:fit:875/0*I1A3DYUGTcDRdIQ2.png)

commented code in the production

生产中的注释代码

- **Unused variable** declarations will acquire some memory.
- Remove unused methods, classes…etc because it will impact the performance of the application.
- Try to avoid nested loops. You can use maps instead.

- 未使用的变量，**声明会占用一些内存。
- 删除未使用的方法、类......等，因为这将影响应用程序的性能。
- 尽量避免嵌套循环。可以使用映射来代替。
# Using comments 使用注释

- Commenting is a good practice unless you abuse it.
- DO NOT comment on everything. Instead, you can write descriptive code using meaningful words for classes, functions, methods, variables…etc.
- Remove commented codes, misleading comments, and story-type comments.
- You can use comments for warnings and explain something difficult to understand at first sight.

- 注释是一种很好的做法，除非你滥用它。
- **切勿对所有内容进行注释**。相反，您可以使用有意义的词来描述类、函数、方法、变量......等，编写描述性代码。
- 删除**注释代码**、**误导性注释**和**故事型注释**。
- 您可以使用注释来警告和解释一些**乍一看难以理解**的内容。
# Use meaningful words for classes, methods, functions, variables, and other attributes. 对类、方法、函数、变量和其他属性使用有意义的词语。

![](https://miro.medium.com/v2/resize:fit:875/0*uJmiodAstz6j0-1j.png)

- This looks very simple, but the impact is huge.
- Always use proper **meaningful and searchable** naming conventions with proper case.
- Usually, we use **nouns or short phrases** when declaring **classes**, **variables**, and **constants**. Ex: String firstName, const isValid
- You can use **verbs and short phrases with adjectives** for **functions** and **methods**. Ex: readFile(), sendData()
- Avoid using **abbreviating variable** names and **intention revealing** names. Ex: int i; String getExUsr;
- If you use this meaningfully, declaration comment lines can be reduced. Since it has meaningful names, a fresh developer can easily understand by reading the code.

- 这看起来很简单，但影响却很大。
- 始终使用**有意义且可搜索的**命名规则，并使用正确的大小写。
- 通常，我们在声明**类**、**变量**和**常量**时使用**名词或短语**。例如：字符串 firstName、常量 isValid
- 在声明**函数**和**方法**时，可以使用**谚语和带有形容词的短语**。例如：readFile()、sendData()
- 避免使用**缩略变量**名和**暴露想法**名。例如：int i; String getExUsr；
- 如果使用这些有意义的名称，就可以减少声明注释行。由于使用了有意义的名称，新开发人员在阅读代码时很容易理解。
# Use proper case for declarations 在声明中使用合适的大小写

![](https://miro.medium.com/v2/resize:fit:875/0*LWJF3cRbzU4wU09A.png)

- There are many different cases like **UPPERCASE, lowercase, camelCase, PascalCase, snake_case, SCREAMING_SNAKE_CASE, kebab-case**…etc.
- But we need to identify which case is dedicated to which variable.
- Usually, I follow,

- 有许多不同的大小写，如 **大写、小写、骆驼大写、帕斯卡大写、蛇大写、SCREAMING_SNAKE_CASE、kebab-case**......等等。
- 但我们需要确定哪个大小写专用于哪个变量。
- 通常，我是这样做的。

classes — **PascalCase**

methods & variables — **camelCase**

方法和变量--**camelCase**

constants — **SCREAMING_SNAKE_CASE**

常量 - **SCREAMING_SNAKE_CASE**

DB-related fields — **snake_case**

与 DB 相关的字段 - **snake_case**

- This is just an example. It can be different from the standard you follow in the company.

- 这只是一个示例。它可能与您在公司中遵循的标准不同。
# Be simple 保持简单

![](https://miro.medium.com/v2/resize:fit:875/0*am8Xo9e4ngoABycE.png)

- Always try to write simple, readable codes.
- The same simple logic can be implemented using different ways, but it is difficult to understand if it is not readable or understandable.
- Sometimes complex logic consumes more memory.
- Try to use **KISS**, **DRY**, and **SOLID** principles when writing codes. I will explain this in a future article.

- 始终尽量编写简单、可读的代码。
- 同样简单的逻辑可以用不同的方法实现，但如果不可读性或不可懂性，就很难理解。
- 有时复杂的逻辑会消耗更多内存。
- 在编写代码时，尽量使用**KISS**、**DRY**和**SOLID**原则。我将在今后的文章中对此进行解释。

> DRY stands for “Don’s Repeat Yourself

如果代码可以公用就尽量公用，不要复制粘贴代码。

> kiss：KISS stands for “Keep It Simple, Stupid”.

KISS就是保持简单，愚蠢。这个愚蠢指的是方法最好只知道干一件事情。这两点和Unix最初的设计的理念是一样的，简单好用即是美。

**SOLID**原则：

**Single Responsibility Principle** 单一职责：每个类和接口有明确目标。

**Open-Closed Principle** 开闭原则：开放扩展，封闭修改。

**Liskov Substitution Principle** 里式替代：尽可能让代码多态。

**Interface Segregation Principle** 接口隔离：实现接口替代类继承。

**Dependency Inversion Principle** 依赖倒转原则：依赖抽象而不是依赖实现。
# Use a common code formatting style 使用通用的代码格式样式

![](https://miro.medium.com/v2/resize:fit:875/0*IW4rJbi2sMKKak9P.png)

- Formatting styles vary from developer to developer. Coding style changes are also considered a change and can make code merging very difficult.
- To avoid this, the team can have a common coding format.

- 格式风格因开发人员而异。编码风格的改变也被认为是一种改变，会使代码合并变得非常困难。
- 为了避免这种情况，团队可以采用统一的编码格式。

# Use SonarLint 使用 SonarLint

![](https://miro.medium.com/v2/resize:fit:240/0*ChADmyd-l_RX102d.png)

- This is very useful for identifying small bugs and best practices to avoid unnecessary bugs and code quality issues.

- 这对识别小错误和最佳实践非常有用，可避免不必要的错误和代码质量问题。

- You can install the plugin into your favorite IDE.

- 您可以将该插件安装到自己喜欢的集成开发环境中。

Follow me for _new tidbits_ on the **domain of tech.**