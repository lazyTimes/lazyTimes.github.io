---
title: 使用Hibernate JSP 303 整合到SSM项目
subtitle: 这个人很懒，不想写副标题
author: lazytime
tags:
  - 无
categories:
  - 未分类
keywords: 请输入关键字（英文逗号分隔多个关键字）
description: 请输入描述信息
copyright: true
date: 2020-07-26 17:46:26
---

# 需求：

1. 在项目中加入注解校验
2. 可以控制错误的操作
3. 对于参数错误进行全局日志记录，方便后续查看

<!-- more -->

# 使用Hibernate JSP 303 整合到项目

https://juejin.im/post/5d3fbeb46fb9a06b317b3c48

https://zhuanlan.zhihu.com/p/49589845

## ValidataUtils

此工具类配置可以大大减少配置https://github.com/hjzgg/usually_util/blob/master/spring-validate-demo/validator/ValidatorUtils.java

```
@Bean
public Validator validator() {
    return new LocalValidatorFactoryBean();
}
@Component
public class ValidatorUtils implements ApplicationContextAware {

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ValidatorUtils.validator = (Validator) applicationContext.getBean("validator");
    }

    private static Validator validator;

    public static Optional<String> validateResultProcess(Object obj)  {
        Set<ConstraintViolation<Object>> results = validator.validate(obj);
        if (CollectionUtils.isEmpty(results)) {
            return Optional.empty();
        }
        StringBuilder sb = new StringBuilder();

        for (Iterator<ConstraintViolation<Object>> iterator = results.iterator(); iterator.hasNext(); ) {
            sb.append(iterator.next().getMessage());
            if (iterator.hasNext()) {
                sb.append(" ,");
            }
        }
        return Optional.of(sb.toString());
    }
}
```

## 自定义校验器

https://www.jb51.net/article/174064.htm

## 自定义校验规则：

https://songwell1024.github.io/2018/08/02/JSR303/

## 解决思路：

### 第一种：使用`@Valid` 加上 `BindResult`

注意事项： `@valid` 加入的DTO后面必须跟上 `BindResult`

```
@valid` 加入的DTO后面必须跟上 `BindResult``@valid` 加入的DTO后面必须跟上 `BindResult
```

，如果不按此规则，将会由Spring的 `bindException` 直接抛出异常

### 第二种：使用切面的方式统一处理异常

```
@Aspect
@Component
public class ControllerValidatorInterceptor {
    
    /**
    * 设置 around 环绕通知切面
    */
    @Around("execution(* com.aerexu.web.*.*(..)) && args(..,bindingResult)")
    public Object doAround(ProceedingJoinPoint pjp, BindingResult bindingResult) throws Throwable {
        Object retVal;
        if (bindingResult.hasErrors()) {
            retVal = doErrorHandle();
        } else {
            retVal = pjp.proceed();
        }
        return retVal;
    }
}
```

## 实用bindResult 到最佳实践

https://vimsky.com/zh-tw/examples/detail/java-class-org.springframework.validation.BindingResult.html

真的写的十分好，这里分享出来

## 常用注解

```
@Null  被注释的元素必须为null
@NotNull  被注释的元素不能为null
@AssertTrue  被注释的元素必须为true
@AssertFalse  被注释的元素必须为false
@Min(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@Max(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Size(max,min)  被注释的元素的大小必须在指定的范围内。
@Digits(integer,fraction)  被注释的元素必须是一个数字，其值必须在可接受的范围内
@Past  被注释的元素必须是一个过去的日期
@Future  被注释的元素必须是一个将来的日期
@Pattern(value) 被注释的元素必须符合指定的正则表达式。
@Email 被注释的元素必须是电子邮件地址
@Length 被注释的字符串的大小必须在指定的范围内
@NotEmpty  被注释的字符串必须非空
@Range  被注释的元素必须在合适的范围内
```