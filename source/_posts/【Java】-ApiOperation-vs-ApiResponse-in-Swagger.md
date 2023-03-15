---
title: 【Java】@ApiOperation vs @ApiResponse in Swagger
subtitle: java注解使用
author: lazytime
url_suffix: apioperate
tags:
  - Java
categories:
  - Java
keywords: Java的注解使用
description: Java的注解使用
copyright: true
date: 2023-03-15 15:00:09
---

# 原文

[https://www.baeldung.com/swagger-apioperation-vs-apiresponse](https://www.baeldung.com/swagger-apioperation-vs-apiresponse)

# 引言

本文内容讨论的是 _@ApiOperation_ 和 _@ApiResponse_ 注解的优劣。

# 介绍Swagger

一个RestFul最重要的是具备“自描述能力”，所谓的自描述能力是能在返回结果的同时，告知客户端调用下
一步的行为，Swagger在一定程度上封装和规范了这些操作。

什么是Swagger？

> Swagger represents a set of open-source tools built around OpenAPI Specification. It can help us design, build, document, and consume REST APIs.

Swagger代表了一套围绕OpenAPI规范建立的开源工具。它可以帮助我们设计、构建、记录REST APIs接口。其中最为常用的注解便是 @ApiOperation 和 @ApiResponse。

```java
@RestController
@RequestMapping("/customers")
class CustomerController {

   private final CustomerService customerService;

   public CustomerController(CustomerService customerService) {
       this.customerService = customerService;
   }
  
   @GetMapping("/{id}")
   public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
       return ResponseEntity.ok(customerService.getById(id));
   }
}
```

下面分别介绍这两个注解的使用，最后是进行对比。

<!-- more -->

# @ApiOperation

关键点：描述单独的一个操作和行为。通常用于单一的路径或者http请求方法。

```java
@ApiOperation(value = "Gets customer by ID", 
        response = CustomerResponse.class, 
        notes = "Customer must exist")
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

下面介绍@ApiOperation的一些属性：

## value

value属性主要是描述接口的行为，注意参数大小不能超过120个字符。从长度限制来看要尽可能的简化注解的描述。

```java
@ApiOperation(value = "Gets customer by ID")
```

## notes

个人认为把这个注解当成`comment`更为合适，notes和value的区别是可以填写的长度限制为text，意思是更为详细的叙述接口功能。

```java
@ApiOperation(value = "Gets customer by ID", notes = "Customer must exist")
```


## response

@ApiOperation注解中定义的响应属性应该包含一般的响应类型。大部分情况下我们会把返回结果用统一的对象包装：

```java
class CustomerResponse {
  
   private Long id;
   private String firstName;
   private String lastName;
  
   // getters and setters
}
```

> 更为常见的是使用类似`AjaxDto`这样的对象封装响应Code，响应Msg和响应数据Data。

在使用的过程中设置Class类，在Swagger文档中将会对应生成相关的对象以及

```java
@ApiOperation(value = "Gets customer by ID",
        response = CustomerResponse.class,
        notes = "Customer must exist")
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

## code

也就是响应code码，对应 [Http Status](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

# @ApiResponse

@ApiResponse 注解主要用于描述接口的返回状态码以及对应的返回信息。虽然@ApiOperation注解描述了操作和一般的返回类型，但@ApiResponse注解描述了其余可能的状态码。

@ApiResponse注解的特点是**方法注解优先于类注解**。我们应该在@ApiResponses注解中使用@ApiResponse注解，无论我们有一个还是多个响应。

下面是正确的@ApiResponse 用法案例：

```java
@ApiResponses(value = {  
        @ApiResponse(code = 400, message = "Invalid ID supplied"),  
        @ApiResponse(code = 404, message = "Customer not found")})  
@GetMapping("/{id}")  
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {  
    return ResponseEntity.ok(customerService.getById(id));  
}
```

对于特殊的状态码进行描述：

```java
@ApiOperation(value = "Gets customer by ID", notes = "Customer must exist")  
@ApiResponses(value = {  
        @ApiResponse(code = 200, message = "OK", response = CustomerResponse.class),  
        @ApiResponse(code = 400, message = "Invalid ID supplied"),  
        @ApiResponse(code = 404, message = "Customer not found"),  
        @ApiResponse(code = 500, message = "Internal server error", response = ErrorResponse.class)})  
@GetMapping("/{id}")  
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {  
    return ResponseEntity.ok(customerService.getById(id));  
}
```

## code 和 message

与@ApiOperation注解中的code属性一样，它应该包含响应的HTTP状态代码。值得一提的是，我们不能用相同的代码属性定义一个以上的@ApiResponse。

```java
@ApiResponse(code = 400, message = "Invalid ID supplied")
```

## response

response可以为特殊的状态码指定对象：

```java
class ErrorResponse {

    private String error;
    private String message;

    // getters and setters
}
```

其次，让我们为内部服务器错误添加一个新的@ApiResponse。

```java
@ApiResponses(value = {
        @ApiResponse(code = 400, message = "Invalid ID supplied"),
        @ApiResponse(code = 404, message = "Customer not found"),
        @ApiResponse(code = 500, message = "Internal server error", response = ErrorResponse.class)})
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

#  @ApiOperation 和 @ApiResponse 区别

|              @ApiOperation              |                       @ApiResponse                        |
| :-------------------------------------: | :-------------------------------------------------------: |
|    通用用于描述一个操作     | 用于描述操作的可能的响应 |
|     通常用于成功的请求     |          可以用于成功或者失败的各种请求          |
| 只能用在方法级别 |        可以用于类或者方法级别        |
|          可以直接使用           |   只能在@ApiResponses注解中使用。   |
| 默认code为200  |       没有默认值        |

