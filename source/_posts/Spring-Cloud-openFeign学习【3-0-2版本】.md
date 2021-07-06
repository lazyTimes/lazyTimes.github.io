---
title: Spring Cloud openFeign学习【3.0.2版本】
subtitle: 内容分为openFeign大致的使用和源码的个人解读
url_suffix: openfeigncloud
author: lazytime
tags:
  - springcloud
categories:
  - springcloud
keywords: openfeign
description: 源码阅读
copyright: true
date: 2021-07-06 23:11:40
---

# Spring Cloud openFeign学习【3.0.2版本】

# 前言

​	内容分为openFeign大致的使用和源码的个人解读，里面参考了不少其他优秀博客作者的内容，很多地方基本算是拾人牙慧了，不过还是顺着源码读了一遍加深理解。

<!-- more -->

# openFeign 是什么？

​	Feign是一个**声明性web服务客户端**。它使编写web服务客户机更加容易，要使用Feign，需要创建一个接口并对其进行注释。它具有可插入注释支持，包括Feign注释和JAX-RS注释。

​	Feign还支持**可插拔编码器和解码器**。Spring Cloud增加了对Spring MVC注解的支持，并支持使用Spring Web中默认使用的相同HttpMessageConverters。

​	Spring Cloud集成了Eureka、Spring Cloud CircuitBreaker和Spring Cloud LoadBalancer，在使用Feign时提供一个**负载均衡的http客户端**。

# 如何学习？

​	框架最大的意义在于使用，其实最好的教程就是边做边参考官方的文档学习。

[官方文档目录地址](https://spring.io/projects/spring-cloud-openfeign)

[官方openFeign的文档](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)

# 应用场景？

​	可以看到openFeign作为服务的调用中转，负责服务之间的连接和请求转发的操作。OpenFeign作为编写服务调用支持组件在spring cloud中占有极为重要的位置。

​	和RPC的通信框架不同，openFeign使用了传统的http作为传输结构。

​	在以往使用Ribbon的时候，服务调用通常使用的是手动调用，这需要花费大量的人工协调时间。现在通过openFeign把服务调用“本地化”。调用其他的服务的接口API像调用本地方法一样。这样既不需要频繁的改动接口，又可以控制服务的调用，而不会导致服务提供方的变动而“失效”。

# Ribbon、Feign和OpenFeign的区别

[Ribbon、Feign和OpenFeign的区别](https://blog.csdn.net/zimou5581/article/details/89949852)

## Ribbon

​	Ribbon 是 Netflix开源的**基于HTTP和TCP**等协议负载均衡组件

​	Ribbon 可以用来做**客户端负载均衡**，调用注册中心的服务

​	Ribbon的使用需要代码里**手动调用目标服务**，请参考官方示例：[官方示例](https://github.com/Netflix/ribbon)

## Feign

​	Feign是Spring Cloud组件中的一个**轻量级RESTful的HTTP服务**客户端。

​	Feign内置了Ribbon，用来做**客户端负载均衡**，去调用**服务注册中心的服务**。

​	Feign的使用方式是：**使用Feign的注解**定义接口，调用这个接口，就可以调用服务注册中心的服务。

​	Feign支持的注解和用法请参考官方文档：[官方文档](https://github.com/OpenFeign/feign)。

​	**Feign本身不支持Spring MVC的注解，它有一套自己的注解**。



## OpenFeign

​	OpenFeign是Spring Cloud 在Feign的基础上支持了Spring MVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的**@RequestMapping**注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

​	根据上面的描述，绘制如下的表格内容：

| -        | Ribbon                                 | Feign                                                        | OpenFeign                                                  |
| -------- | -------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| 使用方式 | 手动调用目标服务                       | Feign的注解定义接口，调用接口就可以调用注册中心服务          | 可以直接使用服务调用的方式调用对应的服务                   |
| 作用     | 客户端负载均衡，服务注册中心的服务调用 | 客户端负载均衡，服务注册中心的服务调用                       | 动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务 |
| 开发商   | Netfix                                 | Spring Cloud                                                 | Spring Cloud                                               |
| 特点     | **基于HTTP和TCP**等协议负载均衡组件    | **轻量级RESTful的HTTP服务**客户端。依靠自我实现的注解进行请求处理 | 支持了Spring MVC的注解的**轻量级RESTful的HTTP服务**客户端  |
| 目前情况 | 维护中                                 | 停止维护                                                     | 维护中                                                     |

# openFeign增加了那些功能：

1. 可插拔的注解支持，包括Feign注解和JSX-RS注解。
2. 支持可插拔的HTTP编码器和解码器。
3. 支持Hystrix和它的Fallback。
4. 支持Ribbon的负载均衡。
5. 支持HTTP请求和响应的压缩。

# openFeign的client实现方替换：

1. 可以使用http client 替换，并且openFeign 提供了良好的配置，可以支持httpclient的细节化配置。
2. 使用okHttpClient, 可以实现 okhttpClient 实现自定义的httpclient注入模式，但是会出现一定的问题。

# 使用方式：

## 1. 添加依赖

​	按照maven的依赖管理，我们需要使用此方式进行处理

```
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
     <version>${feign.version}</version>
     <scope>compile</scope>
     <optional>true</optional>
</dependency>
```

## 2. 开启注解@EnableFeignClients

​	`application`启动类 需要添加对应的配置：`@EnableFeignClients`用于允许访问。

> spring cloud feign的默认配置：
>
> Spring Cloud OpenFeign默认为伪装提供以下bean（`BeanType`beanName ：）`ClassName`：
>
> - `Decoder`feignDecoder ：（`ResponseEntityDecoder`包含`SpringDecoder`）
> - `Encoder` feignEncoder： `SpringEncoder`
> - `Logger` feignLogger： `Slf4jLogger`
> - `MicrometerCapability`micrometerCapability：如果`feign-micrometer`在类路径上并且`MeterRegistry`可用
> - `Contract` feignContract： `SpringMvcContract`
> - `Feign.Builder` feignBuilder： `FeignCircuitBreaker.Builder`
> - `Client`feignClient：如果在类路径`FeignBlockingLoadBalancerClient`上使用Spring Cloud **LoadBalancer**，则使用。如果它们都不在类路径上，则使用默认的伪装客户端。

## 3. yml增加配置：

​	yml文件内部的文件内容如下：

```
feign:
    client:
        config:
            feignName:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: full
                errorDecoder: com.example.SimpleErrorDecoder
                retryer: com.example.SimpleRetryer
                defaultQueryParameters:
                    query: queryValue
                defaultRequestHeaders:
                    header: headerValue
                requestInterceptors:
                    - com.example.FooRequestInterceptor
                    - com.example.BarRequestInterceptor
                decode404: false
                encoder: com.example.SimpleEncoder
                decoder: com.example.SimpleDecoder
                contract: com.example.SimpleContract
                capabilities:
                    - com.example.FooCapability
                    - com.example.BarCapability
                metrics.enabled: false
```

## 4. 具体使用:

​	更多的用法请根据网上资料或者官方文档，下面列举一些具体的配置或者使用方法:

> 如果openFeign的名称发生冲突，需要使用`contextId`对于防止bean的名称冲突
>
> ```
> @FeignClient(contextId = "fooClient", name = "stores", configuration = FooConfiguration.class)
> ```

### 上下文继承

​	如果将FeignClient配置为不从父上下文继承bean，可以使用下面的写法：

```
@Configuration
public class CustomConfiguration{

    @Bean
    public FeignClientConfigurer feignClientConfigurer() {
        return new FeignClientConfigurer() {
            @Override
            public boolean inheritParentConfiguration() {
                return false;
            }
        };
    }
}
```

> 注意：默认情况下feign不会对与斜杠进行编码，如果要对斜杠编码，需要使用如下方式：
>
> ```
> feign.client.decodeSlash：false
> ```

### 日志输出

​	feign的默认日志输出等级如下：

```
logging.level.project.user.UserClient: DEBUG
```

​	下面是日志打印的内容：

- `NONE`：默认不记录任何日志（默认设置）
- `BASIC`：只记录和请求以及响应时间相关的日志信息
- `HEADERS`：记录基本信息以及请求和响应**头**
- `FULL`:记录请求和响应的头、主体和元数据。(所有信息记录)

### 开启压缩

​	可以通过如下配置，开始http压缩：

```
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

​	如果需要更进一步的配置，可以使用如下的形式进行配置：

```
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

​	注意**2048**值为压缩请求的最小阈值，因为如果对于所有请求进行gzip压缩，对于小文件的性能开销要反而要更大

​	通过下面的配置来开启gzip压缩（压缩编码为UTF-8，默认）：

```
feign.compression.response.enabled=true
feign.compression.response.useGzipDecoder=true
```

## 5. 附录：

### yml相关配置表：

​	这部分配置可以直接参考官网的处理：[yml相关配置表](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/appendix.html)

# openFeign的源码解读(重点)

​	下面为借助文章理解和自己看源码的总结。整个调用过程还是比较好理解的。因为说白了本身就是对于一次http请求的抽象和封装而已。不过这部分用到了很多的设计模式，比如随处可见的建造者模式和策略模式。同时这一块的设计使用大量的包访问结构闭包，所以要对其进行二次开发会稍微麻烦一些，但是使用反射这些屏障基本算是形同虚设了。

​	参考资料：**掘金【【图文】Spring Cloud OpenFeign 源码解析】：https://juejin.cn/post/6844904066229927950#heading-24**

## feign工作流程图

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/B837F406-A869-4803-AC74-1B592F4BAF28.png?ynotemdtimestamp=1625580577970)

## 工作流程概览

这里主要介绍一次**openFeign**请求调用的流程，对于注解处理以及组件注册的部分放到了文章的结尾部分。

- **Feign**实例化**newInstance()**
  - 实例化**SyncronizedMethodHandler**以及**ParseHandlersByName**，注入到**ReflectFeign**对象。
- 构建**ParseHandlersByName**对象，对于参数进行转化
- 构建**Contract**对象，对于请求参数进行校验和解析
  - 实例化**SpringMvcContract**对象（继承自Contract对象）
  - 调用**parseAndValidateMetadata()** 处理和校验数据类型
- 通过**jdk动态代理**Proxy创建动态代理对象**MethodInvocationHandler**，调用动态代理对象的**invoke()**方法
- 代理类**SyncronizedInvocationHandler**构建 **requestTeamplate**对象，并发送请求
  - 调用**create()**构建请求实体对象
  - 对于请求参数进行**encode()**操作
  - 构建**client**对象，执行请求
  - 返回请求结果
- 获取请求结果，请求完成

## 详解openFeign工作流程（重点）

### 1. Feign 实例化 - newInstance()

​	当服务通过feign调用另一个服务的时候，在**Fegin.builder**对象中，会调用构造器构造一个**Fegin**实例，下面是**feign.Feign.Builder#build**的代码内容：

```
public Feign build() {
      // 构建核心组件和相关内容
      Client client = Capability.enrich(this.client, capabilities);
      Retryer retryer = Capability.enrich(this.retryer, capabilities);
      List<RequestInterceptor> requestInterceptors = this.requestInterceptors.stream()
          .map(ri -> Capability.enrich(ri, capabilities))
          .collect(Collectors.toList());
      Logger logger = Capability.enrich(this.logger, capabilities);
      Contract contract = Capability.enrich(this.contract, capabilities);
      Options options = Capability.enrich(this.options, capabilities);
      Encoder encoder = Capability.enrich(this.encoder, capabilities);
      Decoder decoder = Capability.enrich(this.decoder, capabilities);
      InvocationHandlerFactory invocationHandlerFactory =
          Capability.enrich(this.invocationHandlerFactory, capabilities);
      QueryMapEncoder queryMapEncoder = Capability.enrich(this.queryMapEncoder, capabilities);
	
      // 初始化SynchronousMethodHandler.Factory工厂，后续使用该工厂生成代理对象的方法
      SynchronousMethodHandler.Factory synchronousMethodHandlerFactory =
          new SynchronousMethodHandler.Factory(client, retryer, requestInterceptors, logger,
              logLevel, decode404, closeAfterDecode, propagationPolicy, forceDecoding);
      // 请求参数解析对象以及参数处理对象。负责根据请求类型构建对应的请求参数处理器
      ParseHandlersByName handlersByName =
          new ParseHandlersByName(contract, options, encoder, decoder, queryMapEncoder,
              errorDecoder, synchronousMethodHandlerFactory);
    // 这里的 ReflectiveFeign 是整个核心的部分
      return new ReflectiveFeign(handlersByName, invocationHandlerFactory, queryMapEncoder);
    }
  }
```

​	执行ReflectiveFeign构建之后，会立马执行该Fegin子类的`ReflectiveFeign#newInstance()`方法。

```
public <T> T target(Target<T> target) {
      return build().newInstance(target);
    }
```

> 这里设计的比较巧妙。但是并不是特别难以理解

```
下面是`ReflectiveFeign#newInstance`方法的代码：
 public <T> T newInstance(Target<T> target) {
     // ParseHandlersByName::apply 方法构建请求参数解析模板和验证handler是否有效
    Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
    Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
    List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();
	// 对于方法handler进行处理
    for (Method method : target.type().getMethods()) {
      if (method.getDeclaringClass() == Object.class) {
        continue;
      } else if (Util.isDefault(method)) {
        DefaultMethodHandler handler = new DefaultMethodHandler(method);
        defaultMethodHandlers.add(handler);
        methodToHandler.put(method, handler);
      } else {
        methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
      }
    }
     // 创建接口代理对象。factory在父类build方法进行初始化
    InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),
        new Class<?>[] {target.type()}, handler);
	// 绑定代理对象
    for (DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
      defaultMethodHandler.bindTo(proxy);
    }
    return proxy;
  }
```

​	下面就上面这段代码进行深入的剖析。

### 2. ParseHandlersByName 参数解析处理 - apply()

​	`ReflectiveFeign#newInstance()`当中首先执行的是`feign.ReflectiveFeign.ParseHandlersByName`对象的`aplly()`方法，进行参数解析和参数解析构建器的构建。同时可以注意到，如果发现`methodHandler` 没有在**feign**中找到对应配置，会抛出`IllegalStateException`异常。

```
public Map<String, MethodHandler> apply(Target target) {
    	// 2.1 小节进行讲解
      List<MethodMetadata> metadata = contract.parseAndValidateMetadata(target.type());
      Map<String, MethodHandler> result = new LinkedHashMap<String, MethodHandler>();
      for (MethodMetadata md : metadata) {
        BuildTemplateByResolvingArgs buildTemplate;
          // 根据请求参数的类型，实例化不同的请求参数构建器
        if (!md.formParams().isEmpty() && md.template().bodyTemplate() == null) {
          // form表单提交形式
            buildTemplate =
              new BuildFormEncodedTemplateFromArgs(md, encoder, queryMapEncoder, target);
        } else if (md.bodyIndex() != null) {
            // 普通编码形式处理
          buildTemplate = new BuildEncodedTemplateFromArgs(md, encoder, queryMapEncoder, target);
        } else {
          buildTemplate = new BuildTemplateByResolvingArgs(md, queryMapEncoder, target);
        }
        if (md.isIgnored()) {
          result.put(md.configKey(), args -> {
            throw new IllegalStateException(md.configKey() + " is not a method handled by feign");
          });
        } else {
          result.put(md.configKey(),
              factory.create(target, md, buildTemplate, options, decoder, errorDecoder));
        }
      }
      return result;
    }
  }
```

#### 2.1 Contract 方法参数注解解析和校验 - parseAndValidateMetadata()

​	此方法的作用是：**调用以解析链接到HTTP请求的类中的方法**。

​	默认实例化对象为：**SpringMvcContract**

```
由于这部分涉及子父类的调用以及多个内部方法的调用并且方法内容较多，下面先介绍下**父类**的`parseAndValidateMetadata()`大致的代码工作流程。
```

1. 检查handler是否为**单继承**（单实现接口），并且**不支持参数化类型**。否则将会抛出异常

2. 遍历所有的内部方法

   1. 如果是静态方法跳过当前循环
   2. 获取method对象以及目标class，执行内部方法`parseAndValidateMetadata()`

   > 内部方法为处理注解方法和参数内容，感兴趣可以自行了解源代码

3. 检查是否为重写方法，如果是则抛出异常`Overrides unsupported`

根据上面的介绍，下面看一下具体的逻辑代码：

```
public List<MethodMetadata> parseAndValidateMetadata(Class<?> targetType) {
      checkState(targetType.getTypeParameters().length == 0, "Parameterized types unsupported: %s",
          targetType.getSimpleName());
      checkState(targetType.getInterfaces().length <= 1, "Only single inheritance supported: %s",
          targetType.getSimpleName());
      if (targetType.getInterfaces().length == 1) {
        checkState(targetType.getInterfaces()[0].getInterfaces().length == 0,
            "Only single-level inheritance supported: %s",
            targetType.getSimpleName());
      }
      final Map<String, MethodMetadata> result = new LinkedHashMap<String, MethodMetadata>();
      for (final Method method : targetType.getMethods()) {
        if (method.getDeclaringClass() == Object.class ||
            (method.getModifiers() & Modifier.STATIC) != 0 ||
            Util.isDefault(method)) {
          continue;
        }
          // 调用内部方法, 处理注解方法和参数信息
        final MethodMetadata metadata = parseAndValidateMetadata(targetType, method);
        checkState(!result.containsKey(metadata.configKey()), "Overrides unsupported: %s",
            metadata.configKey());
        result.put(metadata.configKey(), metadata);
      }
      return new ArrayList<>(result.values());
    }
```

#### 2.2 SpringMvcContract 方法参数注解解析和校验

```
	public MethodMetadata parseAndValidateMetadata(Class<?> targetType, Method method) {
		processedMethods.put(Feign.configKey(targetType, method), method);
        // 使用父类方法获取 MethodMetadata
		MethodMetadata md = super.parseAndValidateMetadata(targetType, method);

		RequestMapping classAnnotation = findMergedAnnotation(targetType,
				RequestMapping.class);
		if (classAnnotation != null) {
			// produces - use from class annotation only if method has not specified this
            // produces - 只有当方法未指定时才从类注释产生
			if (!md.template().headers().containsKey(ACCEPT)) {
				parseProduces(md, method, classAnnotation);
			}

			// consumes -- use from class annotation only if method has not specified this
            // consumes - 只有当method没有指定时才使用from类注释
			if (!md.template().headers().containsKey(CONTENT_TYPE)) {
				parseConsumes(md, method, classAnnotation);
			}

			// headers -- class annotation is inherited to methods, always write these if
			// present
            // headers -- 类注解被继承到方法，如果有的话，一定要写下来
			parseHeaders(md, method, classAnnotation);
		}
		return md;
	}
```

### 3. 创建接口动态代理

​	下面根据一个动态代理的结构图来理解feign是如何完成创建接口的代理对象的。

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210526144216.png?ynotemdtimestamp=1625580577970)

​	首先target就是我们想要调用的目标服务的方法，在进过contract的注解处理之后，会交给proxy对象创建代理对象：

```
InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),
        new Class<?>[] {target.type()}, handler);
```

​	在这里的代码利用工厂构建一个`InvocationHandler`实例，然后再使用`proxy.newInstance`根据代理目标方法对象的类型构建接口代理对象。

​	而`invocationHandler`的构建操作由`InvocationHandlerFactory`工厂构建而成，而工厂的构建细节又由`ReflectiveFeign.FeignInvocationHandler`决定。最终返回`FeignInvocationHandler`完成动态代理的后续代理动作和内容处理。

```
static final class Default implements InvocationHandlerFactory {

  @Override
  public InvocationHandler create(Target target, Map<Method, MethodHandler> dispatch) {
    return new ReflectiveFeign.FeignInvocationHandler(target, dispatch);
  }
}
```

​	创建接口代理对象之后，会执行FeignInvocationHandler 的`invoke()`方法，

```
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  if ("equals".equals(method.getName())) {
    try {
      Object otherHandler =
          args.length > 0 && args[0] != null ? Proxy.getInvocationHandler(args[0]) : null;
      return equals(otherHandler);
    } catch (IllegalArgumentException e) {
      return false;
    }
  } else if ("hashCode".equals(method.getName())) {
    return hashCode();
  } else if ("toString".equals(method.getName())) {
    return toString();
  }
 // 通过dispatch 获取所有方法的handler的引用，执行具体的handler方法
  return dispatch.get(method).invoke(args);
}
```

> 这里涉及了一个数据结构：
>
> `Map<Method, MethodHandler> methodToHandler`，也是动态代理的核心部分
>
> MehtodHandler 是一个 LinkedHashMap的数据结构，他存储的了所有的方法对应接口代理对象的映射。
>
> 此属性由`new ReflectiveFeign.FeignInvocationHandler(target, dispatch);`创建。



#### 3.1 接口代理对象调用`feign.SynchronousMethodHandler#invoke()`请求逻辑

​	到了这一步，就是代理对象执行具体请求逻辑的部分了，这一部分包括创建一个请求模板，参数解析，根据参数配置client，请求编码和请求解码，以及拦截器等等.....涉及的内容比较多。这个小节作为1-3这三个部分的一个分割线。

### 4. SynchronousMethodHandler动态代理对象处理详解

​	首先我们看下整改**SynchronousMethodHandler**的`invoke()`处理代码逻辑：

​	这里还是比较容易理解的，最开始先构建一个`requestTemplate`模板，同时构建请求的相关**option**，复制一个重试器配置给当前的线程使用。然后是核心的`executeAndDecode()`对于请求进行解码和返回结果，如果整个请求执行过程出现重试异常，则尝试调用重试器进行处理，如果重试依然失败，则抛出未受检查的异常或者抛出受检查的异常。最后根据日志的配置登记判断日志的打印和处理。

```
public Object invoke(Object[] argv) throws Throwable {
    // 构建请求处理模板
    RequestTemplate template = buildTemplateFromArgs.create(argv);
    // 配置接口请求参数
    Options options = findOptions(argv);
    // 重试器创建
    Retryer retryer = this.retryer.clone();
    while (true) {
      try {
          // 执行请求
        return executeAndDecode(template, options);
      } catch (RetryableException e) {
        try {
            // 尝试重试和处理
          retryer.continueOrPropagate(e);
        } catch (RetryableException th) {
            // 受检异常处理
          Throwable cause = th.getCause();
          if (propagationPolicy == UNWRAP && cause != null) {
            throw cause;
          } else {
            throw th;
          }
        }
          // 日志打印和处理
        if (logLevel != Logger.Level.NONE) {
          logger.logRetry(metadata.configKey(), logLevel);
        }
        continue;
      }
    }
  }
```

> 下面是阅读源码时临时做的部分笔记，大致浏览即可。
>
> 1. 通过`methodHandlerMap` 分发到不同的请求实现处理器当中
>
> 2. 默认走`SynchronousMethodHandler` 处理不同的请求
>
>    - 构建`requestTemplate` 模板
>    - 构建`requestOptions` 配置
>    - 获取重试器`Retry`
>
> 3. 使用while(true) 进行无线循环. 执行请求并且对于请求的**template**和请求参数进行**decode**处理
>
>    - 调用拦截器对于请求进行拦截处理（使用了责任链模式）
>      - `BasicAuthRequestInterceptor`：默认的调用权限验证拦截
>      - `FeignAcceptGzipEncodingInterceptor` gzip编码处理开关连接器。用于判断是否允许开启gzip压缩
>      - `FeignContentGzipEncodingInterceptor`：请求报文内容gzip压缩拦截处理器
>
>    > 如果日志的配置等级不为none，进行对应日志级别的输出
>
> 4. 执行`client.execute()` 方法，发送http请求
>
>    - 使用`response.toBuilder` 对于响应内容进行构建起的处理（注意源代码里面标注后续版本会废弃这种方式? **为什么要废弃？** **那里不好**）
>
> 5. 对于返回结果解码，调用`AsyncResponseHandler.handlerResponse`对于结果进行处理
>
>    - 这里的判断逻辑比较多，判断的顺序如下：
>      - 如果返回类型为`Response.class`
>      - 如果`Body`内容为null，执行complete调用
>
> > 这里使用了**CompletableFuture** 异步调用处理执行结果。保证整个处理过程是异步执行并且返回的
> >
> > - CompletableFuture.complete()、
> > - CompletableFuture.completeExceptionally 只能被调用一次需要注意。
>
> 如果长度为空或者长度超过 **缓存结果最大长度。\**需要设置`shouldClose`为\**false**，并且同样执行complete调用
>
> - 如果返回状态大于
>
>   200
>
>   并且小于
>
>   300
>
>   - 如果是void返回类型，直接调用`complete`
>   - 否则对于返回结果进行解码，是否需要关闭根据解码之后的结果状态决定**（没看懂）**
>   - 如果是404 并且返回值不为void，则错误处理方法
>   - 如果上述都不满足，根据返回结果的错误信息封装错误结果，并且根据错误结果构建错误对象。最后通过：`resultFuture.completeExceptionally`进行处理
>
> > 特殊处理：如果上面的所有判断出现异常信息，除开io异常需要二次封装处理之外，都会触发默认的`comoleteExceptionally` 方法抛出一个终止异步线程的调用.
>
> ​	+ 验证任务是否完成，如果没有完成任务，调用 `resultFuture.join()` 方法将会在当前线程抛出一个未受检查的异常。
>
> 1. 如果抛出异常，使用retry进行定时重试



#### 构建RequestTemplate模板

​	作用是使用传递给方法调用的参数来创建请求模板。主要内容为请求的各种url处理包括参数处理，url参数处理，对于迭代参数进行展开等等操作。这部分细节处理比较多，由于篇幅有限这里挑重点讲一下：`RequestTemplate template = resolve(argv, mutable, varBuilder);`这个方法，这里会根据事先定义的参数处理器处理参数，具体的代码如下：

```
RequestTemplate resolve(Object[] argv,
                                      RequestTemplate mutable,
                                      Map<String, Object> variables) {
      return mutable.resolve(variables);
    }
```

​	内部调用的是`mutable`对象的`resolve`方法，那么它又是如何处理请求的呢？

##### 根据不同的参数请求模板进行处理:

​	feign通过不同的参数请求模板提供多样化的参数请求处理。 下面先看一下具体的构造图:

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210526181412.png?ynotemdtimestamp=1625580577970)

​	这里很明显使用了策略模式，代码先根据参数找到具体的参数请求处理对象对于参数进行自定义的处理，在处理完成之后，**调用super.resolve()**进行其他内容统一处理（模板方法）。设计的十分优秀并且巧妙，下面是对应的方法签名：

```
`feign.RequestTemplate#resolve(java.util.Map<java.lang.String,?>)`
```

> 这里可能会有疑问，这个**BuildTemplateByResolvingArgs**是在哪里被初始化的？
>
> ```
> BuildTemplateByResolvingArgs buildTemplate;
> // 根据请求参数的类型，实例化不同的请求参数构建器
> if (!md.formParams().isEmpty() && md.template().bodyTemplate() == null) {
>     // form表单提交形式
>     buildTemplate =
>         new BuildFormEncodedTemplateFromArgs(md, encoder, queryMapEncoder, target);
> } else if (md.bodyIndex() != null) {
>     // 普通编码形式处理
>     buildTemplate = new BuildEncodedTemplateFromArgs(md, encoder, queryMapEncoder, target);
> } else {
>     // 使用默认的处理模板
>     buildTemplate = new BuildTemplateByResolvingArgs(md, queryMapEncoder, target);
> }
> ```
>
> 解答：其实早在第二步**ParseHandlersByName**这一步就对于整个请求处理模板进行确认，同时代理对象也会沿用此处理模板保证请求的幂等性.

##### 请求参数处理细节对比：

​	如果是**form表单**提交的参数:

```
Map<String, Object> formVariables = new LinkedHashMap<String, Object>();
      for (Entry<String, Object> entry : variables.entrySet()) {
        if (metadata.formParams().contains(entry.getKey())) {
          formVariables.put(entry.getKey(), entry.getValue());
        }
      }
```

​	如果**form**格式，一般会将map转为**formVariables** 的格式，注意内部使用的是**LinkedHashMap**进行处理的

​	如果是**Body**的处理方式：

```
Object body = argv[metadata.bodyIndex()];
      checkArgument(body != null, "Body parameter %s was null", metadata.bodyIndex());
```

> 注意：
>
> 这部分后续的版本可能会增加更多的处理形式，一切以最新的源码为准。注意文章标题声明的版本

##### 关于报文数据编码和解码的细节：

​	加密的工作是在: **requestTemplate**当中完成的，并且是在**BuildTemplateByResolvingArgs#resolve**中进行处理，根据不同的请求参数类型进行细微的加密操作调整，但是代码基本类似.

​	下面是**Encoder**接口的默认实现：

```
class Default implements Encoder {

    @Override
    public void encode(Object object, Type bodyType, RequestTemplate template) {
      if (bodyType == String.class) {
        template.body(object.toString());
      } else if (bodyType == byte[].class) {
        template.body((byte[]) object, null);
      } else if (object != null) {
        throw new EncodeException(
            format("%s is not a type supported by this encoder.", object.getClass()));
      }
    }
  }
```

1. 如果是字符串类型，则调用对象的tostring 方法
2. 如果是字节数组则转为字节数组进行存储
3. 如果对象为空，则抛出加密encode异常

说完了加密，自然也要说下解码的动作如何处理的，下面是默认的解码接口的实现（注意父类是StringDecoder而不是Decoder）：

```
public class Default extends StringDecoder {

    @Override
    public Object decode(Response response, Type type) throws IOException {
        // 这里的硬编码感觉挺突兀的，不知道是否为设计有失误还是单纯程序员偷懒。
        // 比较倾向于加入 if(response == null ) return null; 这一段代码
      if (response.status() == 404 || response.status() == 204)
        return Util.emptyValueOf(type);
      if (response.body() == null)
        return null;
      if (byte[].class.equals(type)) {
        return Util.toByteArray(response.body().asInputStream());
      }
      return super.decode(response, type);
    }
  }
```

> 这里很奇怪居然用了硬编码的形式。（老外编码总是十分自由）当返回状态为404或者204的时候。则根据对象的数据类型构建相关的数据类型默认值，如果是对象则返回一个空对象
>
> - 204编码代表了空文件的请求
> - 200代表成功响应请求

​	最后一行表示如果类型都不符合情况下使用父类 **StringDecoder** 字符串的类型解码的操作，如果字符串无法解码，则抛出异常信息。感兴趣可以看下`StringDecoder#decode()`的实现细节，这里不再展示。

##### 如果发生错误，如何对错误信息进行编码？

```
public Exception decode(String methodKey, Response response) {
      FeignException exception = errorStatus(methodKey, response);
      Date retryAfter = retryAfterDecoder.apply(firstOrNull(response.headers(), RETRY_AFTER));
      if (retryAfter != null) {
        return new RetryableException(
            response.status(),
            exception.getMessage(),
            response.request().httpMethod(),
            exception,
            retryAfter,
            response.request());
      }
      return exception;
    }
```

1. 根据错误信息和方法签名，构建异常对象
2. 使用重试编码进行返回请求头的处理动作，开启失败之后的稍后重试操作
3. 如果稍后重试失败，则抛出相关异常
4. 返回异常信息

### 4.2 option配置获取

​	代码比较简单，这里直接展开了，如果没有调用参数，返回默认的option陪孩子，否则按照制定条件构建Options配置

```
 Options findOptions(Object[] argv) {
    if (argv == null || argv.length == 0) {
      return this.options;
    }
    return Stream.of(argv)
        .filter(Options.class::isInstance)
        .map(Options.class::cast)
        .findFirst()
        .orElse(this.options);
  }
```

### 4.3 构建重试器

​	重试器这部分会调用一个叫做`clone()`的方法，注意这个clone方法是被重写过的，使用的是默认实现的重试器。另外，个人认为这个方法的起名容易造成误解，个人比较倾向于构建一个叫做`new Default()`的构造函数。

```
 public Retryer clone() {
      return new Default(period, maxPeriod, maxAttempts);
    }
```

​	异常重试代码如下，逻辑比较简单，大致浏览即可

```
 public void continueOrPropagate(RetryableException e) {
      if (attempt++ >= maxAttempts) {
        throw e;
      }

      long interval;
      if (e.retryAfter() != null) {
        interval = e.retryAfter().getTime() - currentTimeMillis();
        if (interval > maxPeriod) {
          interval = maxPeriod;
        }
        if (interval < 0) {
          return;
        }
      } else {
        interval = nextMaxInterval();
      }
      try {
        Thread.sleep(interval);
      } catch (InterruptedException ignored) {
        Thread.currentThread().interrupt();
        throw e;
      }
      sleptForMillis += interval;
    }
```

​	这里的重试间隔按照**1.5的倍数**进行重试，如果超过重试设置的最大因子数则停止重试。

### 4.4 请求发送和结果处理

​	当进行上面的基础配置之后紧接着就是执行请求的发送操作了，在发送只求之前还有一步关键的操作：**拦截器处理**

​	这里会遍历事先配置的拦截器，**对于请求模板做最后的处理操作**。

```
Request targetRequest(RequestTemplate template) {
  for (RequestInterceptor interceptor : requestInterceptors) {
    interceptor.apply(template);
  }
  return target.apply(template);
}
```

#### 关于日志输出级别的控制

​	执行请求这部分代码当中，会出现比较多类似下面的代码。

```
if (logLevel != Logger.Level.NONE) {
      logger.logRequest(metadata.configKey(), logLevel, request);
    }
```

​	关于日志输出的级别根据如下的内容：

```
public enum Level {
    /**
     * No logging.
     	不进行打印，也是默认配置
     */
    NONE,
    /**
     * Log only the request method and URL and the response status code and execution time.
     	只记录请求方法和URL以及响应状态代码和执行时间。
     */
    BASIC,
    /**
     * Log the basic information along with request and response headers.
     	记录基本信息以及请求和响应头。
     */
    HEADERS,
    /**
     * Log the headers, body, and metadata for both requests and responses.
     	记录请求和响应的头、主体和元数据。
     */
    FULL
  }
```

#### client发送请求（重点）

​	这里同样截取了`feign.SynchronousMethodHandler#executeAndDecode`的部分代码，毫无疑问最关键的部分是`client.execute(request, options)`方法。下面是对应的代码内容：

```
Response response;
long start = System.nanoTime();
try {
  response = client.execute(request, options);
  // ensure the request is set. TODO: remove in Feign 12
  response = response.toBuilder()
      .request(request)
      .requestTemplate(template)
      .build();
} catch (IOException e) {
  if (logLevel != Logger.Level.NONE) {
    logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime(start));
  }
  throw errorExecuting(request, e);
}
long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);
```

​	下面是**client**对象的继承结构图：

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210527141927.png?ynotemdtimestamp=1625580577970)

​	根据上面的结构图，简单说明**client**的默认实现：

1. 请求方策略实现，定义顶层接口 **client**，在默认的情况下使用**Default** 类作为实现类。通过子类**proxied**对象实现 **[java.net](http://java.net)** 的**URL请求方式**。也就是说即使没有任何的辅助三方工具，也可以通过此方法api模拟构建http请求。
2. 可以使用**okhttp**和**httpclient** 高性能实现进行替代，需要引入对应的feign接入实现。

**client对应的Default代码逻辑：**

- 构建请求URL对象**HttpUrlConnection**
- 如果是Http请求对象，可以根据条件设置ssl或者域名签名
- 设置http基本请求参数
- 收集Header信息，设置GZIP压缩编码
- 设置accept：*/*
- 检查是否开启内部缓冲，如果设置了则按照指定长度缓冲

​	代码调用的核心部分，默认按照**[java.net](http://java.net)**的**httpconnection** 进行处理。使用原始的网络IO流进行请求的处理，效率比较低下面是对应的具体实现代码：

```
public Response execute(Request request, Options options) throws IOException {
      HttpURLConnection connection = convertAndSend(request, options);
      return convertResponse(connection, request);
    }
```

​	通过数据转化和请求发送之后下面根据结果进行响应内容的封装和处理：

```
// 请求结果处理
Response convertResponse(HttpURLConnection connection, Request request) throws IOException {
    int status = connection.getResponseCode();
    String reason = connection.getResponseMessage();
	// 状态码异常处理
    if (status < 0) {
        throw new IOException(format("Invalid status(%s) executing %s %s", status,
                                     connection.getRequestMethod(), connection.getURL()));
    }
	// 请求头的处理
    Map<String, Collection<String>> headers = new LinkedHashMap<>();
    for (Map.Entry<String, List<String>> field : connection.getHeaderFields().entrySet()) {
        // response message
        if (field.getKey() != null) {
            headers.put(field.getKey(), field.getValue());
        }
    }
	
    Integer length = connection.getContentLength();
    if (length == -1) {
        length = null;
    }
    InputStream stream;
    // 对于状态码400以上的内容进行错误处理
    if (status >= 400) {
        stream = connection.getErrorStream();
    } else {
        stream = connection.getInputStream();
    }
    // 构建返回结果
    return Response.builder()
        .status(status)
        .reason(reason)
        .headers(headers)
        .request(request)
        .body(stream, length)
        .build();
}
```

> 小插曲：关于reason属性（可以跳过）
>
> ​	查看源代码的时候无意间看到这里有一个个人比较在意的点，下面是respose中有一个叫做reason的字段：
>
> ```
> /**
>  * Nullable and not set when using http/2
>  * 作者如下说明 在http2中可以不设置改属性
>  * See https://github.com/http2/http2-spec/issues/202
>  */
> public String reason() {
>   return reason;
> }
> ```
>
> ​	看到这一段顿时有些好奇**为什么不需要设置reason**，当然github上面也有类似的提问。
>
> 这个老哥是在2013年是这么回答的，直白翻译就是：**关我卵事**！
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210527115406.png?ynotemdtimestamp=1625580577970)
>
> 然而事情没有结束，后面又有人详细的进行了提问
>
> ```
> 原文
> i'm curious what was the logical reason for dropping the reason phrase?
> i was using the reason phrase as a title for messages presented to a user in the web browser client. i think most users are accustomed to such phrases, "Bad Request", "Not Found", etc. Now I will just have to write a mapping from status codes to my own reason phrases in the client.
> 机翻：
> 我很好奇，放弃"reason"这个词的逻辑原因是什么? 我使用“reason”作为在web浏览器客户端向用户呈现的消息的标题。我认为大多数用户习惯于这样的短语，“错误请求”，“未找到”等。现在我只需要在客户机中编写一个从状态代码到我自己的理由短语的映射。
> ```
>
> 然后估计是受不了各种提问，上文的**mnot**五年后给出了一个明确的回答：
>
> ![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210527120005.png?ynotemdtimestamp=1625580577970)
>
> ```
> 原因短语——即使在HTTP/1.1中——也不能保证端到端携带;
> 实现可以(也确实)忽略它并替换自己的值(例如，200总是“OK”，不管在网络上发生什么)。
> 
> 考虑到这一点，再加上携带额外字节的开销，将其从线路上删除是有意义的。
> ```
>
> 为了证实他的说法，从 **https://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html** w3c的网站中找到的如下的说明：
>
> ```
> The Status-Code is intended for use by automata and the Reason-Phrase is intended for the human user. The client is not required to examine or display the Reason- Phrase.
> 状态代码用于自动机，而原因短语用于人类用户。客户端不需要检查或显示原因-短语。
> ```
>
> 这一段来源于Http1.1的规范描述。
>
> **所以有时候能从源码发掘出不少的故事，挺有趣的**

#### FeignBlockingLoadBalancerClient 作为负载均衡使用：

​	这个类相当于openFeign和ribbon的中转类，将openfeign的请求转接给ribbon实现负载均衡。到这里会有一个疑问：client是如何做出选择使用ribbon还是spring cloud的呢的呢？

​	其实仔细想想不难理解，负载均衡肯定是在spring bean初始化的时候完成的。**FeignClientFactoryBean**是整个实现的关键。

```
class FeignClientFactoryBean implements FactoryBean<Object>, InitializingBean, ApplicationContextAware 
```

​	下面是`org.springframework.cloud.openfeign.FeignClientFactoryBean#getTarget`方法代码

```
@Override

  public Object getObject() **throws** Exception {

    return getTarget();

  }

/**

   \* @param <T> the target type of the Feign client 客户端的目标类型

   \* @return a {@link Feign} client created with the specified data and the context 指定数据或者上下文

   \* information

   */

  <T> T getTarget() {

    FeignContext context = applicationContext.getBean(FeignContext.class);

    Feign.Builder builder = feign(context);

       // 如果URL为空，默认会尝试使用**

    if (!StringUtils.hasText(url)) {

      if (!name.startsWith("http")) {

        url = "http://" + name;

      }

      else {

        url = name;

      }

      url += cleanPath();

  // **默认使用ribbon作为负载均衡，如果没有找到，会抛出异常**

      return (T) loadBalance(builder, context,

          new HardCodedTarget<>(type, name, url));

    }

    if (StringUtils.hasText(url) && !url.startsWith("http")) {

      url = "http://" + url;

    }

    String url = this.url + cleanPath();

    Client client = getOptional(context, Client.class);

// 根据当前的系统设置实例化不同的负载均衡器

    if (client != null) {

      if (client instanceof LoadBalancerFeignClient) {

        // not load balancing because we have a url,but ribbon is on the classpath, so unwrap
          // 不是负载平衡，因为我们有一个url，但是ribbon在类路径上，所以展开
        client = ((LoadBalancerFeignClient) client).getDelegate();

      }

      if (client instanceof FeignBlockingLoadBalancerClient) {

        // not load balancing because we have a url, but Spring Cloud LoadBalancer is on the classpath, so unwrap
          // 不是负载平衡，因为我们有一个url，但Spring Cloud LoadBalancer是在类路径上，所以展开

        client = ((FeignBlockingLoadBalancerClient) client).getDelegate();

      }

      builder.client(client);

    }

    Targeter targeter = get(context, Targeter.class);

    return (T) targeter.target(this, builder, context,

        new HardCodedTarget<>(type, name, url));

  }
```

​	上面的内容描述了一个负载均衡器的初始化的完整过程。也证明了**spring cloud 使用 ribbon 作为默认的初始化**，感兴趣可以全局搜索一下这一段异常，间接说明默认使用的是ribbon作为负载均衡：

```
throw new IllegalStateException("No Feign Client for defined. Did you forget to include spring-cloud-starter-netflix-ribbon?");
```

> 拓展：
>
> ​	在feign.Client.Default#convertAndSend()，有一段如下的代码设置
>
> ​	`connection.setChunkedStreamingMode(8196);`
>
> ​	如果在代码中禁用ChunkedStreamMode，与设置4096的代码相比有什么效果？
>
> ​	**这样做的结果是整个输出都被缓冲，直到关闭为止，这样Content-length标头可以被首先设置和发送，这增加了很多延迟和内存。对于大文件，不建议使用。**
>
> 答案来源：[HttpUrlConnection.setChunkedStreamingMode的效果](https://stackoverflow.com/questions/46249401/effect-of-httpurlconnection-setchunkedstreamingmode)

#### 关于编解码的处理

​	这一部分请阅读4.1 部分的**关于报文数据编码和解码的细节**部分内容

至此一个基本的调用流程基本就算是完成了。

## openFeign 整体调用链路图

​	先借（偷）一张参考资料的图来看下整个openFeign的链路调用：

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210527160429.png?ynotemdtimestamp=1625580577970)

​	下面是个人根据资料自己画的图：

![img](https://gitee.com/lazyTimes/imageReposity/raw/master/img/openFeign%E8%B0%83%E7%94%A8%E9%93%BE%E8%B7%AF.png?ynotemdtimestamp=1625580577970)

## openFeign注解处理流程

​	我们先看下开启openFeign的方式注解：**@EnableFeignClients**

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {}
```

​	注意这里的一个注解`@Import(FeignClientsRegistrar.class)`。毫无疑问，实现的细节在`FeignClientsRegistrar.class`内部：

​	剔除掉其他的逻辑和细节，关键代码在这一块：

```
for (String basePackage : basePackages) {
         //….
registerFeignClient(registry, annotationMetadata, attributes);
	        //….
        }
```

​	这里调用了`registerFeignClient`注册feign，根据注解配置扫描得到响应的basepakage，如果没有配置，则默认按照注解所属类的路径进行扫描。

​	下面的代码根据扫描的结果注入相关的bean信息，比如url，path，name，回调函数等。最后使用BeanDefinitionReaderUtils 对于bean的方法和内容进行注入。

```
private void registerFeignClient(BeanDefinitionRegistry registry,
            AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
        String className = annotationMetadata.getClassName();
    //bean配置
    
        BeanDefinitionBuilder definition = BeanDefinitionBuilder
                .genericBeanDefinition(FeignClientFactoryBean.class);
        validate(attributes);
        definition.addPropertyValue("url", getUrl(attributes));
        definition.addPropertyValue("path", getPath(attributes));
        String name = getName(attributes);
        definition.addPropertyValue("name", name);
        String contextId = getContextId(attributes);
        definition.addPropertyValue("contextId", contextId);
        definition.addPropertyValue("type", className);
        definition.addPropertyValue("decode404", attributes.get("decode404"));
        definition.addPropertyValue("fallback", attributes.get("fallback"));
        definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
        definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
 
        String alias = contextId + "FeignClient";
        AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();
        beanDefinition.setAttribute(FactoryBean.OBJECT_TYPE_ATTRIBUTE, className);
 
        // has a default, won't be null
    	// 如果未配置会存在默认的配置
        boolean primary = (Boolean) attributes.get("primary");
 
        beanDefinition.setPrimary(primary);
 
        String qualifier = getQualifier(attributes);
        if (StringUtils.hasText(qualifier)) {
            alias = qualifier;
        }
 
        BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
                new String[] { alias });
    	// 注册Bean
        BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
    }
```

​	看完了基本的注册机制，我们再来看看Bean是如何完成自动注入的：这里又牵扯到另一个注解-@FeignAutoConfiguration

### @FeignAutoConfiguration 简单介绍

​	关于feign的注入，在此类中提供了两种的形式：

- 如果存在**HystrixFeign**，则使用 **HystrixTargeter** 方法。

- 如果不存在，此时会实例化一个**DefaultTargeter** 作为默认的实现者

  具体的操作代码如下：

  ```
  		@Configuration(proxyBeanMethods = false)
  	    @ConditionalOnClass(name = "feign.hystrix.HystrixFeign")
  	    protected static class HystrixFeignTargeterConfiguration {
  	 
  	        @Bean
              // 优先使用Hystrix
  	        @ConditionalOnMissingBean
  	        public Targeter feignTargeter() {
  	            return new HystrixTargeter();
  	        }
  	 
  	    }
  	 
  	    @Configuration(proxyBeanMethods = false)
  	 	//如果不存在Hystrix，则使用默认的tagerter
  	    @ConditionalOnMissingClass("feign.hystrix.HystrixFeign")
  	    protected static class DefaultFeignTargeterConfiguration {
  	 
  	        @Bean
  	        @ConditionalOnMissingBean
  	        public Targeter feignTargeter() {
  	            return new DefaultTargeter();
  	        }
  	 
      }
  ```

> 复习一下springboot几个核心的注解代表的含义：
>
> - **@ConditionalOnBean** // 当给定的在bean存在时,则实例化当前Bean
> - **@ConditionalOnMissingBean** // 当给定的在bean不存在时,则实例化当前Bean
> - **@ConditionalOnClass** // 当给定的类名在类路径上存在，则实例化当前Bean
> - **@ConditionalOnMissingClass** // 当给定的类名在类路径上不存在，则实例化当前Bea



### 关于HystrixInvocationHandler的invoke方法：

​	`Feign.hystrix.HystrixInvocationHandler` *当中执行的**invoke**实际上还是SyncronizedMethodHandler* *方法*

```
HystrixInvocationHandler.this.dispatch.get(method).invoke(args);
```

​	内部代码同时还使用了命令模式的命令 **HystrixCommand** 进行封装。由于不是本文重点，这里不做扩展。

> **HystrixCommand 这个对象又是拿来干嘛的？**
>
> 简介：用于包装代码，将执行具有潜在风险的功能(通常是指通过网络的服务调用)与故障和延迟容忍，统计和性能指标捕获，断路器和隔板功能。这个命令本质上是一个阻塞命令，但如果与`observe()`一起使用，它提供了一个可观察对象外观。
>
> 实现接口：`HystrixObservable` / `HystrixInvokableInfo`
>
> `HystrixInvokableInfo`: 存储命令接口的规范，子类要求实现
>
> `HystrixObservable`: 变成观察者支持非阻塞调用

# 总结

​	第一次总结源码，更多的是参考网上的资料顺着别人的思路自己去一点点看的。（哈哈，闻道有先后，术业有专攻）如果有错误欢迎指出。

​	不同于spring那复杂层层抽象，openFeign的学习和“模仿”价值更具有意义，很多代码一眼就可以看到设计模式的影子，比较适合自己练手和学习提高个人的编程技巧。

​	另外，openFeign使用了很多的包访问结构，这对于在此基础上二次扩展的sentianl框架是个头疼的问题，不过好在可以站在反射大哥的背后，直接暴力访问。

# 参考资料：

[掘金博客【非常好】](https://juejin.cn/post/6844904066229927950#heading-16)

[关于负载均衡的介绍来源](https://my.oschina.net/wecanweup/blog/3238261)

[官方文档](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#feign-metrics)

> 结合源码再回顾官方文档提到的功能

[在线代码格式化](https://tool.oschina.net/highlight)

[在线画图软件](https://www.websequencediagrams.com/)
