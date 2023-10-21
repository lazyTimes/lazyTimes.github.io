---
title: '【Mysql】Working with time zones, timestamps and datetimes in Laravel and MySQL'
subtitle: Mysql的日期工作机制
url_suffix: worktimestampmysql
author: lazytime
tags:
  - Mysql
categories:
  - Mysql
keywords: Mysql的日期工作机制
description: Mysql的日期工作机制
copyright: true
date: 2023-10-21 22:24:58
---

# Source

[Working with time zones, timestamps and datetimes in Laravel and MySQL - Advanced and Qualified electronic signature marketplace (eideasy.com)](https://eideasy.com/working-with-time-zones-timestamps-and-datetimes-in-laravel-and-mysql/)

There seems to be quite a bit of confusion around how timestamps, datetimes and time zones really work. This article aims to demystify these concepts and give some recommendations and best practices on how to handle dates and time zones in a sane way in your Laravel app and MySQL.

<!-- more -->

关于时间戳、日期和时区的真正工作原理，似乎存在不少困惑。本文旨在揭开这些概念的神秘面纱，并就如何在 Laravel 应用程序和 MySQL 中以合理的方式处理日期和时区给出一些建议和最佳实践。

> 补充：Laravel 是PHP生态的框架，Java 开发人员可以忽略
## How the TIMESTAMP type works in MySQL

[The official documentation of MySQL](https://dev.mysql.com/doc/refman/8.0/en/datetime.html) explains it as follows:

>  MySQL converts TIMESTAMP values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. 
>  
>   MySQL 会将 TIMESTAMP 值从当前时区转换到 UTC 以进行存储，并在检索时从 UTC 转换回当前时区。
>  
>  (This does not occur for other types such as DATETIME.) By default, the current time zone for each connection is the server’s time. 
>
> (默认情况下，每个连接的当前时区是服务器时间。
>  
>  The time zone can be set on a per-connection basis. As long as the time zone setting remains constant, you get back the same value you store. 
>  
> 时区可根据每个连接进行设置。只要时区设置保持不变，就会返回存储的相同值。
>  
>  If you store a TIMESTAMP value, and then change the time zone and retrieve the value, the retrieved value is different from the value you stored. 
>  
>  如果存储了 **TIMESTAMP** 值，然后更改时区并检索该值，则检索到的值与存储的值不同。
>  
>  This occurs because the same time zone was not used for conversion in both directions. 
>
>出现这种情况是因为在两个方向的转换中没有使用相同的时区。
>  
>  The current time zone is available as the value of the [time_zone](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_time_zone) system variable. 
>  
>  当前时区可以通过 [time_zone](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_time_zone) 系统变量的值获得。
>  
>  For more information, see [Section 5.1.15, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html).
>  
>  更多信息，请参阅 [第 5.1.15 节，"MySQL 服务器时区支持"](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)。

This explanation is perhaps a little bit abstract. So, let’s add some context and see what is really happening behind the scenes.

这种解释可能有点抽象。因此，让我们补充一些背景知识，看看幕后到底发生了什么。
### Current time zone

In order to understand how the timestamp conversions work we first need to know what’s meant by **current time zone.**

要了解时间戳转换的工作原理，我们首先需要了解**当前时区**的含义。

In short, **current time zone** is the value of the SESSION time_zone. By default this is the SYSTEM time of the server that the database is running on. Let’s run some queries to illustrate this.

简而言之，**当前时区**就是 `Session` 时区的值。默认情况下，这是数据库所运行服务器的系统时间。让我们运行一些查询来说明这一点。

Running `SELECT @@SESSION.time_zone;` will return the current SESSSION time_zone like this:

运行 "**SELECT @@SESSION.time_zone;** "将返回当前的 SESSSION 时区，如下所示：

```
+---------------------+
| @@SESSION.time_zone |
+---------------------+
| SYSTEM              |
+---------------------+
```

Let’s change the SESSION time_zone to “+02:00” by running: `SET SESSION time_zone = '+02:00';`

运行 `SET SESSION time_zone = '+02:00';` 将 SESSION 时区更改为 "**+02:00**"。

`SELECT @@SESSION.time_zone;` will now return:

现在将返回 `SELECT @@SESSION.time_zone;`：

```
+---------------------+
| @@SESSION.time_zone |
+---------------------+
| +02:00              |
+---------------------+
```

## Practical examples of how timestamp works 时间戳工作原理实例

Let’s now go through some examples with specific dates and times to see how the timestamp storage and retrieval works in real life.

现在，让我们用具体的日期和时间举几个例子，看看时间戳的存储和检索在实际生活中是如何工作的。

We’ll start by creating a table with a TIMESTAMP column to store our test data.

首先，我们将创建一个带有 `TIMESTAMP` 列的表来存储测试数据。

```
CREATE TABLE timestamp_test (
    `timestamp` TIMESTAMP,
);
```

We’ll now set our session time_zone to +02:00 and store some data

现在，我们要将 `Session` 时区设置为 `+02:00`，并存储一些数据

```
SET SESSION time_zone = '+02:00';

INSERT INTO timestamp_test VALUES ('1970-01-01 03:00:00');
```

Check that the value got stored:

检查数值是否已存储：

```sql
SELECT * FROM timestamp_test;
```

We’ll see:

我们拭目以待：

```
+---------------------+
| timestamp           |
+---------------------+
| 1970-01-01 03:00:00 |
+---------------------+
```

Schematic representation of the storage process:

下面是存储过程示意图：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20231020075820.png)


So far so good. But what happens if we change the session time_zone?

到目前为止一切顺利。但如果我们更改会话时区，会发生什么呢？

Let’s set our session time_zone to `+00:00` and retrieve the data again.

让我们把会话时区设置为 "+00:00"，然后再次检索数据。

```
SET SESSION time_zone = '+00:00';
SELECT * FROM timestamp_test;
```

We’ll see:

我们拭目以待：

```
+---------------------+
| timestamp           |
+---------------------+
| 1970-01-01 01:00:00 |
+---------------------+
```

Schematic representation of the retrieval process:

检索过程示意图：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20231020075841.png)

**Key takeaways:**

**主要收获：**

1. MySQL stores the timestamp value as a **[Unix timestamp](https://en.wikipedia.org/wiki/Unix_time)** in seconds.
2. MySQL does **not** store any information about the timezone.
3. Every time you store a value as a timestamp, it is converted to the Unix timestamp according to the current session time_zone.
4. Every time you retrieve a timestamp, it is converted to the datetime value according to the current session time_zone.

1. MySQL 将时间戳值存储为 **[Unix时间戳](https://en.wikipedia.org/wiki/Unix_time)**，单位为秒。
2. MySQL **不存储任何有关时区的信息**。
3. 每次以时间戳存储值时，都会根据当前会话时区将其转换为 Unix 时间戳。
4. 每次检索时间戳时，都会根据当前会话时区将其转换为日期时间值。

**Note** A simple algorithm to convert dates to timestamps according to a specific timezone might look something like this (in case you’re interested how that’s actually done):

**注意**，根据特定时区将日期转换为时间戳的简单算法可能是这样的（如果你对实际操作感兴趣的话）：

1. Get the difference between the datetime and the Unix epoch (1970-01-01 00:00:00) in seconds.
2. Convert your current timezone offset to seconds
3. Subtract your current timezone offset from the value you got in step 1.

5. 获取日期时间与 Unix 时间（1970-01-01 00:00:00）之间的差值（以秒为单位）。
6. 将当前时区偏移转换为秒
7. 用步骤 1 中得到的值减去当前时区偏移。

For example, let’s say our time zone offset is +02:00 and we wish to convert 1970-01-01 03:00:00 to a Unix timestamp.

1. 1970-01-01 03:00:00 – 1970-01-01 00:00:00 = 3h = 3 * 60 * 60 = 10800
2. +02:00 in seconds is: 2 * 60 * 60 = 7200
3. 10800 – 7200 = 3600

例如，假设我们的时区偏移为 +02:00，我们希望将 1970-01-01 03:00:00 转换为 Unix 时间戳。

1. 1970-01-01 03:00:00 - 1970-01-01 00:00:00 = 3h = 3 * 60 * 60 = 10800
2. +02:00（秒）为： 2 * 60 * 60 = 7200 3.
3. 10800 - 7200 = 3600

Another example, let’s say our time zone offset is -03:00 and we wish to convert 1970-01-01 08:00:00 to a Unix timestamp.

1. 1970-01-01 08:00:00 – 1970-01-01 00:00:00 = 8h = 8 * 60 * 60 = 28800
2. -03:00 in seconds is: -3 * 60 * 60 = -10800
3. 28800 – – 10800 = 39600 (note that we actually add the values as double minuses give a +)

再比如，我们的时区偏移是 -03:00，我们希望将 1970-01-01 08:00:00 转换为 Unix 时间戳。

1. 1970-01-01 08:00:00 - 1970-01-01 00:00:00 = 8h = 8 * 60 * 60 = 28800
2. -03:00 的秒数为： -3 * 60 * 60 = -10800
3. 28800 - - 10800 = 39600（注意，我们实际上是将这些值相加，因为双减得到的是 +）
## How the TIMESTAMP type differs from the DATE and DATETIME types **TIMESTAMP 类型与 DATE 和 DATETIME 类型的区别**

In case of TIMESTAMP, the actual value that is stored and retrieved depends on the session time_zone whereas DATE and DATETIME are always retrieved as the exact same values that were stored. 

就 `TIMESTAMP` 而言，存储和检索的实际值取决于**Session 时区**，而 DATE 和 DATETIME 的检索值始终与存储值完全相同。

You can imagine the DATE and DATETIME values as static strings. 

您可以将 DATE 和 DATETIME 值想象成**静态字符串**。

The string you store does not change upon retrieval. 

您存储的字符串在检索时不会改变。

You’ll always get back the exact same value that you stored no matter the database’s or session’s time zone.

无论数据库或会话的时区如何，您都将返回所存储的**完全相同的值**。

TIMESTAMP can only store values from 1970-01-01 00:00:00 to 2038-01-19 03:14:17. The reason for this is how the Unix time is encoded: [https://en.wikipedia.org/wiki/Year_2038_problem](https://en.wikipedia.org/wiki/Year_2038_problem)

TIMESTAMP 只能存储 1970-01-01 00:00:00 至 2038-01-19 03:14:17 的值。

**原因在于 Unix 时间的编码方式： [https://en.wikipedia.org/wiki/Year_2038_problem](https://en.wikipedia.org/wiki/Year_2038_problem)**

DATETIME and DATE do not have such a limit.

`DATETIME` 和 `DATE` 就没有这种限制。
## How Laravel handles dates and times Laravel **如何处理日期和时间**

We’ve seen how the timestamp works on MySQL’s side. Let’s now see how dates and times are handled by Laravel.

我们已经了解了 MySQL 如何处理时间戳。现在让我们看看 Laravel 是如何处理日期和时间的。

Laravel uses [Carbon](https://carbon.nesbot.com/docs/) for generating dates ([https://laravel.com/docs/10.x/helpers#dates](https://laravel.com/docs/10.x/helpers#dates)). 

Laravel 使用 [Carbon](https://carbon.nesbot.com/docs/) 生成日期（[https://laravel.com/docs/10.x/helpers#dates](https://laravel.com/docs/10.x/helpers#dates)）。

Carbon in turn uses PHPs Date/Time functions [https://www.php.net/manual/en/ref.datetime.php](https://www.php.net/manual/en/ref.datetime.php). 

而 Carbon 又使用 PHP 的日期/时间函数 [https://www.php.net/manual/en/ref.datetime.php](https://www.php.net/manual/en/ref.datetime.php)。

This means that when we generate a current date then that’s done according to PHP’s timezone. 

这意味着当我们生成当前日期时，是根据 PHP 的时区来生成的。

But what determines PHP’s timezone? 

但 PHP 的时区是由什么决定的呢？

Well, Laravel conveniently does that for you via the config/app.php timezone setting.

Laravel 可以通过配置/app.php 中的时区设置为您实现这一功能。

What kind of implications does the above have on how the dates are saved to our database? We can bring an example to illustrate this.

上述情况对如何将日期保存到数据库有什么影响？我们可以举例说明。

Let’s consider the following situation:

- timezone in our app’s config/app.php is set to `Europe/Berlin`
- our database session time_zone is `Europe/Tallinn` The mysql.timezone setting in config/database.php. If you do not specifically set it, then the database will probably use the system time of the server that it’s running on.

1. We generate a date in our Laravel app using the `now()` helper function which returns us the following date: “2023-10-13 16:00:00”. This is the current datetime in `Europe/Berlin`
2. We then send “2023-10-13 16:00:00” to our MySQL database for storage in a timestamp column (for example by creating a Model and calling save() on it)
3. Our database takes “2023-10-13 16:00:00” and converts it to a Unix timestamp according to `Europe/Tallinn` timezone and then stores it. Notice what’s happening here? We generated the datetime according to `Europe/Berlin` but our database converted it to a timestamp according to `Europe/Tallinn`
4. When we retrieve the timestamp, our database converts the timestamp back to datetime according to `Europe/Tallinn` (session time_zone). Which results in “2023-10-13 16:00:00” (the original datetime we generated) So, at a glance everything seems to be ok. However, what happens if we change our app’s timezone to also be `Europe/Tallinn` ?
5. On retrieval, nothing changes, we still get back 2023-10-13 16:00:00 as the conversion depends on the database session time_zone and not on our app timezone.
6. Real issues arise when we start doing date comparisons in our app. Let’s say the date we originally saved was the creation date of a token and 30 minutes have passed since we generated it. We now wish to see whether the token is expired. For that:
    1. we get our current time with `now()` (which now generates dates according to `Europe/Tallinn` timezone as we changed our app’s timezone), we get 2023-10-13 17:30:00
    2. we get the token’s creation time from the database: 2023-10-13 16:00:00
    3. token should be valid for 1h, so we subtract the creation date from the current time and get a difference of 1.5h which seems to indicate that the token has expired. However in reality, only 30 minutes have passed.

让我们考虑一下下面的情况：
- 应用程序配置`/app.php` 中的时区设置为  `Europe/Berlin`。
- 我们的数据库会话时区是`欧洲/塔林` config/database.php 中的 mysql.timezone 设置。如果没有特别设置，数据库可能会使用运行服务器的系统时间。

7. 我们在 **Laravel** 应用程序中使用 `now()` 辅助函数生成一个日期，返回如下日期："`2023-10-13 16:00:00`"。这是当前在 `Europe/Berlin` 的日期时间。  
8. 然后，我们将 "`2023-10-13 16:00:00` "发送到 MySQL 数据库的时间戳列中（例如，通过创建一个模型并调用 `save()` 函数）。  
9. 我们的数据库接收 "`2023-10-13 16:00:00`"，并根据 `Europe/Berlin`  时区将其转换为 **Unix** 时间戳，然后将其存储起来。注意到这里发生了什么吗？我们根据 `Europe/Berlin` 生成了日期时间，但我们的数据库根据 `Europe/Berlin` 将其转换为时间戳。
10. 当我们检索时间戳时，我们的数据库又将时间戳转换成了 "`Europe/Tallinn`"（会话时区）的日期时间。结果是 "`2023-10-13 16:00:00`"（我们生成的原始日期时间）。因此，乍一看一切似乎都没问题。但是，如果我们将应用程序的时区也改为 "`Europe/Tallinn`"，会发生什么情况呢？  
11. 在检索时，没有任何变化，我们仍然得到 `2023-10-13 16:00:00`，因为转换取决于数据库会话的时区，而不是应用程序的时区。  
12. 当我们开始在应用程序中进行日期比较时，真正的问题就出现了。假设我们最初保存的日期是令牌的创建日期，而生成令牌后已经过去了 30 分钟。我们现在希望查看令牌是否过期。

为此：  
1. 我们使用 `now()` 获取当前时间（由于我们更改了应用程序的时区，因此现在根据 `Europe/Tallinn` 时区生成日期），得到 `2023-10-13 17:30:00`
2. 我们从数据库中得到令牌的创建时间：`2023-10-13 16:00:00`
3. 令牌的有效期应为 1 小时，因此我们将创建日期减去当前时间，得到 1.5 小时的差值，这似乎表明令牌已过期。但实际上只过了 30 分钟。
## Key takeaways and best practices 主要收获和最佳做法

It might seem that running the database and the Laravel app in different timezones is pretty safe if you never change the timezone configs. However, this is a risky bet to make.

如果不更改时区配置，在不同时区运行数据库和 `Laravel` 应用程序似乎很安全。然而，这样做是有风险的。

Timezone changes might easily happen if you are running many instances of your apps and databases. The majority of cloud providers set their instance timezones to UTC by default so if you are running a different timezone you need to be extra careful to always set the instances to that sepcific timezone.

如果您正在运行多个应用程序和数据库实例，时区变化就很容易发生。大多数云提供商默认将实例时区设置为 UTC，因此如果您运行的是不同的时区，则需要格外小心，始终将实例设置为该特定时区。

You’ll also need to consider daylight saving times. For example if your database session time_zone is UTC and your app timezone is `Europe/Berlin` then you’ll end up with a plethora of issues on the last Sunday of October when the offset of `Europe/Berlin` changes due to daylight saving time change.

您还需要考虑夏令时。例如，如果您的**数据库会话时区是 UTC，而应用程序时区是 `Europe/Tallinn`**，那么在十月的最后一个星期天，当 `Europe/Tallinn`的偏移量因夏令时变化而改变时，您就会遇到大量问题。

All this considered, the sanest way to handle dates in Laravel and MySQL is as follows:

1. Always set the timezones of your apps and databases to UTC. This way you’ll not have to deal with any conversion and timezone issues.
2. If you wish to display dates according to your end-user’s timezone then convert the date to end-user’s timezone just before displaying it. Avoid storing it in a different timezone.

综上所述，在 `Laravel` 和 `MySQL` 中处理日期的最合理方法如下：

1. 始终将应用程序和数据库的时区设置为 UTC。这样就不必处理任何转换和时区问题。
2. 如果您希望**根据最终用户的时区显示日期**，那么**在显示之前将日期转换为最终用户的时区**。避免将日期存储在不同的时区。

As for whether to use DATETIME or TIMESTAMP – that decision is up to you and is use case dependent.

至于是使用 DATETIME 还是 TIMESTAMP，这取决于您的使用情况。
