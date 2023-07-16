---
title: 【PHP】英文博客专栏PHP快速入门个人笔记
subtitle: 学英文顺带了解PHP技术
url_suffix: php_study1
author: lazytime
tags:
  - PHP
categories:
  - PHP
keywords: php,英文博客
description: php英文内容了解
copyright: true
date: 2023-01-24 11:41:12
---

# 英文博客专栏PHP快速入门

# 引言

	本文是对于英文原始博客的一个PHP入门专栏的个人笔记摘录，因为非常入门并且自身有JAVA语言基础，看的比较快并且会忽略很多共同点，建议读者有能力可以看看博客的原文顺带提升英文能力，作者文字表达能力非常强，写的非常棒。
	
	这篇专栏介绍了PHP8入门，专栏写于2022年中旬，不管是单词还是语法句式都十分通俗易懂，**学技术的同时提升英语水平并且有助于提升自信心**。

# 目录

-   [介绍](#介绍)
-   [过往历史](#过往历史)
-   [php是一个怎么样的语言](#php是一个怎么样的语言)
-   [设置PHP](#设置PHP)
-   [第一个PHP程序](#第一个PHP程序)
-   [基本类型](#基本类型)
-   [操作符](#操作符)
    -   [字符串操作](#字符串操作)
-   [编写注释](#编写注释)
-   [和数字有关的内置函数](#和数字有关的内置函数)
-   [Array数组](#Array数组)
    -   [arrays常用函数](#arrays常用函数)
-   [关联数组](#关联数组)
-   [条件语句](#条件语句)
-   [循环](#循环)
-   [函数](#函数)
    -   [匿名函数](#匿名函数)
    -   [值传递和地址传递](#值传递和地址传递)
    -   [箭头函数](#箭头函数)
    -   [使用map,reduce,filter函数循环数组处理](#使用mapreducefilter函数循环数组处理)
-   [面向对象](#面向对象)
    -   [面向对象讨论](#面向对象讨论)
    -   [如何构建对象](#如何构建对象)
    -   [属性和方法](#属性和方法)
    -   [继承](#继承)
    -   [重写](#重写)
    -   [静态](#静态)
    -   [对象比较](#对象比较)
    -   [对象遍历](#对象遍历)
    -   [对象克隆](#对象克隆)
    -   [魔术方法](#魔术方法)
-   [文件包含](#文件包含)
-   [文档系统的有用常量、函数和变量](#文档系统的有用常量函数和变量)
-   [错误](#错误)
-   [异常](#异常)
-   [日期](#日期)
-   [常量和枚举](#常量和枚举)
-   [PHP web平台部署](#PHP-web平台部署)
    -   [处理HTTP请求](#处理HTTP请求)
    -   [\$\_SEVER 对象](#_SEVER-对象)
    -   [使用cookies](#使用cookies)
    -   [Sessions](#Sessions)
    -   [IO](#IO)
    -   [database](#database)
    -   [JSON](#JSON)
    -   [email](#email)
-   [Composer](#Composer)
-   [部署PHP应用](#部署PHP应用)

<!-- more -->

# 原始博客地址

[https://thevalleyofcode.com/php/](https://thevalleyofcode.com/php/ "https://thevalleyofcode.com/php/")

# 介绍

PHP是一个两级分化的语言，觉得它好的人称赞它简单，PHP的语法比较自由上手非常简单。而不好的人则会像我一样认为是个**四不像**语言，既有前端脚本的影子，但是同时支持面向对象的方式组合代码，总是会有种奇怪的感觉。

但是不得不承认，世界上绝大多数WEB网站都是PHP构建的，PHP是web领域当之无愧的佼佼者。虽然这语言现在在国内是一潭死水，但是在国外它是能排进前十的热门编程语言。

PHP在短短的几年内快速发展，从最开始几年的PHP4和PHP5的膨胀，到现在PHP8的版本发布，更新迭代的速度还是很快的。


# 过往历史

PHP起源于1994年的个人博客网站，作者是rasmus lerdorf，PHP在1997到2000随着互联网的快速崛起并且爆炸式增长。

用途：

-   和HTML存在一点点交互动态的HTML语言，以及web应用程序当中对外提供访问。
-   Facebook就是构建在PHP网站之上的，早期微微博也同样用的PHP语言
-   wiki百科同样使用PHP构建


## PHP是一个怎么样的语言

虽然PHP被戏称脚本语言，但是实际上它是解释型语言，和广大编译运行的服务端语言没什么区别。只不过和其他大部分解释型语言不同点是PHP**不需要编译**就可以运行，或者可以认为编译的动作本身就是**自动**的。这和Java，GO以及c语言等等都有很大不同。在JAVA领域PHP非常像JSP，但是实际对比会发现要比JSP更灵活和方便，也更好用。

> 这个语言内部可自动通过编译器把代码翻译成机器可以认识以及可以运行的语言。

从个人角度看PHP被称作脚本语言是比较合适的词，因为它在web领域如鱼得水。此外因为PHP是动态类型语言，开发者不需要关注变量类型，但是有时候又因为类型转化的问题出现一些难以察觉的错误。

动态类型语言是高级编程语言的趋势这一点毋庸置疑。就连JDK11也实现了 **var** 关键词的动态类型语法糖定义就可以看出端倪。

最后用作者的原文总结：PHP是一门很像JavaScript的语言，不同的是它有动态类型，灵活类型的解释型**后端**语言。



# 设置PHP

本部分作者介绍了mamp的安装使用，个人没有使用经验就不详细记录了，对于PHP作者推荐使用VScode 编辑器开发，个人使用下来发现确实好用，当然Jerbrian的PHP IDE也不错，对于常年使用IDEA的开发人员基本可以无缝衔接。

PHP 开发一般依赖**套件**，PHP本身就是起源于个人博客专职于WEB Application领域，所以他需要最为根本的软件比如**Apach，Mysql**，**Redis**等等。

PHP开源套件软件很多，这里就不过多展开了。当然套件开发不是强制的，当然开发者开发过程中也可以单独部署中间件和数据库。

对于php的web应用，必备组件无外乎下面几个：

-   PHP语言环境变量，推荐最新版的PHP8。
-   数据库，通常以MySQL为主。
-   apache或者nignix作为web服务器。

PHP是面向http web应用程序开发语言，很多时候都需要和HTML页面配合，这和古老的JSP语言有点类似，但是实际使用的时候更多是和模板引擎以及框架配合。

# 第一个PHP程序

PHP的Helloworld非常简单，只需要在mamp或者其他PHP程序的开发软件根目录创建`index.html`的文件即可。很多web server服务器基本都使用**index.html**作为默认的访问页面，所以如果直接访问localhost端口的webserver根路径，那么就会展示对应`index.html`页面。

PHP代码通常以及`<?php`开头以及`?>`结尾，中间编写有关PHP语言代码即可，我们可以在`index.html`文件全文替换成下面的代码。

```php
<?php
echo 'World';
?>
```

>  虽然访问的是html页面，但是里面的PHP代码却会被识别翻译并且执行。



# 基本类型

PHP是动态类型语言，定义变量方式如下：

```php
<?php
$a = 5;
$b = '444';

?>
```

PHP支持下面的基础类型：

-   `bool` boolean values (true/false)
-   `int` integer numbers (no decimals)
-   `float` floating-point numbers (decimals)
-   `string` strings
-   `array` arrays
-   `object` objects
-   `null` a value that means “no value assigned”

如果要知道变量的数据类型，可以使用`var_dump()`的方法检查：

```php
$age = 20;

var_dump($age);
```


# 操作符

PHP的基础操作符：

算数操作： `+`, `-`, `*`, `/` (division), `%` (remainder) and `**` (exponential).

赋值操作：`=`

比较操作：`<`, `>`, `<=`, `>=`，此外还有相等和全等操作，含义和JS的类似，相等可以类型不匹配比如 5=='5'，全等类型必须一致，比如5==='5'就是false。

- `==` returns true if the two operands are equal.

- `===` returns true if the two operands are identical.

和比较操作相反的有!==以及!=符号。

自增操作：++和 - - 操作。

特殊符号：think new lines `\n` or tabs `\t`

拼接操作：PHP和其他语言比较大的区别，那就是类似字符串拼接用的是 “.”

```php
$fullName = $firstName . ' ' . $lastName;
```



## 字符串操作

字符串的操作和其他后端语言类似，下面简单列举博客中的一些实验，这里直接上代码就不过多解释了：

```php
$name = 'Flavio';
strlen($name); //6

$name = 'Flavio';
substr($name, 3); //"vio" - start at position 3, get all the rest
substr($name, 2, 2); //"av" - start at position 2, get 2 items

$name = 'Flavio';
str_replace('avio', 'ower', $name); //"Flower"

$name = 'Flavio';
$itemObserved = str_replace('avio', 'ower', $name); //"Flower"

```

-   [trim()](https://www.php.net/manual/en/function.trim.php "trim()") strips white space at the beginning and end of a string
-   [strtoupper()](https://www.php.net/manual/en/function.strtoupper.php "strtoupper()") makes a string uppercase
-   [strtolower()](https://www.php.net/manual/en/function.strtolower.php "strtolower()") makes a string lowercase
-   [ucfirst()](https://www.php.net/manual/en/function.ucfirst.php "ucfirst()") makes the first character uppercase
-   [strpos()](https://www.php.net/manual/en/function.strpos.php "strpos()") finds the firsts occurrence of a substring in the string
-   [explode()](https://www.php.net/manual/en/function.explode.php "explode()") to split a string into an array
-   [implode()](https://www.php.net/manual/en/function.implode.php "implode()") to join array elements in a string



# 编写注释

编写注释的方法如下：

```php
// single comment
/*

this is a comment

*/

//or

/*
 *
 * this is a comment
 *
 */

//or to comment out a portion of code inside a line:

/* this is a comment */
```



# 和数字有关的内置函数

作者事先列举一些和数字或者数学计算有关函数：

-   [round()](https://www.php.net/manual/en/function.round.php "round()") to round a decimal number, up/down depending if the value is > 0.5 or smaller
-   [ceil()](https://www.php.net/manual/en/function.ceil.php "ceil()") to round a a decimal number up
-   [floor()](https://www.php.net/manual/en/function.floor.php "floor()") to round a decimal number down
-   [rand()](https://www.php.net/manual/en/function.rand.php "rand()") generates a random integer
-   [min()](https://www.php.net/manual/en/function.min.php "min()") finds the lowest number in the numbers passed as arguments
-   [max()](https://www.php.net/manual/en/function.max.php "max()") finds the highest number in the numbers passed as arguments
-   [is\_nan()](https://www.php.net/manual/en/function.is-nan.php "is_nan()") returns true if the number is not a number

# Array数组

数组定义可以用方括号或者array函数，数组可以当做其他编程语言的列表（容器）看待，不需要定义长度并且容量自动增长。

列表里面的元素类型可以不一致，甚至元素可以是另一个列表。

```php
// 数组定义
$list = [];

$list = array();

// 初始化定义
$list = [1, 2];

$list = array(1, 2);

$list = ['a', 'b'];
$list[0]; //'a' --the index starts at 0
$list[1]; //'b'

$list = [1, [2, 'test']];

```

添加元素可以使用空方括号的方式设置值，这时候参数会自动在末尾追加。

```php
$list = ['a', 'b'];
$list[] = 'c';

/*
$list == [
  "a",
  "b",
  "c",
]
*/
```

使用**array\_unshift** 添加元素到列表头部：

```php
$list = ['b', 'c'];
array_unshift($list, 'a');

/*
$list == [
  "a",
  "b",
  "c",
]
*/
```

使用count函数计算数组的元素数量：

```php
$list = ['a', 'b'];

count($list); //2
```

检查元素是否在数组，使用**in\_array** 函数\`：

```php
$list = ['a', 'b'];

in_array('b', $list); //true
```



## arrays常用函数

常用函数根据作者笔记记录即可。

-   `is_array()` to check if a variable is an array
-   `array_unique()` to remove duplicate values from an array
-   `array_search()` to search a value in the array and returns the key
-   `array_reverse()` to reverse an array
-   `array_reduce()` to reduce an array to a single value using a callback function
-   `array_map()` to apply a callback function to each item in the array. Typically used to create a new array by modifying the values of an existing array, without altering that.
-   `array_filter()` to filter an array to a single value using a callback function
-   `max()` to get the maximum value contained in the array
-   `min()` to get the minimum value contained in the array
-   `array_rand()` to get a random item from the array
-   `array_count_values()` to count all the values in the array
-   `implode()` to turn an array into a string
-   `array_pop()` to remove the last item of the array and return its value
-   `array_shift()` same as `array_pop()` but removes the first item instead of the last
-   `sort()` to sort an array
-   `rsort()` to sort an array in reversing order
-   `array_walk()` similarly to `array_map()` does something for every item in the array, but in addition it can change values in the existing array

&#x20;

# 关联数组

到目前为止，我们已经使用了带有增量数字索引的数组：0、1、2... 您还可以使用带有命名索引（键）的数组，我们称它们为关联数组：

```php
$list = ['first' => 'a', 'second' => 'b'];

$list['first'] //'a'
$list['second'] //'b'
```

可以通过关联数组进行标记key以及value，关联数组同样有比较多的操作方法：

-   `array_key_exists()` to check if a key exists in the array
-   `array_keys()` to get all the keys from the array
-   `array_values()` to get all the values from the array
-   `asort()` to sort an associative array by value
-   `arsort()` to sort an associative array in descending order by value
-   `ksort()` to sort an associative array by key
-   `krsort()` to sort an associative array in descending order by key

在此处查看所有有关联数组函数：[ https://www.php.net/manual/en/ref.array.php](https://www.php.net/manual/en/ref.array.php)


# 条件语句

条件语句的最基础用法：

```php
$age = 17;

if ($age > 18) {
  echo 'You can enter the pub';
} else {
  echo 'You cannot enter the pub';
}
```

这里用了cannot而不是can't是因为单引号嵌套会出现“截断”导致报错，需要单引号内部嵌套需要使用转义符`\`**反斜杠**。

`<`, `>`, `<=`, `>=`, `==`, `===` , `!=`, `!==` 这些符号在实际使用和条件语句一起使用：

这里需要注意PHP提供了专门的 elseif，而不能像其他语言一样使用 `else[空格]if` 的语法：

```php
$age = 17;

if ($age > 20) {
  echo 'You are 20+';
} elseif ($age > 18) {
  echo 'You are 18+';
} else {
  echo 'You are <18';
}
```

Swtich的语法和其他编程语言是一致的：

```php
$age = 17

switch($age) {
  case 15:
    echo 'You are 15';
    break;
  case 16:
    echo 'You are 16';
    break;
  case 17:
    echo 'You are 17';
    break;
  case 18:
    echo 'You are 18';
    break;
  default:
    echo "You are $age";
}
```

# 循环

PHP的循环语句语法有`while`, `do while`, `for`, and `foreach`，`while`和`do while`的方法和大部分编程语言没什么不同。

```php
$counter = 0;

while ($counter < 10) {
  echo $counter;
  $counter++;
}


$counter = 0;

do {
  echo $counter;
  $counter++;
} while ($counter < 10);




```

主要差别是`foreach`语法，可以用他遍历列表，也可以用来遍历列表获取索引，也就遍历关联数组的`key/value`值。

```php
$list = ['a', 'b', 'c'];

foreach ($list as $value) {
  echo $value;
}

$list = ['a', 'b', 'c'];

foreach ($list as $key => $value) {
  echo $key;
}

```

对于普通for循环，可以使用count函数计算数组长度的size。

```php
$list = ['a', 'b', 'c'];

for ($i = 0; $i < count($list); $i++) {
  echo $list[$i];
}

//result: abc
```

和循环搭配使用的break和continue语法：

```php
$list = ['a', 'b', 'c'];

for ($i = 0; $i < count($list); $i++) {
  if ($list[$i] == 'b') {
    break;
  }
  echo $list[$i];
}
// result a

$list = ['a', 'b', 'c'];

for ($i = 0; $i < count($list); $i++) {
  if ($list[$i] == 'b') {
    continue;
  }
  echo $list[$i];
}

//result: ac

```

# 函数

PHP函数的主要特点：

-   PHP的函数只支持单返回值。
-   如果没有返回值或者省略则接收为null，注意这里是有陷阱的，如果调用一个无返回值的方法，会获得null的结果，PHP并不会对此报错。
-   参数可以等号设置默认值。
-   可以指定参数类型，也可以省略，省略会自动根据上下文猜测类型。

```php
function sendEmail($to) {
  echo "send an email to $to";
}

sendEmail('test@test.com');

// result: send an email to test@test.com 
```

可以手动指定参数的类型，当然绝大多数情况下不会这样写（很啰嗦还浪费时间），所以看一下就可以直接忘记：

```php
function sendEmail(string $to, string $subject, string $body) {
  //...
}
```

PHP函数的参数支持定义的时候指定默认值，如果调用方没有传值就使用默认值：

```php
function sendEmail($to, $subject = 'test', $body = 'test') {
  //...
}

sendEmail('test@test.com')
```

带返回值的函数定义如下，我们同样可以手动指定函数的返回值类型：

```php
function sendEmail($to): bool {
  return true;
}

function sendEmail($to) {
  return true;
}

$success = sendEmail('test@test.com');

if ($success) {
  echo 'email sent successfully';
} else {
  echo 'error sending the email';
}

```

## 匿名函数

PHP的匿名函数和JavaScript的写法是类似的，使用变量接收不带名字的`function`方法，由于不带返回值的函数默认返回Null，所以可以认为匿名函数的变量就是Null。

匿名函数是支持变量传递的，语法是在匿名方法后面追加**use**和**括号**。

```php
$test = 'test';

$myfunction = function() use ($test) {
  echo $test;
  return 'ok';
};

$myfunction()
```

## 值传递和地址传递

PHP默认情况下的参数传递都是**值传递**，也就是说外部的参数传递在函数内部出现改变是**不会**一并改变的，因为值传递是用了一份变量副本进行数据操作。

地址传递或者说引用传递需要在参数前面加**取地址**的符号，这里的写法就类似C语言的指针了。

```php
$character = 'a';

function test(&$c) {
  $c = 'b';
}

test($character);

echo $character; //'b'
```

## 箭头函数

PHP的箭头函数相当于JS的**函数式编程**，和Java的箭头函数类似，但是箭头函数用了等号而已。

```php
$printTest = fn() => 'test';

$printTest(); //'test'

$multiply = fn($a, $b) => $a * $b;

$multiply(2, 4) //8

```

前面提到过匿名函数需要使用 `use `语句接收外部参数，而箭头函数就**不需要**如此定义便可以直接接收外部参数，写法方便和简洁易懂：

```php
$a = 2;
$b = 4;

$multiply = fn() => $a * $b;

$multiply()
```

总之PHP的函数有三种定义方法，普通函数，箭头函数和匿名函数。



## 使用map,reduce,filter函数循环数组处理

**array_map**：函数可以对于每个元素调用回调函数并且返回结果，最后会返回一个全新的列表。首个参数是回调函数，其次是列表。

**array_filter**：函数则是对于每个元素调用回调函数并且过滤掉不符合的元素，注意第一个参数是数组，然后第二个参数是回调函数，filter是符合函数回调结果的可以认为是有效的。

**array_reduce**：函数比较特殊一些，最后有一个参数有一个初始值，函数会从初始化的值对后续的每个元素进行回调函数合并，比如计算阶乘的值就可以用这个函数。

```php
$numbers = [1, 2, 3, 4];
$doubles = array_map(fn($value) => $value * 2, $numbers);

//$doubles is now [2, 4, 6, 8]

$numbers = [1, 2, 3, 4];
$even = array_filter($numbers, fn($value) => $value % 2 === 0)

//$even is now [2, 4]

$numbers = [1, 2, 3, 4];

$result = array_reduce($numbers, fn($carry, $value) => $carry * $value, 1)

```

# 面向对象

## 面向对象讨论

PHP的面向对象和JAVA的比较相似，可以说大部分语法都可以通用。

## 如何构建对象

构建对象在PHP当中也是使用new的方式，可以通过new构建多个对象，但是对象名称不能重复。

## 属性和方法

属性和方法常常配合使用，这里一并介绍魔术方法构造参数。方法可以指定构造函数 `__construction`，其中可以添加初始化对象的行为，PHP 当中对象有很多内置函数都以 **双下划线**开头。

```php
class Dog {
  public $name;

  public function __construct($name) {
    $this->name = $name;
  }

  public function bark() {
    echo $this->name . ' barked!';
  }
}

$roger = new Dog('Roger');
$roger->bark();
```

每个类默认有一个不执行任何工作的空构造器，重写之后如果无空构造函数，需要传入指定参数才能初始化，否则会出现PHP的error异常。

```php
class Dog {
  public string $name;

  public function __construct($name) {
    $this->name = $name;
  }

  public function bark() {
    echo $this->name . ' barked!';
  }
}

$roger = new Dog('Roger');
$roger->name; //'Roger'
$roger->bark(); //'Roger barked!'

// result
TypeError: Dog::__construct():
Argument #1 ($name) must be of type int,
string given on line 14
```

对象属性在PHP中存在三个限定符号，可以手动指定下面三个级别：

```php
protected

private

public
```

这几个类别分别对应了继承对象可见，私有，对外公开，和JAVA、Python语言类似，这里就不过多扩展含义和更多用法案例了。

```php
class Dog {
  public $name;
  public $age;
  public $color;
}

$roger = new Dog();

$roger->name = 'Roger';
$roger->age = 10;
$roger->color = 'gray';

var_dump($roger);

/*
object(Dog)#1 (3) {
  ["name"]=> string(5) "Roger"
  ["age"]=> int(10)
  ["color"]=> string(4) "gray"
}
*/
```

如果需要外部访问，多数情况建议用get和set的方式，对于类内部的属性首先需要定义public，其次引用需要使用**this→xxx**的方式，注意这个this是不能省略的，也是和JAVA差别比较大的点，而外部则为对象的变量引用设置的名称加上→符号，比如**dog→bark()**

```php
$roger = new Dog('Roger');
$roger->name; //'Roger'
$roger->bark(); //'Roger barked!'
```

属性只有在**public**修饰符描述的情况下才能对外访问和修改，如果为private或者protected则不行，限定符的安全访问和Java的没什么区别。

>  方法内部的\$this比较特殊，代表当前对象本身引用，和后端编程语言JAVA等类似。

## 继承

PHP的对象支持继承，具体语法如下：

```php
class Dog extends Animal {

}

$roger = new Dog();
$roger->eat();

```

## 重写

PHP重写和JAVA的规则类似，所以我们按照JAVA的对象继承理解即可：

```php
class Animal {
  public $age;

  public function eat() {
    echo 'the animal is eating';
  }
}

class Dog extends Animal {
  public function eat() {
    echo 'the dog is eating';
  }
}
```

## 静态

静态方法和静态属性都是在属性或者方法名称前面加static。对于static类或者对象内部使用self来定义，引用方式为两个冒号，比如**User::getName**。

**User::getName**标识静态变量的写法是强制规定的，否则编译器会报错`Undefined variable '$version'.intelephense(1008)`。

```php
class Utils {
  public static $version = '1.0';
}
// 静态常量的对象内部引用
self::$version;
// 静态常量的外部引用
class Utils {
  public static function version() {
    return '1.0';
  }
}
Utils::version

```

## 对象比较

前面的操作符提到过双等号和三等号有不同的含义，对于大部分情况下对象的比较`==`和`===`会返回true和false，下面的例子就是很好的解释。

```php
class Dog {
  public $name = 'Good dog';
}

$roger = new Dog();
$syd = new Dog();

echo $roger == $syd; //true

echo $roger === $syd; //false
```

## 对象遍历

对象遍历通常是遍历所有的内部属性值，可以使用关联循环的写法，这个对象遍历是PHP的一些语法特性，算是比较有意思的东西。

```php
class Dog {
  public $name = 'Good dog';
  public $age = 10;
  public $color = 'gray';
}

$dog = new Dog();

foreach ($dog as $key => $value) {
  echo $key . ': ' . $value . '<br>';
}
```

## 对象克隆

PHP的`clone` 方法和JAVA一样属于浅拷贝，深入拷贝需要额外编写一些代码。

```php
class Dog {
  public $name;
}

$roger = new Dog();
$roger->name = 'Roger';

$syd = clone $roger;
```

## 魔术方法

魔术方法可以理解为PHP为了方便开发者管理对象而提供的一些”切面“，开发者可以通过重写对象的特定方法控制行为：

```php
class Dog {
  public $name;

  public function __clone() {
    $this->cloned = true;
  }
}

$roger = new Dog();
$roger->name = 'Roger';

$syd = clone $roger;
echo $syd->cloned;
```

其他的魔术方法包含：`__call()`, `__get()`, `__set()`, `__isset()`, `__toString()`。

# 文件包含

文件包含的操作在PHP中有四种写法：`include`, `include_once`, `require`, `require_once`.

> `include` loads the content of another PHP file, using a relative path.
>
> `require` does the same, but if there’s any error doing so, the program halts. `include` will only generate a warning.

“include”：使用相对路径加载另一个PHP文档的内容。

“require”：执行相同的操作，但如果载入有任何错误进程将停止。注意“include”**只会生成警告**，require会直接抛出异常信息。

**`include_once`和`require_once`在没有`_once`的情况下执行与其相应函数相同的操作，但它们额外确保在进程执行期间仅包含一次文件。**

按照作者的经验法则是经验法则永远不要使用包含或要求，因为您可能会加载同一个文档2次，include\_once和require\_once帮助您避免此问题。

下面介绍文件包含的相关操作：

```php
require_once('test.php');

//now we have access to the functions, classes
//and variables defined in the `test.php` file

require_once('../test.php');
require_once('test/test.php');
require_once('/var/www/test/file.php');

```

# 文档系统的有用常量、函数和变量

有关文件的魔法常量：

```php
__FILE__
```

还有一个和服务器有关的全局常量：

```php
$_SERVER['SCRIPT_FILENAME']
```

除此之外其他的一些常用常量或者函数：

```php
getcwd():内置函数
__DIR__：另一个神奇常量
将__FILE__与 dirname（） 组合以获得

当前文档夹的完整路径：dirname(__FILE__)
使用 $_SERVER[“DOCUMENT_ROOT”]
```

# 错误

PHP的错误或者说异常信息分为下面三类：

-   Warnings
-   Notices
-   Errors

前面两个错误都是警告类似的，虽然有可能在程序运行过程中会出现问题但是不影响程序运行，而最后一个error则是会由PHP的解释器直接返回报错信息。

默认情况下PHP是不展示错误信息的，我们可以修改\``php.ini`\`的配置进行调整。为了更快的了解配置文件的位置和相关信息，我们可以使用 phpinfo()方法和查看：

```php
<?php
phpinfo();
?>
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105203.png)


原作者案例的对应的路径为：`/Applications/MAMP/bin/php/php8.1.0/conf/php.ini`，默认情况下为off，意味着错误将不再显示在网站中，但在这种情况下将在 MAMP（如果是别的开发脚手架则为其他的路径） 的 logs 文档夹的php\_error.log文档中看到它们。

个人的wampServer的对应错误日志信息如下：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105234.png)


我们可以指定错误日志重定向到特定的目录：

```php
; Log errors to specified file. PHP's default behavior is to leave this value
; empty.
; http://php.net/error-log
; Example:
;error_log = php_errors.log
```

添加错误信息可以通过方法error\_log('test');处理，下面截取框架的用法：

```php
 /**
     * Logs user information to webserver logs.
     *
     * @param string $user   user name
     * @param string $status status message
     *
     * @return void
     */
    public static function logUser($user, $status = 'ok')
    {
        if (function_exists('apache_note')) {
            apache_note('userID', $user);
            apache_note('userStatus', $status);
        }
        /* Do not log successful authentications */
        if (! $GLOBALS['PMA_Config']->get('AuthLogSuccess') && $status == 'ok') {
            return;
        }
        $log_file = self::getLogDestination();
        if (empty($log_file)) {
            return;
        }
        $message = self::getLogMessage($user, $status);
        if ($log_file == 'syslog') {
            if (function_exists('syslog')) {
                @openlog('phpMyAdmin', LOG_NDELAY | LOG_PID, LOG_AUTHPRIV);
                @syslog(LOG_WARNING, $message);
                closelog();
            }
        } elseif ($log_file == 'php') {
            @error_log($message);
        } elseif ($log_file == 'sapi') {
            @error_log($message, 4);
        } else {
            @error_log(
                date('M d H:i:s') . ' phpmyadmin: ' . $message . "\n",
                3, $log_file
            );
        }
    }
```

# 异常

异常通常是除开编程语言语法之外的可控错误，PHP和JAVA一样使用了`try{...}catch(Exception $e)`的方式进行处理：

```php
try {
  //do something
} catch (Throwable $e) {
//we can do something here if an exception happens
  echo $e->getMessage();
}
```

实验中我们可以使用除0的异常检查异常信息的打印：

```php
echo 1 / 0;
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105147.png)


异常处理的原则是细分不同的具体异常给出不同的提示，PHP的异常捕获规则和JAVA一致：

```php
try {
  echo 1 / 0;
} catch (DivisionByZeroError $e) {
  echo 'Ooops I divided by zero!';
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

PHP同样支持finally的写法：

```php
try {
  echo 1 / 0;
} catch (DivisionByZeroError $e) {
  echo 'Ooops I divided by zero!';
} catch (Throwable $e) {
  echo $e->getMessage();
} finally {
  echo ' ...done!';
}
```

更多异常处理可以参考下面的网站：

[https://www.php.net/manual/en/reserved.exceptions.php](https://www.php.net/manual/en/reserved.exceptions.php "https://www.php.net/manual/en/reserved.exceptions.php")

# 日期

# 常量和枚举

我们可以在 PHP 中使用 define（） 内置函数定义常量，使用语法如下：

```php
define('TEST', 'some value');
```

这种常量定义在使用的时候不需要使用\$符号：

```php
define('TEST', 'some value');

echo TEST;
```

`define` 类似 C 语言的 typeof。

其他定义常量的方法是const：

```php
const BREED = 'Siberian Husky';
```

第三种定义常量的方法是定义枚举：

```php
enum Status {
  case EATING;
  case SLEEPING;
  case RUNNING;
}
```

枚举常量的使用方法如下：

```php
class Dog {
  public Status $status;
}

$dog = new Dog();

$dog->status = Status::RUNNING;

if ($dog->status == Status::SLEEPING) {
  //...
}
```

# PHP web平台部署

PHP 是一种服务器端语言，通常以 2 种方式使用。

1.  第一种方法是类似JSP一样在HTML中嵌入PHP后端语言代码达到动态数据展示的效果。
2.  第二种是PHP更像是负责生成“应用进程”的引擎，模板语言来生成HTML，并且所有内容都由我们所谓的框架管理。（推荐）

## 处理HTTP请求

本部分介绍了在没有任何框架的情况下如何接收和处理HTTP请求，我们可以在webroot的路径创建一个test.php文件，此时如果对于脚手架配置伪静态，可以直接通过/test访问。

WEB应用绝大部分都是POST和GET请求，PHP提供了`$_GET`, `$_POST` and `$_REQUEST` 这些方法

> $ _GET：对于任何请求，您可以使用  $\_ GET 对象访问所有查询字符串数据，该对象称为超全局，并在我们所有的 PHP 文档中自动可用。
> \$\_ POST：对于 POST、PUT 和 DELETE 请求，更有可能需要以`urlencoding` 数据的形式发布的数据或使用 FormData 对象，PHP 使用`  $_POST  `为您提供该对象。
> $_REQUEST：$\_ REQUEST囊括了上面两个魔法常量的内容。

下面的案例介绍了有关这些魔法常量的用法

```php
 if (isset($_GET['tables'])) {
    $constrains = $GLOBALS['dbi']->getForeignKeyConstrains(
        $_REQUEST['db'],
        $_GET['tables']
    );
    $response = Response::getInstance();
    $response->addJSON('foreignKeyConstrains',$constrains);
}
```

## 使用cookies\$\_SEVER 对象

`$_SERVER` 包含了许多非常有用的服务器信息，我们可以使用Phpinfo方法获取服务器内容：

```php
<?php
phpinfo();
?>
```

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105136.png)


下面是基本用法：

-   `$_SERVER['HTTP_HOST']`
-   `$_SERVER['HTTP_USER_AGENT']`
-   `$_SERVER['SERVER_NAME']`
-   `$_SERVER['SERVER_ADDR']`
-   `$_SERVER['SERVER_PORT']`
-   `$_SERVER['DOCUMENT_ROOT']`
-   `$_SERVER['REQUEST_URI']`
-   `$_SERVER['SCRIPT_NAME']`
-   `$_SERVER['REMOTE_ADDR']`


\$\_GET方法的使用案例如下：

```php
<form>
  <input type="text" name="name" />
  <input type="submit" />
</form>

<?php
if (isset($_GET['name'])) {
  echo '<p>The name is ' . $_GET['name'];
}
?>
```

\$\_POST方法的使用案例如下：

```php
<form **method="POST"**>
  <input type="text" name="name" />
  <input type="submit" />
</form>

<?php
if (isset($_POST['name'])) {
  echo '<p>The name is ' . $_POST['name'];
}
?>
```

这些内容直接使用的情况比较少，通常在框架中可以看到类似的引用。


## 使用cookies

PHP通过`$_COOKIE`变量可以获取到所有和Cookie有关的信息。

```php
if (isset($_COOKIE['name'])) {
  $name = $_COOKIE['name'];
}
```

[`setcookie()`](https://www.php.net/manual/en/function.setcookie.php) 方法可以设置cookie信息，我们可以添加第三个参数来说明 cookie 何时过期。如果省略，Cookie 将在会话结束时/浏览器关闭时过期。

```php
setcookie('name', 'Flavio');
```

比如下面的代码可以设置7天之后过期：

```php
setcookie('name', 'Flavio', time() + 3600 * 24 * 7);
```

我们只能在 Cookie 中存储有限数量的数据，用户在清除浏览器数据时可以在客户端清除 Cookie。下面是Cookie使用的一些简单例子：

```php
<?php
if (isset($_POST['name'])) {
  setcookie('name', $_POST['name']);
}
if (isset($_POST['name'])) {
  echo '<p>Hello ' . $_POST['name'];
} else {
  if (isset($_COOKIE['name'])) {
    echo '<p>Hello ' . $_COOKIE['name'];
  }
}
?>

<form method="POST">
  <input type="text" name="name" />
  <input type="submit" />
</form>
```

## Sessions

Cookie 的一个非常有趣的用例是基于 cookie 的会话。

```php
<?php
session_start();
?>
```

你会发现访问对应文件之后，你将会看到一个新cookie名称（**PHPSESSID**）被定义。这个ID就是我们常说的**session ID**。

![image-20230123212351539](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/image-20230123212351539.png)

与我们使用 cookie 的方式类似，我们现在可以使用 $_SESSION 来存储用户发送的信息，但这次它不存储在客户端。只有PHPSESSID会被存储到客户端。

```php
<?php
session_start();

if (isset($_POST['name'])) {
  $_SESSION['name'] = $_POST['name'];
}
if (isset($_POST['name'])) {
  echo '<p>Hello ' . $_POST['name'];
} else {
  if (isset($_SESSION['name'])) {
    echo '<p>Hello ' . $_SESSION['name'];
  }
}
?>

<form method="POST">
  <input type="text" name="name" />
  <input type="submit" />
</form>
```

![image-20230123212507500](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/image-20230123212507500.png)

可以使用`session_unset()`方法删除Session当中的信息，如果需要移除session的cookie信息，可以使用下面的方法：

```php
setcookie(session_name(), '');
```

## IO

PHP服务端访问可以使用下面的方法：

文件是否存在`file_exists()`：

```php
file_exists('test.txt') //true
```

文件大小获取：

```php
filesize('test.txt')
```

PHP的文件读写使用同一个方法，不同的是提供访问参数使用了 读模式和写模式：

```php
$file = fopen('test.txt', 'r')
```

上面的方法为只读模式，同时提供描述信息作为返回值。我们可以调用 `fclose($fd)` 终止文件读写。

下面是把文件内容读取到变量的方法，这里吐槽一下使用要比JAVA的套版代码简单很多，也比较符合新生高级编程语言的设计思路。

```php
$file = fopen('test.txt', 'r')

fread($file, filesize('test.txt'));

//or

while (!feof($file)) {
	$data .= fgets($file, 5000);
}
```

> 注意：feof() 检查我们是否尚未到达文档末尾，因为 **fgets 一次读取 5000 字节**

逐行扫描文件套版代码：

```php
$file = fopen('test.txt', 'r')

while(!feof($file)) {
  $line = fgets($file);
  //do something
}
```

要写入文件，必须一开始打开文件的时候指定写入模式，同时配合`fwrite`函数以及`fclose`形成完整的文件读写。

```php
$data = 'test';
$file = fopen('test.txt', 'w')
fwrite($file, $data);
fclose($file);
```

删除文件的方法如下：

```php
unlink('test.txt')
```

更多文件读写方法参考：[https://www.php.net/manual/en/ref.filesystem.php](https://www.php.net/manual/en/ref.filesystem.php)

## database

所有后端语言必备的东西，这里列举了几个常用的database：

- [PostgreSQL](https://www.php.net/manual/en/book.pgsql.php)
- [MySQL](https://www.php.net/manual/en/set.mysqlinfo.php) / MariaDB
- [MongoDB](https://www.php.net/manual/en/set.mongodb.php)

绝大多数情况如果你需要一个数据库，你应该使用**框架或ORM**，这将节省SQL注入的安全问题，比如比较常见的[Laravel’s Eloquent](https://laravel.com/docs/eloquent) 框架。

## JSON

[JSON](https://flaviocopes.com/json/) 是一种可移植的数据格式，我们用于表示数据并将数据从客户端发送到服务器。PHP提供了下面两个常用方法来实现JSON字符串和对象之间的转化：

- `json_encode()` to encode a variable into JSON
- `json_decode()` to decode a JSON string into a data type (object, array…)

下面是一些简单使用例子：

```php
$test = ['a', 'b', 'c'];

$encoded = json_encode($test); // "["a","b","c"]" (a string)

$decoded = json_decode($encoded); // [ "a", "b", "c" ] (an array)
```

## email

发送邮件可以使用下面的方法：

```php
mail('test@test.com', 'this subject', 'the body');
```

[https://github.com/PHPMailer/PHPMailer](https://github.com/PHPMailer/PHPMailer) 提供了更多有邮件相关的实用API。

# 部署PHP应用Composer

Composer类似NodeJS的 NPM，和大部分的一站式依赖管理是类似的。Composer安装之后的页面内容如下：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105104.png)


可以使用`composer require nesbot/carbon` 命令对于项目进行依赖导入。正确导入之后会出现`composer.json`以及`composer.lock`文件，`composer.lock`文件可以对于依赖版本进行锁定。

```json
{
    "require": {
        "nesbot/carbon": "^2.58"
    }
}
```

当对应目录引入依赖之后，在PHP文件当中可以使用下面的用法：

```php
<?php
require 'vendor/autoload.php';

use Carbon\Carbon;

echo Carbon::now();

```

最终访问对应页面的展示内容如下：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230124105041.png)


**“require 'vendor/autoload.php';**”行作用是**启用自动加载**。这里顺带提一下还记得我们谈论require\_once（）和include\_once（）吗？我们不需要手动搜索要包含的文档，我们只需使用 **use**  关键字将库导入我们的代码即可。

# 部署PHP应用

最后作者写了一篇从零开始搭建GIT的文章比较有意思，本部分内容建议结合一些框架项目学习，博客提到的内容比较入门这里就不记录了。

See [how to setup Git and GitHub from zero](https://flaviocopes.com/github-setup-from-zero/ "how to setup Git and GitHub from zero")

# 写在最后

更多的当作英文学习资料看待，顺带能学到一些技术内容，挺不错的。
