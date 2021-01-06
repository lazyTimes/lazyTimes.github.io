---
title: 一文弄懂spring validate
subtitle: 对象校验是一个十分恼火的问题，但是好在有脚手架可以给我们开箱即用
url_suffix: springvalidate
author: lazytime
tags:
  - spring
categories:
  - java
keywords: springvalidate,spring
description: 对象校验是一个十分恼火的问题，但是好在有脚手架可以给我们开箱即用
copyright: true
date: 2021-01-06 23:48:50
---



[TOC]

# 前言:

校验参数在以前基本都是使用大量的if/else，稍微方便一点的可以使用反射+自定义注解的形式，但是复用性不是很好，并且每个人对于的自定义注解有着自己的使用习惯，不过好在spring开发了validated框架用于注解校验，可以节省很多的校验`ifelse`代码，这篇文章通篇介绍了如何使用spring validated。

<!-- more -->

# 文章目的：

1. 了解 **validate** 校验，以及使用注解对于参数进行快速校验
2. 关于统一全局异常处理，以及一些踩坑问题
3. 如何快速的集成和使用 **spring validate**
4. 讨论`list\<Object\>`校验的问题，分析如何使用list对象内容校验



# 简单介绍

spring Validation 是一种参数检验工具,集成在spring-context包中, 常用于spring mvc中Controller的参数处理,主要针对整个实体类的多个可选域进行判定.对于不合格的数据信息springMVC会把它保存在错误对象中，这些错误信息我们也可以通过SpringMVC提供的标签在前端JSP页面上进行展示。

实现方式，一般使用较多的是两个注解：`@Validated`、`@Valid`，下面简单介绍一下以及分析他们的区别

1. 实现`Validator`,利用`BindingResult`获取Errors信息
2. 采用`@Valid` 以及 `JSR-303`中的参数判定注解

## `@Valid`和`@Validated`区别

| 区别             | @Valid                                          | @Validated              |
| ---------------- | ----------------------------------------------- | ----------------------- |
| 提供者           | JSR-303规范                                     | Spring                  |
| **是否支持分组** | 不支持                                          | 支持                    |
| 标注位置         | METHOD, FIELD, CONSTRUCTOR, PARAMETER, TYPE_USE | TYPE, METHOD, PARAMETER |
| **嵌套校验**     | 支持                                            | 不支持                  |

## 常用注解

网上有很多类似表格，这里直接COPY了一份，对于不同的api版本可能出现部分注解过时等情况，注意！

| meta-data                   | comment              | version             |
| :-------------------------- | :------------------- | :------------------ |
| @Null                       | 对象，为空           | Bean Validation 1.0 |
| @NotNull                    | 对象，不为空         | Bean Validation 1.0 |
| @AssertTrue                 | 布尔，为True         | Bean Validation 1.0 |
| @AssertFalse                | 布尔，为False        | Bean Validation 1.0 |
| @Min(value)                 | 数字，最小为value    | Bean Validation 1.0 |
| @Max(value)                 | 数字，最大为value    | Bean Validation 1.0 |
| @DecimalMin(value)          | 数字，最小为value    | Bean Validation 1.0 |
| @DecimalMax(value)          | 数字，最大为value    | Bean Validation 1.0 |
| @Size(max, min)             | min<=value<=max      | Bean Validation 1.0 |
| @Digits (integer, fraction) | 数字，某个范围内     | Bean Validation 1.0 |
| @Past                       | 日期，过去的日期     | Bean Validation 1.0 |
| @Future                     | 日期，将来的日期     | Bean Validation 1.0 |
| @Pattern(value)             | 字符串，正则校验     | Bean Validation 1.0 |
| @Email                      | 字符串，邮箱类型     | Bean Validation 2.0 |
| @NotEmpty                   | 集合，不为空         | Bean Validation 2.0 |
| @NotBlank                   | 字符串，不为空字符串 | Bean Validation 2.0 |
| @Positive                   | 数字，正数           | Bean Validation 2.0 |
| @PositiveOrZero             | 数字，正数或0        | Bean Validation 2.0 |
| @Negative                   | 数字，负数           | Bean Validation 2.0 |
| @NegativeOrZero             | 数字，负数或0        | Bean Validation 2.0 |
| @PastOrPresent（时间）      | 过去或者现在         | Bean Validation 2.0 |
| @FutureOrPresent（时间）    | 将来或者现在         | Bean Validation 2.0 |



## 快速入门

springvalidation入门使用都十分的简单，基本十分钟不到就能快速的集成，目前springboot的项目已经越来越多，所以本文基本都是基于**springboot**构建的

**第一步：pom.xml 加入注解**

这里为了方便版本控制增加了版本控制配置：

> 注意：hibernate-validate 的版本到本文为止已经出现了7.0.0，这个版本的校验做了不少的改动。
>
> https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/?v=7.0#_validating_constraints

```xml
<properties>
    <hibernate-validate.version>5.2.0.Final</hibernate-validate.version>
</properties>
```

增加完配置之后，增加对应的maven依赖，需要引入如下两个依赖配置

```xml
  <!--jsr 303-->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
<!-- hibernate validator-->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <!-- 注意使用了版本控制 -->
    <version>${hibernate-validate.version}</version>
</dependency>
```

加完这两个注解之后，不要急着先进行其他的编写，可以先进行

**第二步：增加注解**

这里给出一个案例进行参考，验证对象增加注解：

```java
public class TestProduct {
    
    @NotBlank
    @Length(max = 7)
    @Pattern(regexp = "^(([1-9]{1}\\d*)|([0]{1}))(\\.(\\d){0,2})?$")
    private String tranAmount;

    @NotBlank
    @Length(max = 3,min = 3)
    @Pattern(regexp = "^[A-Z]{3}$")
    private String currency;

    @NotBlank
    @Length(max = 100)
    private String tranReason;

    @NotBlank
    @Length(max = 100)
    private String gatherName;

    @NotBlank
    @Length(max = 10)
    private String business_type;

    /
    @NotBlank
    @Length(max = 10)
    private String pay_channel;

   	// .......其他校验
    // 过滤 get/set 方法
}

```

在`controller`层增加`@Validated`注解，加入之后成为如下的效果：

```java
@Controller
@RequestMapping("/test")
public class TestController{
    @RequestMapping(value = "/test", method = RequestMethod.POST)
    @ResponseBody
    public Object create(@Validated @RequestBody Product requestString, BindingResult bindResult) {
        // 统一处理校验注解的错误信息
        Result stringBuilder = dealWithError(bindResult);
        if (stringBuilder != null){
            return stringBuilder;
        }
       // 自己的业务处理...
        return ....;
    }
}

```

**第三步：验证注解是否生效**

到这一步就可以直接请求接口，在接口处进行断点，如果请求正确会直接进入对应的断点，否则会抛出如下案例所示的异常信息，如果校验不通过，会抛出`MethodArgumentNotValidException`或者`ConstraintViolationException`异常，下面是案例：

```json
{
    "timestamp": "2021-01-01T12:08:49.859+00:00",
    "status": 400,
    "error": "Bad Request",
    "trace": "org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object 'product' on field 'name': rejected value [null]; codes [NotBlank.product.name,NotBlank.name,NotBlank.java.lang.String,NotBlank]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [product.name,name]; arguments []; default message ......",
    "message": "Validation failed for object='product'. Error count: 1",
    "errors": [
        {
            "codes": [
                "NotBlank.product.name",
                "NotBlank.name",
                "NotBlank.java.lang.String",
                "NotBlank"
            ],
            "arguments": [
                {
                    "codes": [
                        "product.name",
                        "name"
                    ],
                    "arguments": null,
                    "defaultMessage": "name",
                    "code": "name"
                }
            ],
            "defaultMessage": "不能为空",
            "objectName": "product",
            "field": "name",
            "rejectedValue": null,
            "bindingFailure": false,
            "code": "NotBlank"
        }
    ],
    "path": "//test/valid"
}
```

到此，一个对象的注解校验基本实现了，但是我们发现注解校验的方式抛出的异常信息不是十分友好，基本都会配合统一的异常处理来处理请求参数的问题，下面后文会单独讲如何使用全局异常处理来统一的处理异常信息。





## 自定义注解校验：

如果默认的注解规则无法满足业务需求，这时候validator提供了自定义注解的形式帮助开发者可以进行自定的规则校验。

第一步：定义自定义注解：

首先第一步是确定自己需要自定义的注解：比如我这里定义了一个检查时间格式的注解

```java
/**
 * 日期格式校验注解
 */
import org.hibernate.validator.constraints.EAN;
import org.hibernate.validator.constraints.NotBlank;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;
// 注意这里有静态导入
import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

// javadoc 文档标识
@Documented
// 可以注入的类型，字段和参数类型
@Target({PARAMETER, FIELD})
// 运行时生效
@Retention(RUNTIME)
// 触发校验的对象
@Constraint(validatedBy = {TimeValidator.class})
public @interface Time {

    // 必须
    String message() default "时间格式校验失败";

    // 必须
    Class<?>[] groups() default {};

    // 必须
    Class<? extends Payload>[] payload() default {};
    
    String value = "";
	
    // 下面部分可以忽略
    @Target({ FIELD, METHOD, PARAMETER, ANNOTATION_TYPE })
    @Retention(RUNTIME)
    @Documented
    @interface List {
        Time[] value();
    }
}

```

`@Constraint`注解声明约束及其可配置属性，同时在对应的真实注解处理类`TimeValidator`里面，可以随意的注入需要的`bean`（`AutoWired`等）

注意除开`value`这个属性之外，其他三个属性`message`、`groups`、`payload`都是**必须要**定义的，否则进行校验的时候，会抛出如下的错误：

```
HV000074: com.xxx.xxx.valid.annotation.Time contains Constraint annotation, but does not contain a groups parameter.
```

对于注解的解释：

+ `@Retention(RUNTIME)`：指定此类型的注释将在运行时通过反射方式可用
+ `@Constraint`：指定用于验证元素的验证器
+ `@Target`：注解的标识范围，比如这里注解可以是参数或者字段

对应的三个固定参数含义：

+ `message` 定制化的提示信息，主要是从ValidationMessages.properties里提取，也可以依据实际情况进行定制

+ `groups `这里主要进行将validator进行分类，不同的类group中会执行不同的validator操作

+ `payload` 主要是针对bean的，使用不多。



第二步：定义真实注解处理类：

需要实现接口`ConstraintValidator`，泛型的第一个参数为注解类，第二个参数为具体校验对象的类型

下面定义校验时间格式是否正确的一个案例，写的非常粗浅。

```java
public class TimeValidator implements ConstraintValidator<Time, String> {

    /**
     * 初始化注解的校验内容
     * @param constraintAnnotation
     */
    @Override
    public void initialize(Time constraintAnnotation) {
        System.err.println("test" + constraintAnnotation);

    }

    /**
    具体校验代码
    */
    @Override
    public boolean isValid(String value, ConstraintValidatorContext constraintContext) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        simpleDateFormat.setLenient(false);
        boolean isValid = true;
        try {
            simpleDateFormat.parse(value);
        } catch (Exception e) {
            isValid = false;
        }
        // 如果校验失败，设置自定义错误信息
        if ( !isValid ) {
            constraintContext.disableDefaultConstraintViolation();
            constraintContext.buildConstraintViolationWithTemplate(
                    "{com.zxd.interview.valid.annotation." +
                            "Time.message}"
            )
            .addConstraintViolation();//很重要的一步，需要将自定义的信息提示模板加入
        }
        return isValid;
    }
}
```

对于`ConstraintValidator`接口内容如下：

```java
public interface ConstraintValidator<A extends Annotation, T> {

	/**
	初始化验证器，为isValid(Object, ConstraintValidatorContext)调用做准备。传递给定约束声明的约束注释。
	保证在使用此实例进行验证之前调用此方法。
	默认的实现是no-op。
	 */
	default void initialize(A constraintAnnotation) {
	}

	/**
	 * 实现验证逻辑。值的状态不能被改变。
	该方法可以并发访问，实现必须确保线程安全。
	@value：被校验的值
	@ConstraintValidatorContext 校验上下文
	 */
	boolean isValid(T value, ConstraintValidatorContext context);
}
```

然后只需要将注解应用到对应的对象上面，在请求的时候就可以进行对应的参数校验了：

```java
public class Product {

    @NotBlank
    private String name;
	// 自己定义的注解
    @Time
    private String time;

    // 省略get/set
}

```

> `HV000074`这个错误是如何来的？：
>
> 首先我们需要明确一点：javax.validator - JSR303的规范是由**Hibernate validate**作为标准实现的，也就是说虽然Spring已经为我们进行了适配，但是在校验的时候依然使用的Hibernate Validator，所以我们定义自定义的注解需要按照固定的要求规范：
>
> 旧版本的文档：https://docs.jboss.org/hibernate/validator/4.2/reference/en-US/html_single/
>
> 较新版本的文档：https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/?v=7.0#_validating_constraints
>
> `org.hibernate.validator.internal.util.logging`下面的几个包中定义了日志以及异常信息：
>
> ```java
> private static final String getConstraintWithoutMandatoryParameterException = "HV000074: %2$s contains Constraint annotation, but does not contain a %1$s parameter.";
> ```
>
> 具体的提示信息如下图所示：`org.hibernate.validator.internal.util.logging.Log_$logger`
>
> 注解定义了如下的异常信息提示
>
> ```java
> @Message(
>      id = 74,
>      value = "%2$s contains Constraint annotation, but does not contain a %1$s parameter."
>  )
>  ConstraintDefinitionException getConstraintWithoutMandatoryParameterException(String var1, String var2);
> ```

> 注意以下几个点：
>
> + 静态字段和属性无法验证。
> + 建议在一个类中坚持使用字段 *或*属性注释。不建议对字段*和*随附的getter方法进行注释*，*因为这将导致对该字段进行两次验证。

## 使用 `Validator` 校验：

下面介绍一下使用`Validator`要如何校验，简单的使用可以使用`Validation.buildDefaultValidatorFactory()`获取`ValidatorFactory`，通过`factory.getValidator()`获取对应的校验器`Validator`：

```java
@Override
public void doSomething(Product product) {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    Validator validator = factory.getValidator();
    Set<ConstraintViolation<Product>> validate = validator.validate(product);
    Map<Object, Object> objectObjectMap = new HashMap<>();
    validate.forEach(item -> {
        //            System.err.println("item = "+ item);
        String message = item.getMessage();
        //            System.err.println("message " + message);
        objectObjectMap.put(item.getPropertyPath(), message);

    });
    objectObjectMap.forEach((k, v) -> {
        System.err.println("key = " + k + " value = " + v);
    });
}
```

下面构建一个工具类：

```java
public class ValidateUtils {

    private static Validator validator =
            Validation.byProvider(HibernateValidator.class)
            .configure()
            .failFast(true)
            .buildValidatorFactory()
            .getValidator();

    private static final SmartValidator validatorAdapter = new SpringValidatorAdapter(validator);


    public static Validator getValidator() {
        return validator;
    }

    private static SmartValidator getValidatorAdapter(Validator validator) {
        return validatorAdapter;
    }

    /**
     * 校验参数，用于普通参数校验 [未测试！]
     *
     * @param
     */
    public static void validateParams(Object... params) {
        Set<ConstraintViolation<Object>> constraintViolationSet = validator.validate(params);

        if (!constraintViolationSet.isEmpty()) {
            throw new ConstraintViolationException(constraintViolationSet);
        }
    }

    /**
     * 校验对象
     *
     * @param object
     * @param groups
     * @param <T>
     */
    public static <T> void validate(T object, Class<?>... groups) {
        Set<ConstraintViolation<T>> constraintViolationSet = validator.validate(object, groups);

        if (!constraintViolationSet.isEmpty()) {
            throw new ConstraintViolationException(constraintViolationSet);
        }
    }

    /**
     * 校验对象
     * 使用与 Spring 集成的校验方式。
     *
     * @param object 待校验对象
     * @param groups 待校验的组
     * @throws BindException
     */
    public static <T> void validateBySpring(T object, Class<?>... groups)
            throws BindException {
        DataBinder dataBinder = getBinder(object);
        dataBinder.validate((Object[]) groups);

        if (dataBinder.getBindingResult().hasErrors()) {
            throw new BindException(dataBinder.getBindingResult());
        }
    }

    private static <T> DataBinder getBinder(T object) {
        DataBinder dataBinder = new DataBinder(object, ClassUtils.getShortName(object.getClass()));
        dataBinder.setValidator(getValidatorAdapter(validator));
        return dataBinder;
    }
}
```

上面的工具类代码来源于文章：https://www.jianshu.com/p/2432d0f51c0e

## 定义分组校验：

有时候我们需要某个对象在这个接口是必填的，而在另一个参数里面又不需要必填，比如我们使用dto接受更新或者新增的参数，新增不需要校验`主键`或者其他的字段信息，但是注解校验器却拦截返回错误信息，这种情况下就需要使用分组校验的方法。

第一步：设置分组接口：

建议继承Default，因为默认的`groups`就是`groups = {Default.class}`

```java
public interface GroupUpdate extends Default {
}
```

第二步：在需要分组校验的注解上增加groups

例如我在对象某个注解增加：

```java
@NotBlank(groups = {GroupUpdate.class})
private String name;
```

第三步：在@validated中加入对应的分组：

这里定义了两个接口来代替新增和修改

```java
// 更新接口
@RequestMapping("/test/update")
public Object update(@Validated(GroupUpdate.class) Product product) throws ParamException {
    System.err.println(product);
     //....
}
// 新增接口
@RequestMapping("/test/add")
public Object add(@Validated Product product) throws ParamException {
    System.err.println(product);
    //....
}
```

第四步：分组校验结果：

按照同样的参数请求两个接口，分组的不同出现了不同的情况

可以看到指定分组之后，如果validated里面没有指定group，在校验的时候将会跳过指定分组的校验

```java
org.springframework.validation.BeanPropertyBindingResult: 2 errors
Field error in object 'product' on field 'name': .....
Field error in object 'product' on field 'time': .....
org.springframework.validation.BeanPropertyBindingResult: 1 errors
Field error in object 'product' on field 'time': .....

```

### 分组继承：

自定义的分组可以使用继承方式进行校验，比如我们将很多个分组封装到一个特定的分组里面，方便我们自由组合多个自定义分组下面请看如下的案例：

首先是实体对象，通过继承的形式的形式，对于校验对象来说继承会将父对象的属性一并校验：

```java
public class Bag extends Product {
	// 分组使用 GroupAdd
    @NotNull(message = "颜色不能为空",groups = {GroupAdd.class})
    private String color;
  	// get/set省略
}

```

定义对应的分组，比如我们将增删改操作的分组集成到一个叫做`操作`的分组里面：

```java
public interface GroupsOpration extends GroupUpdate,GroupDel,GroupAdd{
}
public interface GroupUpdate extends Default {
}
public interface GroupDel extends Default {
}
public interface GroupAdd extends Default {
}
```

下面分别定义对应的接口进行测试

```java
 /**
     * 测试组继承
     * @param product
     * @return
     * @throws ParamException
     */
    @RequestMapping("/test/bag1")
    public Object bag1(@Validated(GroupsOpration.class) Bag product) throws ParamException {
        System.err.println(product);
        return null;
    }

    /**
     * 测试组继承
     * @param product
     * @return
     * @throws ParamException
     */
    @RequestMapping("/test/bag2")
    public Object bag2(@Validated Bag product) throws ParamException {
        System.err.println(product);
        return null;
    }
```

分别请求`/bag1`和`/bag2`得到如下的结果：

```java
Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 2 errors
Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 error     
```







## 如何处理validated异常信息

### 第一种：统一全局异常处理

全局统一异常处理算是用的比较多的一种，可以解决基础的常见异常问题结果都返回，也可以使用

基本都写法和格式如下：

1. 在`类名`注明：`@ControllerAdvice`或者`@RestControllerAdvice`，分别对应`@Controller`和`@RestController`，至于这两个注解的区别可以自行进行学习和补充。

```java
@ControllerAdvice
@@RestControllerAdvice
```

2. 对应的方法内部，使用`@ExceptionHandler`进行方法标注，在请求参数里面配套使用,即可对于指定的异常，如果在参数里面加入特定异常，那么在执行改方法的时候，会将对应的对象进行方法参数注入，这样就可以拿到抛出异常的对象信息进行自定义的异常处理了。

```java
@ExceptionHandler({Exception.class})//主要注解
@ResponseStatus(HttpStatus.OK)
@ResponseBody
public Object allError(Exception ex) {
    return HttpStatus.ACCEPTED;
}
```

3. 下面是一个最终完整的案例

```java
@RestControllerAdvice //@ControllerAdvice 对应 @Controller
public class ExceptionDealHandler {

    /**
     * 拦截未知的运行时异常
     */
    @ExceptionHandler(IllegalStateException.class)
    public Object notFount(IllegalStateException e)
    {
        if (AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class) != null)
        {
            throw e;
        }

        return HttpStatus.ACCEPTED;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Object doSomethings(MethodArgumentNotValidException e){
        return HttpStatus.ACCEPTED;
    }

    @ExceptionHandler(BindException.class)
    public Object bindError(BindException bind){
        return HttpStatus.ACCEPTED;
    }

    @ExceptionHandler({ConstraintViolationException.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Object handleConstraintViolationException(ConstraintViolationException ex) {
        return HttpStatus.ACCEPTED;
    }

    @ExceptionHandler({Exception.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Object allError(Exception ex) {
        return HttpStatus.ACCEPTED;
    }
}

```







#### 统一异常处理无法生效？

1.确保注解@RestControllerAdvice/@ControllerAdvice的类被**spring容器管理到**。

>   ①spring boot Java配置检查@SpringBootApplication(**scanBasePackages** = )（scanBasePackages 配置的包是否包含这个类默认情况下spring boot项目扫描的是@SpringBootApplication注解所在类的包及子包）
>
>   ② xml配置的spring 普通项目检查**<context:component-scan base-package="com.test"/>**

2. 检查项目中所有的切面编程，**是否在某个切面将异常try-catch然后没有扔出来**。常见的就是切面的环绕处理，捕获了异常忘记抛出来。

3. 检查项目中是否有其他的相同的全局异常处理类，例如BaseController中**是否已经定义**了

如果上面的排查方式都都尝试还是无法正常校验的话可能就是出现所谓统一全局处理的坑了，下面会讲解到对应的坑，如果 **问题超过15分钟还没自我通过自己努力尽力解决**，那么就赶紧上网找资料吧，碰运气基本都可以完美解决。



### 第二种：控制器进行特定异常处理

一般这种使用方式比较少，毕竟有全局异常处理的情况下很少去使用控制器层的异常处理，某些特殊情况可以用到，了解即可。

和全局异常处理器类似，只不过定义方法修改到了对应的controller控制器层

```java
@RestController
public class TestValid {

    // 注解校验接口
    @RequestMapping("/test/valid")
    public Object test(@Validated Product product) throws ParamException {
        System.err.println(product);
        throw new ParamException("","","");
    }

    /**
    在控制器层处理异常信息，仅仅适用于当前控制器
    */
    @ExceptionHandler(Exception.class)
    public Object processException(Exception e){
        System.err.println(e.getMessage());
        return null;
    }

}
```

### 第三种：BindingResult对象处理异常信息：

除开上面的方式之外，validate还提供`BindResult`对象封装异常信息，需要将该对象 **紧跟**`@Validated`注解，就可以在校验失败之后在`BindResult`对象里面进行自定义的异常处理

```java
/**
     * 测试组继承
     * @param product
     * @return
     * @throws ParamException
     */
@RequestMapping("/test/bag1")
@ResponseBody
public Object bag1(@Validated(GroupsOpration.class) Bag product, BindingResult bindingResult) throws ParamException {
    // 异常信息处理
    if(bindingResult.hasErrors()){
        List<FieldError> fieldErrors = bindingResult.getFieldErrors();
        Map<Object, Object> objectObjectHashMap = new HashMap<>();
        for (FieldError fieldError : fieldErrors) {
            objectObjectHashMap.put(fieldError.getField(), fieldError.getRejectedValue());
        }
        return objectObjectHashMap;
    }
    System.err.println(product);
    return null;
}
```

请求结果如下：

```json
{
    "color": null,
    "time": "222"
}
```



## 统一全局处理器的坑：

```java
@ExceptionHandler(IllegalStateException.class)
public Object notFount(IllegalStateException e)
{
    // 处理异常结果
}
```

但是如果 @ExceptionHandler 注解中未声明要处理的异常类型，则默认为参数列表中的异常类型。注意只能绑定一个参数，如果在**参数列表指定多个异常参数将无法生效**，比如如下的写法是错误的，为什么会有这种情况，需要研究源码才能的得出结果。

```java
@ExceptionHandler()
public Object notFount(Exception e,BindException bindExce, RuntimeException run) { // 这里指定多个Exception将无法生效
    // 处理异常结果
      return null;
}
```

上面的写法会出现如下的异常，大致的意思是找不到合适的解析器，就是说spring找不到合适的异常解析器去解析分发异常的请求：

```java
Could not resolve parameter [2] in public java.lang.Object com.zxd.interview.valid.ExceptionDealHandler.notFount(java.lang.Exception,org.springframework.validation.BindException,java.lang.RuntimeException): No suitable resolver
```

但是万事没有绝对，下面这种写法是可行的，在抛出`BindException`的异常之后，异常处理器将会正常的处理请求

```java
@ExceptionHandler()
public Object notFount(Exception e,BindException bindExce) {//BindException 正常拦截处理
    // 处理异常结果
    return null;
}
```

目前个人猜测是在定义参数类型的时候，定义的异常类上面出现“雷同”的**构造方法**，而spring在进行反射解析时候找到了对应的重复构造方法，导致无法生成代理对象完成异常处理，导致导致抛出异常。总的来说和spring validate的代理机制有关，有兴趣的小伙伴可以自行研究一下。

```java
/**
自定义异常类1
*/
public class BusinessException extends Exception{
    BusinessException(String code1, String code2, int code3){
		// ......
    }
    // ......
}
/**
自定义异常类2
*/
public class ParamException extends Exception{
    public ParamException(String code, String message, String error){
		// ......
    }
    // ......
}
```

下面这种写法是**错误的**，即使他们的构造方法不同，在抛出异常的时候依然出现了问题

```java
 @ExceptionHandler()
public Object errors(ParamException e, BusinessException busine){//错误的写法
    return HttpStatus.ACCEPTED;
}
```

建议的写法，也是最稳妥的写法：

```java
@ExceptionHandler(ParamException.class)
public Object errors(ParamException e){//错误的写法
    return HttpStatus.ACCEPTED;
}
```

总的来说，还是不建议使用`@ExceptionHandler()`这种形式，会发生各种莫名其妙的问题。



### 总结自定义异常

根据上面的分析可以看出，统一全局异常处理如果不好好处理，很容易出现各种莫名其妙的问题，所以总结一下统一全局异常处理需要注意的点：

+ 建议一个异常处理对应一个方法，不要定义多个异常用一个方法处理，**特别是自定义的异常类**
+ 注意统一异常处理的异常处理优先级按照**方法定义的顺序进行**，比如如果出现BindException以及Exception，如果抛出的异常是BindException处理方法则优先定义则执行这一步，否则执行最大的`Exception`
+ 注意注解的异常拦截和方法参数的异常类**保持一致**，否则spring mvc 代理将抛出异常。







# 怎样校验`list<Object>`（重点）

一般来说sprIng validate使用基本多看看文档或者找找博客都能解决，但是笔者遇到一个很纠结的问题，请看如下的代码：

```java
@RequestMapping(value = "/createBatch", method = RequestMethod.POST)
@ResponseBody
public Object createBatch(@RequestBody @Validated List<ApiPaymentMsgDto> list, BindingResult bindResult, HttpServletRequest httpServletRequest) {
    if (CollectionUtils.isEmpty(list)) {
        return Result.build(401, "请求参数为空");
    }
    //.....省略代码
}
```

使用JSON数据跑接口测试发现**无法对`list<Bean>`对象进行校验！**，那么这个问题就比较蛋疼了，因为集合的对象校验还是用的非常多的，下面针对这个“坑”讨论一下产生的原因和解决方式。

## 为什么无法校验`List<Object>`?

查阅了很多资料之后，我找到了stackflow一篇文章的解释，文章原文如下：

> StackFlow文章地址：https://stackoverflow.com/questions/17207766/spring-mvc-valid-on-list-of-beans-in-rest-service

```
Section 3.1.3 of the JSR-303 Specification says that:

In addition to supporting instance validation, validation of graphs of object is also supported. The result of a graph validation is returned as a unified set of constraint violations. Consider the situation where bean X contains a field of type Y. By annotating field Y with the @Valid annotation, the Validator will validate Y (and its properties) when X is validated. The exact type Z of the value contained in the field declared of type Y (subclass, implementation) is determined at runtime. The constraint definitions of Z are used. This ensures proper polymorphic behavior for associations marked @Valid.

Collection-valued, array-valued and generally Iterable fields and properties may also be decorated with the @Valid annotation. This causes the contents of the iterator to be validated. Any object implementing java.lang.Iterable is supported.
```

下面是英文的机翻：

```
JSR-303规范的3.1.3节说:
除了支持实例验证外，还支持对象图形的验证。
图形验证的结果作为约束违反的统一集合返回。
考虑bean X包含一个类型为Y的字段的情况，通过使用@Valid注释字段Y，验证器将在验证X时验证Y(及其属性)。
类型Y(子类，实现)声明的字段中包含的值的确切类型Z是在运行时确定的。
使用Z的约束定义。
这确保标记为@Valid的关联具有正确的多态行为。

集合值、数组值以及通常可迭代的字段和属性也可以用@Valid注释进行装饰。
这将导致验证迭代器的内容。
任何实现java.lang的对象。
支持Iterable。
```

`@Valid`是JSR-303批注，JSR-303适用于JavaBeans的验证。A`java.util.List`不是JavaBean（根据JavaBean的[官方描述](https://docs.oracle.com/javaee/5/tutorial/doc/bnair.html)），因此不能使用兼容JSR-303的验证器直接对其进行验证。这有两个观察结果支持。



## 简单粗暴的方式：

最为简单粗暴的方式是既然不能自动校验，那我们换成手动好了，这种方式的优缺点如下：

优点：

1. 校验的细节由自己决定，可以附加业务的校验，也可以自由灵活的组合
2. 可以编写健壮的工具类代码，甚至脱离spring validator（指hibernate validator）

缺点：

1. 代码复用性差，这个和编程水平有关，工具类也分写的好和写的差
2. 因为需要思考细节，容易出逻辑漏洞和其他BUG，所谓做的越多越容易出错就是这个道理
3. 需要学习更多的api使用，增加学习成本



```java
	/**
     * 校验集合bean内容是否符合校验规则
     * 
     返回样例：
         {
            "status": 401,
            "msg": "第 1 条信息：手机号必须为11位,币种必须是3位大写字母|第 2 条信息：币种必须是3位大写字母,手机号必须为11位",
            "data": null
         }
     * @param apiObj 接口传输对象
     * @return
     */
    public ZmtResult validListBean(List<ApiPaymentMsgDto> apiObj) {
        if (CollectionUtils.isEmpty(apiObj)) {
            return ZmtResult.build(401, "请求参数为空");
        }
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < apiObj.size(); i++) {
            ApiPaymentMsgDto apiPaymentMsgDto = apiObj.get(i);
            if (null == apiPaymentMsgDto) {
                return ZmtResult.build(400, "第 %s 条信息请求参数为空", i);
            }
            Validator validator = ValidatorUtil.getValidator();
            Set<ConstraintViolation<ApiPaymentMsgDto>> validate = validator.validate(apiPaymentMsgDto);
            // 如果不存在校验异常，则返回空信息
            if (org.apache.commons.collections.CollectionUtils.isEmpty(validate)) {
                continue;
            }
            builder.append(String.format("第 %s 条信息：", i + 1));
            for (ConstraintViolation<ApiPaymentMsgDto> apiPaymentMsgDtoConstraintViolation : validate) {
                builder.append(apiPaymentMsgDtoConstraintViolation.getMessage());
                builder.append(",");
            }
            builder.deleteCharAt(builder.length() - 1);
            builder.append("|");
        }
        // 如果存在错误信息，返回错误提示，否则返回空对象证明没有异常
        if (builder.length() > 0) {
            builder.deleteCharAt(builder.length() - 1);
            return ZmtResult.build(401, builder.toString());
        } else {
            return null;
        }

    }
```



## 一种优雅的设计解决方案：

看下stackFlow的一位老哥的解决办法：

原文链接：https://stackoverflow.com/questions/28150405/validation-of-a-list-of-objects-in-spring/36313615#36313615 （需要翻墙）

下面是机翻的版本：

> 我发现了另一种有效的方法。基本的问题是您想要一个列表作为服务的输入有效负载，但是javax验证不会验证列表，只验证JavaBean。诀窍是使用一个自定义的list类，它既是list又是JavaBean:

根据大佬的说明，我尝试实现了一个针对校验使用的list，注意需要提供get/set方法，以及使用泛型，在连接里面进行了模板代码和实现，可以直接拿去复制：

```java
/**
 * 为了兼容注解校验使用的一种设计
 * <p>{@link #{https://stackoverflow.com/questions/28150405/validation-of-a-list-of-objects-in-spring/36313615#36313615}</p>
 *
 * @author zhaoxudong
 * @version 1.0
 * @date 2021/1/3 23:39
 */
public class ValidatorList<E> implements List<E> {
    
    @Valid
    private List<E> list;

    /**
     * 关键
     * @return
     */
    public List<E> getList() {
        return list;
    }

    /**
     * 关键
     * @return
     */
    public void setList(List<E> list) {
        this.list = list;
    }

    public ValidatorList() {
        this.list = new ArrayList<E>();
    }

    public ValidatorList(List<E> list) {
        this.list = list;
    }
}
```

上面这个设计由于代码内容过长这里贴链接了：

https://gitee.com/lazyTimes/interview/blob/master/src/main/java/com/zxd/interview/valid/utils/ValidatorList.java

先不管其他的问题，先验证一下是否可以正常使用，而实际的体验：

```java
 /**
     * 测试stackflow 的一种优雅设计，可以实现对应的list 集合bean对象校验
     *
     * @param products      校验对象
     * @param bindingResult 异常绑定器
     * @return
     * @throws ParamException
     */
@RequestMapping("/test/testvalidList")
@ResponseBody
public Object testvalidList(@RequestBody @Validated ValidatorList<Product> products, BindingResult bindingResult) throws ParamException {
    if (bindingResult.hasErrors()) {
        List<FieldError> fieldErrors = bindingResult.getFieldErrors();
        fieldErrors.forEach(item -> {
            String defaultMessage = item.getDefaultMessage();
            System.err.println(defaultMessage);
        });
    }
    System.err.println(products);
    return null;
}
```

以上就是个人经过了研究的结果，不得不感叹思路真心很不错，目前个人使用正常，如果有问题欢迎下方留言讨论



# 其他扩展

## JSR - 303：

Hibernate Validator 是 Bean Validation 的参考实现，说白了`Hibernate Validator`就是`JSR-303`。

下载 JSR 303 – Bean Validation 规范 http://jcp.org/en/jsr/detail?id=303



## Hibernate - validator：

如果想要深入了解源代码实现，有必要研究一下`Hibernate - validator`的文档，从官方文档学习是一个推荐的方法：

旧版本的文档：https://docs.jboss.org/hibernate/validator/4.2/reference/en-US/html_single/

较新版本的文档：https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/?v=7.0#_validating_constraints



# 总结：

在学习过程中尝试看源代码，但是后来发现个人源代码学习经验不足，胡乱的看代码是很那坚持的，看了几个小时之后突然醒悟了，还是要从官方给的案例和文档的中，从易到难，而不是一上来就直接看源码，既缺少了大局，而且收效也低。

另外有条件尽可能上`stackflow`，里面大神真的多，可以学到很多很棒棒的设计。

文章字数比较多，感谢观看，如果觉得本文差强人意，可以查看下面的内容，里面的最后两篇文章有关于一部分原理对接讲解可以看一看，个人了解不够深入就不写出来误人子弟了。



# 巨人的肩膀：

Validation in Spring Boot：https://www.baeldung.com/spring-boot-bean-validation

Spring Validation最佳实践及其实现原理，参数校验没那么简单！：https://segmentfault.com/a/1190000023471742

spring官方那个案例：https://spring.io/guides/gs/validating-form-input/

Java Bean Validation（参数校验） 最佳实践： https://www.cnblogs.com/softidea/p/9712571.html

这么写参数校验(validator)就不会被劝退了~：https://juejin.cn/post/6844903902811275278#heading-0

springMVC Validation 参数检验工具：https://www.jianshu.com/p/999c6c10a5c6

Bean Validation: Integrating JSR-303 with Spring：http://web.archive.org/web/20120508030459/http://blog.orange11.nl/2009/08/04/bean-validation-integrating-jsr-303-with-spring/

Validation and Exception Handling with Spring：https://medium.com/sprang/validation-and-exception-handling-with-spring-ba44b3ee0723

@ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常：https://blog.csdn.net/kinginblue/article/details/70186586

**【Spring源码分析】40-Spring Validation参数校验的使用与原理**：https://blog.csdn.net/shenchaohao12321/article/details/100163991

**SpringBoot + Validator 参数校验配置 - - - [深度]**：https://www.jianshu.com/p/2432d0f51c0e