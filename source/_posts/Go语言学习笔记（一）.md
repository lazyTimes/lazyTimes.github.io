---
title: Go语言学习笔记（一）
subtitle: Go语言学习笔记
author: lazytime
tags:
  - go
categories:
  - golang
keywords: golang
description: go语言学习
copyright: true
date: 2020-07-26 11:33:37
---

# Go语言学习笔记（一）

## Go语言是什么？

*Go 编程语言是一个使得程序员更加有效率的开源项目。Go 是有表达力、简洁、清晰和有效率的。它的并行机制使其很容易编写多核和网络应用,而新奇的类型系统允许构建有弹性的模块化程序。Go 编译到机器码非常快速,同时具有便利的垃圾回收和强大的运行时反射。它是快速的、静态类型编译语言,但是感觉上是动态类型的,解释型语言。*

<!-- more -->

## GO能做什么？

https://www.zhihu.com/question/57404512 知乎回答

## 为什么要学习Go?

客观原因：

1. GO 可以非常高效的完成一些JAVA需要很多功夫完成的操作

例如：

- 管理日志
- IO监控
- 生成模板代码

1. 近两年GO诞生的框架越来越多，他的便捷性得到认可
2. 技多不压身

主管原因：

1. 想用GO完成一些工作上重复干的活儿
2. 编写一些工具帮助日常开发

## 如何学习？

https://www.zhihu.com/question/23486344 知乎第二个

https://tour.go-zh.org/list 快速入门（需要翻墙）

https://mikespook.com/learning-go/ 《学习GO语言》中文版，不过出版比较早，最好自我实践

https://gobyexample.com/ 一些基础的语代码

# 快速入门

## 安装GO

进入官网：https://golang.org/

（因为不可抗力的原因，这个房子国内没法访问。。。也可能是我联通网的问题）

window下载地址（注意32位和64位的区别）：

https://dl.google.com/go/go1.13.8.windows-amd64.msi

https://dl.google.com/go/go1.13.8.windows-386.msi

Linux下载地址（注意32位和64位的区别）：

https://dl.google.com/go/go1.13.8.src.tar.gz

https://dl.google.com/go/go1.13.8.darwin-amd64.tar.gz

## Go语言的特点

> 并行 Go 让函数很容易成为 非常 轻量的线程。这些线程在 Go 中被叫做 goroutines

> Channel 这些 goroutines 之间的通讯由 channel[18, 25] 完成；

> 快速 编译很快，执行也很快。目标是跟 C 一样快。编译时间用秒计算；

> 安全 当转换一个类型到另一个类型的时候需要显式的转换并遵循严格的规则。Go 有垃圾收集，在 Go 中无须 free() ，语言会处理这一切；

> 标准格式化 Go 程序可以被格式化为程序员希望的（几乎）任何形式，但是官方格式是存在的。标准也非常简单： gofmt 的输出就是 官方认可的格式 ；

> 类型后置 类型在变量名的 后面 ，像这样 var a int ，来代替 C 中的 int a ；UTF-8 任何地方都是 UTF-8 的，包括字符串 以及 程序代码。你可以在代码中使用 Φ = Φ + 1 ；

> 开源 Go 的许可证是完全开源的，参阅 Go 发布的源码中的 LICENSE 文件；

> 开心 用 Go 写程序会非常开心！

## IDEA 配置GO语言支持

1. 打开`Setting`
2. 选择`Setting`，输入`GO`
3. 安装`GO`语言支持

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216135342.png?ynotemdtimestamp=1595729096959)

1. 同时安装GO的Template的插件
2. 重启IDEA

## IDEA 创建GO项目

完成上一步的操作之后，我们可以实现IDEA创建GO项目

1. 打开IDEA，左上角`File`，New->选择GO工程

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216145822.png?ynotemdtimestamp=1595729096959)

1. 自定义一个项目名称

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216145942.png?ynotemdtimestamp=1595729096959)

1. 创建一个HelloWorld包
2. 写入如下代码

```
package main

import "fmt"

func main() {
	fmt.Print("hello");
}
```

1. 写好之后，便可以直接右上角运行

以上便是一个最简单的GO语言Helloworld代码，感觉比较懵逼，工具做了哪些事情？

### 使用原始的方式构建一个helloword

使用原始的GO语言方式需要满足前提条件：**环境变量是否设置**

1. 在指定的目录比如`D:\java\GO\go-study\helloword`下面创建一个`hello.go`的文件
2. 使用`NotePad++`编写脚本
3. 和上面的代码一样，照抄或者复制粘贴，这里直接复制了上面的代码
4. 打开`cmd`，进入对应的目录`cd D:\java\GO\go-study\helloword`，**注意切换一下盘符**
5. 执行命令`go run hello.go`，即可看到结果

> 早期的GO语言需要类似`javac`的编译动作，但是现在没有这个必要了
>
> 其他情况：
>
> - Go 被安装在 ˜/go ，而 $GOROOT 被设置为 GOROOT=˜/go ；
> - 希望编译的 Go 代码放在 ˜**/g/src** 而 $GOPATH 设置为 **GOPATH=˜/g** 。在使用包的 时候需要用到这个变量（参阅第 3 章）。

## 注意事项:

1. `package main`必须首先出现，紧跟着是`import`
2. 首行这个是必须的。所有的 Go 文件以 `package <something>`开头，对于独立运行的执行文件必须是 `package main`；

# 了解GO语言

## 变量定义

- 需要先声明类型，注意和JAVA完全不一样：例如`var a int`
- 紧接着`:=`方式设置变量，l例如：`a := 15`

> Golang 在变量定义的时候设置了默认值，和JAVA的区别在于使用默认赋值
>
> 同时要注意一下：如果在函数体内部定义变量不使用GO会产生一个报错

注意：

- `dd:=1`这种形式只能在函数体内部
- `ar ss = "sssdd"` 会根据赋值猜测变量类型

### 定义多个变量

```
var (
	x int
	b bool
)
```

### 平行声明和赋值

```
// 平行声明
var a,b int 
// 平行赋值
a,b := 34, 35
```

### 明确变量长度

如果你希望明确其长度，可以使用如下的定义

```
var df int32 = 55;
```

注意：默认的通用int会根据硬件去判断32位还是64位

其他定义举例：

```
int8 ， int16 ， int32 ， int64 和 byte ， uint8 ， uint16 ， uint32 ，uint64 
```

**其中byte 是 uint8 的别名**

### 可以混合使用类型吗？

来看下面的定义

```
1 package main
2
3 func main() {
4 	var a int ← 通用整数类型
5 	var b int32 ← 32 位整数类型
6 	a = 15
7 	b = a + a ← 混合这些类型是非法的
8 	b = b + 5 ← 5 是一个（未定义类型的）常量，所以这没?问题
9 }
```

`types.go:7: cannot use a + a (type int) as type int32 in assignment` 赋值非法的时候运行会报错，编译器会提示语法错误

## 常量定义

```
const (
    a = iota
    b = iota
)
```

第一个 iota 表示为 0，因此 a 等于 0，当 iota 再次在新的一行使用时，它的值增加了 1，因此 b 的值是 1

省略重复的`iota`

```
const (
    a = iota
    b
)
```

指定确切的类型

```
const (
    a = 0 ← Is an int now
    b string = "0"
)
```

### 字符串

`：`在 Go 中字符串是不可变的，字符串的修改需要如下方法

```
s := "hello" 
c := [] rune (s) // 转换 s 为 rune 数组
c[0] = 'c'// 修改数组的第一个元素
s2 := string (c) //创建 新的 字符串 s2 保存修改
fmt.Printf("%s\n", s2) //用 fmt.Printf 函数输出字符串
```

多行文本的写法

```
s:= "sdsdadsa "+ 
"asdsad" + 
"sddsd"
```

注意`"+"`号必须在末尾

### 复数

Go 原生支持复数，写法如下：

```
var c complex64 = 5+5i ; 
fmt.Printf("Value is: %v", c)
```

### 错误

在GO里面，错误也被当做类型处理，声明的方式如下:

```
var e error 
```

默认为`null`， 本质上是一个接口

## 运算符和内建函数

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216193505.png?ynotemdtimestamp=1595729096959)

> Go **不支持**运算符重载

## 关键字

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216193610.png?ynotemdtimestamp=1595729096959)

先做个大概了解，详细使用的时候在记录笔记

## 控制语句

### if-else

```
if (true && true) 
{
    fmt.Print("hello")
    return
}
```

注意格式，这种写法是不被允许的，需要将第一个左括号放到`if`的同一行

### goto

写法如下：

```
func myfunc() {
	i := 0
Here: ← 这行的第一个词，以分号结束作为标签
    println(i)
    i++
    goto Here ← 跳转
}
```

标签**大小写敏感**

### for

写法分成三种

第一种：一般的for循环

```
var i = 5
Tag:
    if i == 0 {
        return
    }
for i := 0; i<10 ; i++ {
    fmt.Println(i);
}
i--
goto Tag
```

第二种：相当于while

```
var z = 6
for z > 5 {
    z++
    if z > 7 {
        break
    }
    print("hello wolrd")
}
```

第三种：表示死循环

```
for {
    .....
}
```

注意下面的写法是固定的，不能使用逗号表达式，只能使用平行赋值的方式

```
for i, j := 0, len (a)-1 ; i < j ; i, j = i+1, j-1 {
	a[i], a[j] = a[j], a[i] ← 平行赋值
}
```

> 注意GO 里面是没有while语句的
>
> 只能使用for进行书写while的循环

### range

```
list := []string{"aaa", "vvv", "ccc", "ddd", "eeee"}
	for k,v := range list {
		print(k, "-", v)
	}
```

上面的例子是如下运转的：

1. k 表示为 下标， v为值， range 代表了迭代器
2. 迭代元素从 0 到 4 ，元素从 aaa...eeee 迭代

也可以在字符串上直接使用 range ，看下面的案例

```

```

## switch

switch非常的灵活，可以在case里面写上表达式

```
func unhex(c byte ) byte {
    switch {
    case '0' <= c && c <= '9':
    return c - '0'
    case 'a' <= c && c <= 'f':
    return c - 'a' + 10
    case 'A' <= c && c <= 'F':
    return c - 'A' + 10
    }
    return 0
}
```

它不会匹配失败后自动向下尝试，但是可以使用 fallthrough 使其这样做

```
switch i {
case 0: fallthrough
case 1:
f() // 当 i == 0 时， f 会被调用！
}
```

还可以使用下面这张写法，逗号分隔多个条件

```
case ' ', '?', '&', '=', '#', '+': ← , as ”or”
return true
}
```

## 内建函数

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200216204755.png?ynotemdtimestamp=1595729096959)

### array、slices 和 map

#### array

```
var arr = [3]int{}
for item := range arr{
    print(item);
}
ss := [...] int{1,2,2,3,4,4}
// 初始化
a := [3] int{ 1, 2, 3}
ff := [3][2] int { { 1,2 } , { 3,4 } , { 5,6 } }
bb := [3][2] int { [2] int { 1,2 } , [2] int { 3,4 } , [2] int { 5,6 } }
print(a, bb, ss);
```

#### slices

```
slice`总是指向底层的一个`array
```

总体来说go的语法基本和java相符，但是部分内容需要小心对待，个人不是很爽go语言的部分奇葩语法

slice 总是与一个固定长度的 array 成对出现。其影响 slice 的容量和长度

slice： slice := array[0:n]

对比图；

![img](https://raw.githubusercontent.com/lazyTimes/imageRepository/master/img/20200217221715.png?ynotemdtimestamp=1595729096959)

##### 扩展 slice

```
append` 和 `copy
```

函数`append` 向`slice s`追加零值或其他 `x`值，并且返回追加后的新的、与 s 有相同类型的`slice`

```
s0 := [] int { 0, 0 }  
s1 := append (s0, 2) //追加一个元素， s1 == []int{0, 0, 2} ；
s2 := append (s1, 3, 5, 7) //追加多个元素， s2 == []int{0, 0, 2, 3, 5, 7} ；
s3 := append (s2, s0...) // 追加一个 slice， s3 == []int{0, 0, 2, 3, 5, 7, 0, 0} 。注意这三个点！
```

函数 copy 从源 slice src 复制元素到目标 dst ，并且返回复制的元素的个数。

```
var a = [...] int { 0, 1, 2, 3, 4, 5, 6, 7 }
var s = make ([] int , 6)
n1 := copy (s, a[0:]) ← n1 == 6, s == []int{0, 1, 2, 3, 4, 5}
n2 := copy (s, s[2:]) ← n2 == 4, s == []int{2, 3, 4, 5, 4, 5}
```

#### map

案例：

```
monthdays := map [ string ] int {
"Jan": 31, "Feb": 28, "Mar": 31,
"Apr": 30, "May": 31, "Jun": 30,
"Jul": 31, "Aug": 31, "Sep": 30,
"Oct": 31, "Nov": 30, "Dec": 31, ← 逗号是必须的
}
```

只做声明的时候：`monthdays := make ( map[ string ] int )`

如何使用：`monthdays["Dec"]`

```
for _, days := range monthdays { ← 键没有使用，因此用 _, days
year += days
}
```

检查元素是否存在

```
var value int
var present bool
value, present = monthdays["Jan"] ← 如果存在， present 则有值 true
← 或者更接近 Go 的方式
v, ok := monthdays["Jan"] ← “逗号 ok ”形式
```

# 练习

## Q1. (0) For-loop

1. 创建一个基于 for 的简单的循环。使其循环 10 次，并且使用 fmt 包打印出计数 器的值。
2. 用 goto 改写 1 的循环。关键字 for 不可使用。
3. 再次改写这个循环，使其遍历一个 array，并将这个 array 打印到屏幕上。

# 函数

```
0-func 1-(p mytype) 2-funcname(3- q int ) (r,s int ) { return 0,0 }
```

. . 0 关键字 func 用于定义一个函数； . . 1 函数可以绑定到特定的类型上。这叫做 接收者 。有接收者的函数被称作 method。 第 5 章将对其进行说明； . . 2 funcname 是你函数的名字； . . 3 int 类型的变量 q 作为输入参数。参数用 pass-by-value 方式传递，意味着它们会 被复制； . . 4 变量 r 和 s 是这个函数的 命名返回值。在 Go 的函数中可以返回多个值。参阅 第 28 页的 “多值返回”。如果不想对返回的参数命名，只需要提供类型： ( int , int ) 。 如果只有一个返回值，可以省略圆括号。如果函数是一个子过程，并且没有任何 返回值，也可以省略这些内容； . . 5 这是函数体。注意 return 是一个语句，所以包裹参数的括号是可选的。