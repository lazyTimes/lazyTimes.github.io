---
title: '【Mysql】The DATE, DATETIME, and TIMESTAMP Types'
subtitle: 官方文档阅读+个人理解
url_suffix: mysqldate
author: lazytime
tags:
  - Mysql
categories:
  - Mysql
keywords: Mysql,Mysql时间相关参考
description: 官方文档阅读加个人理解
copyright: true
date: 2023-11-20 13:31:29
---
# Source

[https://dev.mysql.com/doc/refman/8.0/en/datetime.html](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)

# Mysql 官方文档解释

The `DATE`, `DATETIME`, and `TIMESTAMP` types are related. 

DATE"、"DATETIME "和 "TIMESTAMP "类型是相关的。

This section describes their characteristics,  how they are similar, and how they differ. MySQL recognizes `DATE`, `DATETIME`, and `TIMESTAMP` values in several formats, described in [Section 9.1.3, “Date and Time Literals”](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html "9.1.3 Date and Time Literals"). 

本节将介绍它们的特点、相似之处和不同之处。MySQL以几种格式识别`DATE`、`DATETIME`和`TIMESTAMP`值，在[第9.1.3节，"日期和时间字面"](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html "9.1.3 日期和时间字面")中描述。

For the `DATE` and `DATETIME` range descriptions, “supported” means that although earlier values might work, there is no guarantee.

对于 `DATE` 和 `DATETIME` 范围描述，"支持 "表示虽然早期值可能有效，但不能保证。

The `DATE` type is used for values with a date part but no time part. MySQL retrieves and displays `DATE` values in ``'_`YYYY-MM-DD`_'`` format. The supported range is `'1000-01-01'` to `'9999-12-31'`.

`Date` "类型用于包含日期部分但不包含时间部分的值。MySQL 以 ``'_`YYY-MM-DD`_'`` 格式检索和显示 `DATE` 值。支持的范围是`1000-01-01` 至 `9999-12-31`。

The `DATETIME` type is used for values that contain both date and time parts. 

<!-- more -->

DATETIME "类型用于包含日期和时间部分的值。

MySQL retrieves and displays `DATETIME` values in ``'_`YYYY-MM-DD hh:mm:ss`_'`` format. 

MySQL 以`YYYY-MM-DD hh:mm:ss`格式检索和显示`DATETIME`值。

The supported range is `'1000-01-01 00:00:00'` to `'9999-12-31 23:59:59'`.

支持的范围是`'1000-01-01 00:00:00'` 至`'9999-12-31 23:59:59'`。

The `TIMESTAMP` data type is used for values that contain both date and time parts. `TIMESTAMP` has a range of `'1970-01-01 00:00:01'` UTC to `'2038-01-19 03:14:07'` UTC.

“`TIMESTAMP`”数据类型用于包含日期和时间部分的值。 “TIMESTAMP”的范围为“1970-01-01 00:00:01”UTC 到“2038-01-19 03:14:07”UTC。

A `DATETIME` or `TIMESTAMP` value can include a trailing fractional seconds part in up to microseconds (6 digits) precision. 

数据时间 "或 "时间戳 "值可包括尾部小数秒部分，精度可达微秒（6 位）。

In particular, any fractional part in a value inserted into a `DATETIME` or `TIMESTAMP` column is stored rather than discarded. 

特别是，插入`DATETIME`或`TIMESTAMP`列的值中的**任何小数部分**都会被存储而不是被丢弃。

With the fractional part included, the format for these values is ``'_`YYYY-MM-DD hh:mm:ss`_[._`fraction`_]'``, the range for `DATETIME` values is `'1000-01-01 00:00:00.000000'` to `'9999-12-31 23:59:59.499999'`, and the range for `TIMESTAMP` values is `'1970-01-01 00:00:01.000000'` to `'2038-01-19 03:14:07.499999'`. 

如果包含小数部分，这些值的格式是 ``'_`YYY-MM-DD hh:mm:ss`_[._`fraction`_]'``, 

- `DATETIME`值的范围是 ``'1000-01-01 00:00:00. 000000'`到`'9999-12-31 23:59:59.499999'`
- `TIMESTAMP`值的范围是`'1970-01-01 00:00:01.000000'`到`'2038-01-19 03:14:07.499999'`。

The fractional part should always be separated from the rest of the time by a decimal point; no other fractional seconds delimiter is recognized. 

For information about fractional seconds support in MySQL, see [Section 11.2.6, “Fractional Seconds in Time Values”](https://dev.mysql.com/doc/refman/8.0/en/fractional-seconds.html "11.2.6 Fractional Seconds in Time Values").

The `TIMESTAMP` and `DATETIME` data types offer automatic initialization and updating to the current date and time. For more information, see [Section 11.2.5, “Automatic Initialization and Updating for TIMESTAMP and DATETIME”](https://dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html "11.2.5 Automatic Initialization and Updating for TIMESTAMP and DATETIME").

小数部分应始终用小数点与时间的其余部分分隔；不识别其他小数秒分隔符。有关 MySQL 支持小数秒的信息，请参阅 [第 11.2.6 节，"时间值中的小数秒"](https://dev.mysql.com/doc/refman/8.0/en/fractional-seconds.html "11.2.6 时间值中的小数秒")。

MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. 

MySQL 将 `TIMESTAMP` 值从当前时区转换到 UTC 以进行存储，并从 UTC 返回到当前时区以进行检索。

(This does not occur for other types such as `DATETIME`.) 

(这不会发生在其他类型，如 `DATETIME`）。

By default, the current time zone for each connection is the server's time. 

默认情况下，每个连接的当前时区是**服务器**时间。

The time zone can be set on a per-connection basis. 

时区可按**每个连接**设置。

As long as the time zone setting remains constant, you get back the same value you store. 

**只要时区设置保持不变，就会返回存储的相同值。**

If you store a `TIMESTAMP` value, and then change the time zone and retrieve the value, the retrieved value is different from the value you stored. 

如果存储了一个 `TIMESTAMP` 值，然后更改时区并检索该值，检索到的值将与存储的值不同。

This occurs because the same time zone was not used for conversion in both directions. 

出现这种情况是因为在两个方向的转换中没有使用相同的时区。

The current time zone is available as the value of the [`time_zone`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_time_zone) system variable. For more information, see [Section 5.1.15, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html "5.1.15 MySQL Server Time Zone Support").

当前时区可作为 [`time_zone`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_time_zone) 系统变量的值。更多信息，请参阅[第 5.1.15 节，"MySQL 服务器时区支持"](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html "5.1.15 MySQL 服务器时区支持")。

In MySQL 8.0.19 and later, you can specify a time zone offset when inserting a `TIMESTAMP` or `DATETIME` value into a table. See [Section 9.1.3, “Date and Time Literals”](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html "9.1.3 Date and Time Literals"), for more information and examples.

在 MySQL 8.0.19 及更高版本中，在表中插入 `TIMESTAMP` 或 `DATETIME` 值时，可以指定时区偏移。更多信息和示例请参阅 [第 9.1.3 节 "日期和时间字串"](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html "9.1.3 日期和时间字串")。

Invalid `DATE`, `DATETIME`, or `TIMESTAMP` values are converted to the “zero” value of the appropriate type (`'0000-00-00'` or `'0000-00-00 00:00:00'`), if the SQL mode permits this conversion. 

如果 SQL 模式允许转换，无效的 `DATE`、`DATETIME` 或 `TIMESTAMP` 值会被转换为相应类型的 "零 "值（`'0000-00-00'` 或 `'0000-00-00 00:00:00'`）。

The precise behavior depends on which if any of strict SQL mode and the [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date) SQL mode are enabled; see [Section 5.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 Server SQL Modes").

确切的行为取决于启用了严格 SQL 模式和 [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date) SQL 模式中的哪一种；请参阅 [5.1.11 节，"服务器 SQL 模式"](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 服务器 SQL 模式")。

In MySQL 8.0.22 and later, you can convert `TIMESTAMP` values to UTC `DATETIME` values when retrieving them using [`CAST()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast) with the `AT TIME ZONE` operator, as shown here:

在 MySQL 8.0.22 及更高版本中，使用带有 `AT TIME ZONE` 操作符的 [`CAST()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast)检索时，可以将 `TIMESTAMP` 值转换为 UTC `DATETIME` 值，如下所示：

```sql
mysql> SELECT col,
     >     CAST(col AT TIME ZONE INTERVAL '+00:00' AS DATETIME) AS ut
     >     FROM ts ORDER BY id;
+---------------------+---------------------+
| col                 | ut                  |
+---------------------+---------------------+
| 2020-01-01 10:10:10 | 2020-01-01 15:10:10 |
| 2019-12-31 23:40:10 | 2020-01-01 04:40:10 |
| 2020-01-01 13:10:10 | 2020-01-01 18:10:10 |
| 2020-01-01 10:10:10 | 2020-01-01 15:10:10 |
| 2020-01-01 04:40:10 | 2020-01-01 09:40:10 |
| 2020-01-01 18:10:10 | 2020-01-01 23:10:10 |
+---------------------+---------------------+
```

For complete information regarding syntax and additional examples, see the description of the [`CAST()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast) function.

有关语法和其他示例的完整信息，请参阅 [`CAST()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast) 函数的说明。

Be aware of certain properties of date value interpretation in MySQL:

注意 MySQL 中日期值解释的某些属性：

MySQL permits a “relaxed” format for values specified as strings, in which any punctuation character may be used as the delimiter between date parts or time parts.

对于指定为字符串的值，MySQL 允许使用一种 **"宽松 "格式**，其中日期部分或时间部分之间可以使用任何标点符号作为分隔符。

In some cases, this syntax can be deceiving. 

在某些情况下，这种语法可能具有欺骗性。

For example, a value such as `'10:11:12'` might look like a time value because of the `:`, but is interpreted as the year `'2010-11-12'` if used in date context. 	

例如，"'10:11:12'"这样的值可能因为": "而看起来像一个时间值，但如果在日期上下文中使用，则会被解释为 **年份"'2010-11-12'"**。	
	
The value `'10:45:15'` is converted to `'0000-00-00'` because `'45'` is not a valid month.

值`'10:45:15'`被转换为`'0000-00-00'`，因为`'45'`不是有效的月份。
    
The only delimiter recognized between a date and time part and a fractional seconds part is the decimal point.

日期和时间部分与小数秒钟部分之间的**唯一分隔符**是**小数点**。
    
The server requires that month and day values be valid, and not merely in the range 1 to 12 and 1 to 31, respectively. 

服务器要求月份和日期值必须有效，包括但不限于 1 至 12 和 1 至 31 的范围内。

With strict mode disabled, invalid dates such as `'2004-04-31'` are converted to `'0000-00-00'` and a warning is generated. 

服务器要求月份和日期值必须有效，包括但不限于 1 至 12 和 1 至 31 的范围内。

With strict mode enabled, invalid dates generate an error. 

启用严格模式后，存储无效日期会产生错误。

To permit such dates, enable [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates). 

要允许此类日期，请启用 [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates)。

> - [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates)
	  Do not perform full checking of dates. Check only that the month is in the range from 1 to 12 and the day is in the range from 1 to 31. This may be useful for Web applications that obtain year, month, and day in three different fields and store exactly what the user inserted, without date validation. This mode applies to [`DATE`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 The DATE, DATETIME, and TIMESTAMP Types") and [`DATETIME`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 The DATE, DATETIME, and TIMESTAMP Types") columns. It does not apply to [`TIMESTAMP`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 The DATE, DATETIME, and TIMESTAMP Types") columns, which always require a valid date.

	  如果启用 [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates)，则不会对日期进行严格检查。
	
	  非严格模式只检查**月是否在 1 至 12 的范围内，日是否在 1 至 31 的范围内**。这对于在三个不同字段中获取年、月、日，并准确存储用户插入的内容而不进行日期验证的网络应用程序可能很有用。这种模式适用于 [`DATE`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 日期、数据时间和 TIMESTAMP 类型") 和 [`DATETIME`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 日期、数据时间和 TIMESTAMP 类型") 列。它不适用于 [`TIMESTAMP`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html "11.2.2 DATE、DATETIME 和 TIMESTAMP 类型") 列，这些列**始终需要有效日期**。


​	
	  With [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates) disabled, the server requires that month and day values be legal, and not merely in the range 1 to 12 and 1 to 31, respectively. With strict mode disabled, invalid dates such as `'2004-04-31'` are converted to `'0000-00-00'` and a warning is generated. With strict mode enabled, invalid dates generate an error. To permit such dates, enable [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates).


​	
	  如果禁用 [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates)，服务器会要求月和日的值必须是合法的，而不只是分别在 1 至 12 和 1 至 31 的范围内。禁用严格模式后，**诸如`'2004-04-31'`之类的无效日期会被转换为`'0000-00-00'`**，并产生警告。启用严格模式后，无效日期会产生错误。
	
	  要允许使用此类日期，请启用 [`ALLOW_INVALID_DATES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_allow_invalid_dates)。

See [Section 5.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 Server SQL Modes"), for more information.
    

更多信息，请参见第 [Section 5.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 Server SQL Modes")。

- MySQL does not accept `TIMESTAMP` values that include a zero in the day or month column or values that are not a valid date. 

MySQL 不接受在日或月列中包含零的 `TIMESTAMP` 值，也不接受不是有效日期的值。

The sole exception to this rule is the special “zero” value `'0000-00-00 00:00:00'`, if the SQL mode permits this value. 

这一规则的唯一例外是特殊的 "零 "值`'0000-00-00 00:00:00'`，如果 SQL 模式允许该值的话。

The precise behavior depends on which if any of strict SQL mode and the [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date) SQL mode are enabled; see [Section 5.1.11, “Server SQL Modes”](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 Server SQL Modes").

具体行为取决于是否启用了**严格 SQL 模式**和 [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date) SQL 模式；请参阅 [5.1.11 节，"服务器 SQL 模式"](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html "5.1.11 服务器 SQL 模式")。
    
Dates containing 2-digit year values are ambiguous because the century is unknown. MySQL interprets 2-digit year values using these rules:

包含 2 位数年份值的日期是模糊的，因为世纪不明。MySQL 使用这些规则解释两位数的年份值：

Year values in the range `00-69` become `2000-2069`

00-69 "范围内的年份值变为 "**2000-2069**"。

Year values in the range `70-99` become `1970-1999`.

年份范围`70-99`中的数值变为`1970-1999`。

See also [Section 11.2.9, “2-Digit Years in Dates”](https://dev.mysql.com/doc/refman/8.0/en/two-digit-years.html "11.2.9 2-Digit Years in Dates").

另请参见 [第 11.2.9 节，"日期中的两位数年份"](https://dev.mysql.com/doc/refman/8.0/en/two-digit-years.html "11.2.9 日期中的两位数年份")。

# Linux系统如何查看设置所在的时区？

下面是查看设置所在时区的方法：

```sh
date -R
```

```mysql
[root@localhost alexxander]# date -R
Fri, 21 Jul 2023 16:29:07 -0400
```

可以看到这里使用的是美国的时区，个人安装Linux系统的时候选择的时区未做修改就会出现这样的情况。

> -0400 就是西四区。

那么我们应该如何设置Linux的所在时区？

**方法一：使用tzselect设置时区**

```mysql
[root@localhost conf]# tzselect 
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.

#? 5
Please select a country.
 1) Afghanistan		  18) Israel		    35) Palestine
 2) Armenia		  19) Japan		    36) Philippines
 3) Azerbaijan		  20) Jordan		    37) Qatar
 4) Bahrain		  21) Kazakhstan	    38) Russia
 5) Bangladesh		  22) Korea (North)	    39) Saudi Arabia
 6) Bhutan		  23) Korea (South)	    40) Singapore
 7) Brunei		  24) Kuwait		    41) Sri Lanka
 8) Cambodia		  25) Kyrgyzstan	    42) Syria
 9) China		  26) Laos		    43) Taiwan
10) Cyprus		  27) Lebanon		    44) Tajikistan
11) East Timor		  28) Macau		    45) Thailand
12) Georgia		  29) Malaysia		    46) Turkmenistan
13) Hong Kong		  30) Mongolia		    47) United Arab Emirates
14) India		  31) Myanmar (Burma)	    48) Uzbekistan
15) Indonesia		  32) Nepal		    49) Vietnam
16) Iran		  33) Oman		    50) Yemen
17) Iraq		  34) Pakistan

#? 9
Please select one of the following time zone regions.
1) Beijing Time
2) Xinjiang Time

#? 1

The following information has been given:

	China
	Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:	Sat Jul 22 05:28:59 CST 2023.
Universal Time is now:	Fri Jul 21 21:28:59 UTC 2023.
Is the above information OK?
1) Yes
2) No

#? yes
Please enter 1 for Yes, or 2 for No.

#? 1

You can make this change permanent for yourself by appending the line
	TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai

```

**tzselect命令只告诉你选择的时区的写法，并不会生效**，我们可以在各种诸如`.profile`、`.bash_profile`或者`/etc/profile`中设置正确的**TZ**环境变量并导出，例如在`/etc/profile`里面设置 `TZ='Asia/Shanghai'`：

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20231022142509.png)

修改完成之后，我们执行`source /etc/profile`让配置生效，并且使用`date -R`查看当前时间：

```sh
[root@localhost conf]# date -R
Sat, 22 Jul 2023 05:34:56 +0800

```

这里发现时间还是存在偏差问题，现在我们要让docker中的Mysql实时同步当前硬件的 clock时间，需要注意的是设置系统时间**需要root的权限**。 

`hwclock`命令用于访问服务器的硬件CMOS时间，无论读取还是设置都需要root权限，例如：

```sh
# 获取系统硬件时间
$ sudo hwclock
Fri 23 Jan 2015 03:33:17 PM CST  -0.567492 seconds

# 设置操作系统的软件时间，与系统硬件时间同步
$ sudo hwclock -s

# 设置系统硬件时间，与操作系统的软件时间同步
$ sudo hwclock -w
```

通过执行`hwclock -s`，我们可以将Linux设置为一个“近似当前时间”的时间，**Linux操作系统维护的软件时间随着服务器的长时间运行会出现漂移，最终会越来越不准确。**

**方法2：复制相应的时区文件，替换系统时区文件；或者创建链接文件**

在`/usr/share/zoneinfo/`下面有很多时区文件，可以复制这些时区文件覆盖`/etc/localtime`文件，或修改符号链接`/etc/locatime`对应的文件。

```sh
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
# CST/UTC/GMT 是什么？

**CST**：中国标准时间（China Standard Time），这个解释可能是针对**RedHat Linux**。

**UTC**：协调世界时，又称世界标准时间，简称UTC，从英文国际时间/法文协调时间”`Universal Time/Temps Cordonné`”而来。中国大陆、香港、澳门、台湾、蒙古国、新加坡、马来西亚、菲律宾、澳洲西部的时间与UTC的时差均为+8，也就是UTC+8。

**GMT**：格林尼治标准时间（旧译格林威治平均时间或格林威治标准时间；英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。
# 个人验证1：timestamp 是如何工作的

注意下面的所有实验均在控制台进行，**请不要使用Navicat进行测试**，看到的结果和控制台结果存在差异。

> MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. 

MySQL 将 `TIMESTAMP` 值从当前时区转换到 **UTC** 以进行存储，并从 UTC 返回到当前时区以进行检索。

在Mysql中可以通过下的语句查看当前的时区信息：

```mysql

SHOW VARIABLES LIKE '%time_zone%'
```

这里解释下**UTC**以及**SYSTEM**的含义：

- `system_time_zone`：表明使用系统时间。
- `time_zone`：相对于 UTC 时间的偏移，比如 '+08:00' 或者 '-6:00'。

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20231017141245.png)

**全局参数 `system_time_zone`**

系统时区，在MySQL启动时，会检查当前系统的时区，根据系统时区设置全局参数`system_time_zone`的值。

> The system time zone. When the server starts, it attempts to determine the time zone of the host machine automatically and uses it to set the[`system_time_zone`](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_system_time_zone) system variable. The value does not change thereafter.
> 
> 系统时区。服务器启动时，会尝试自动确定主机的时区，并以此设置[`system_time_zone`](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_system_time_zone) 系统变量。此后，该值不会改变。

注意，`system_time_zone` 变量只有全局值没有会话值，不能动态修改，MySQL 启动时，将尝试自动确定服务器的时区，并使用它来设置 system_time_zone 系统变量。

**全局参数 `time_zone`**

用来设置每个连接会话的时区，默认为`system`时，使用全局参数`system_time_zone`的值。

> The current time zone. This variable is used to initialize the time zone for each client that connects. By default, the initial value of this is 'SYSTEM' (which means, “use the value of system_time_zone”).
> 
> 当前时区。该变量用于为每个连接的客户端初始化时区。默认情况下，初始值为 "SYSTEM"（即 "使用 system_time_zone 的值"）。

需要注意，在一些系统中，`system_time_zone`的值是CST，中国标准时间=CST(China Standard Time) UT+8:00 ，**mysql的时区=system_time_zone+time_zone**。

下面是在`Session`当中通过更改时区对应`timestamp`的影响。 

```mysql
-- 时间戳测试表和数据
CREATE TABLE `timestamp_test`  (
  `id` varchar(50) NOT NULL COMMENT '主键',
  `time` timestamp NULL COMMENT '时间戳',
  PRIMARY KEY (`id`)
);

-- 存入 + 8 时区
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('1', '2023-10-17 13:48:55');
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('2', '2023-10-17 13:10:55');
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('3', '2023-10-17 13:22:55');

-- 查看时区
show variables like '%time_zone%';
-- system_time_zone	UTC
-- time_zone	SYSTEM

-- 更改时区
set session time_zone = '+8:00';
-- 查看时间
select id,`time` from timestamp_test;
-- 1	2023-10-17 13:48:55
-- 2	2023-10-17 13:10:55
-- 3	2023-10-17 13:22:55


set session time_zone = '+2:00';
-- 查看时间
select id,`time` from timestamp_test;
-- +----+---------------------+
-- | id | time                |
-- +----+---------------------+
-- | 1  | 2023-10-17 15:48:55 |
-- | 2  | 2023-10-17 15:10:55 |
-- | 3  | 2023-10-17 15:22:55 |
-- +----+---------------------+


```

在当前session修改时区之后，对应的`timestamp`读取时间也出现变化。
# 个人验证2：时区设置影响

参考：[https://opensource.actionsky.com/20211214-time_zone/](https://opensource.actionsky.com/20211214-time_zone/)
## 1.`NOW()` 和 `CURTIME()` 系统函数的返回值受当前 **session** 的时区影响

`select now()`，包括`insert .. values(now())`、以及字段的 `DEFAULT CURRENT_TIMESTAMP` 属性也受此影响。这里依旧使用上面的案例：

```mysql
-- 时间戳测试表和数据
CREATE TABLE `timestamp_test`  (
  `id` varchar(50) NOT NULL COMMENT '主键',
  `time` timestamp NULL COMMENT '时间戳',
  PRIMARY KEY (`id`)
);

-- 存入 + 8 时区
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('1', '2023-10-17 13:48:55');
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('2', '2023-10-17 13:10:55');
INSERT INTO `timestamp_test` (`id`, `time`) VALUES ('3', '2023-10-17 13:22:55');

```

更改数据库时区为`+02:00`之后，对应结果如下：

```mysql
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | UTC    |
| time_zone        | +02:00 |
+------------------+--------+
2 rows in set (0.01 sec)

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2023-07-21 21:51:48 |
+---------------------+
1 row in set (0.00 sec)

```

可以看到，修改`time_zone`会对函数的结果产生影响。
##  2.timestamp 数据类型字段存储的数据受时区影响

根据Mysql文档的描述，`timestamp` 数据类型会存储**当前session的时区信息**，读取时会根据当前 session 的时区进行转换，而`date`和`datetime`则不会因为时区的变更而出现数据变更。
# 国内安装Mysql的时区问题避坑

## 1.明确指定时区

在 `my.cnf` 写入 `default-time-zone='+08:00'`，其他地区和开发确认取对应时区即可。

至于为什么要明确指明时区，一方面是Mysql 在很多没有DBA的公司都是全部由运维负责，运维如果没有设置时区，在数据库迁移到海外服务器的时候可能会出现时区变更的各种问题，另一方面是这样明确的设置可以**减少系统计算的开销**，如果系统大量使用timestamp的数据类型，这也是一个不小的优化点。
## 2.JAVA应用读取到的时间和北京时间差了14个小时，为什么？怎么解决？

基于`mysql-connector-java-8.0.19` 进行分析，使用 `com.mysql.cj.jdbc.Driver`，在获取`java.sql.Connection`的过程会确定时区，调用如下方法。

```java
com.mysql.cj.protocol.a.NativeProtocol#configureTimezone
  
/**
    * Configures the client's timezone if required.
    * 
    * @throws CJException
    *             if the timezone the server is configured to use can't be
    *             mapped to a Java timezone.
    */
public void configureTimezone() {
String configuredTimeZoneOnServer = this.serverSession.getServerVariable("time_zone");
	
	if ("SYSTEM".equalsIgnoreCase(configuredTimeZoneOnServer)) {
	configuredTimeZoneOnServer = this.serverSession.getServerVariable("system_time_zone");
	}
	
	String canonicalTimezone = getPropertySet().getStringProperty(PropertyKey.serverTimezone).getValue();
	
	if (configuredTimeZoneOnServer != null) {
	// user can override this with driver properties, so don't detect if that's the case
	if (canonicalTimezone == null || StringUtils.isEmptyOrWhitespaceOnly(canonicalTimezone)) {
	    try {
	    canonicalTimezone = TimeUtil.getCanonicalTimezone(configuredTimeZoneOnServer, getExceptionInterceptor());
	    } catch (IllegalArgumentException iae) {
	    throw ExceptionFactory.createException(WrongArgumentException.class, iae.getMessage(), getExceptionInterceptor());
	    }
	}
	}
	
	if (canonicalTimezone != null && canonicalTimezone.length() > 0) {
	this.serverSession.setServerTimeZone(TimeZone.getTimeZone(canonicalTimezone));
	
	//
	// The Calendar class has the behavior of mapping unknown timezones to 'GMT' instead of throwing an exception, so we must check for this...
	//
	if (!canonicalTimezone.equalsIgnoreCase("GMT") && this.serverSession.getServerTimeZone().getID().equals("GMT")) {
	    throw ExceptionFactory.createException(WrongArgumentException.class, Messages.getString("Connection.9", new Object[] { canonicalTimezone }),
	                                            getExceptionInterceptor());
	}
	}
	
	this.serverSession.setDefaultTimeZone(this.serverSession.getServerTimeZone());
}

```

上面的代码大致逻辑如下：

1. 从mysql的`time_zone`读取值。

2. 如果值是`SYSTEM`则使用`system_time_zone`的值.

3. 如果`jdbc url`配置了时区则使用url里的，如 `jdbc:mysql://localhost:3306/test?useSSL=true&serverTimezone=Asia/Shanghai`，则最优先使用URL设置的时区。

那么为什么JAVA应用读取到的时间和北京时间差了14个小时？通常是因为**没有在URL里面设置时区属性**，某些系统下，MySQL默认使用的是系统时区**CST**（CST 在 **RedHat** 上是 +08:00 时区），而应用和MySQL 建立的连接的`session time_zone`为`CST`。

实际上，`CST` 一共能代表4个时区：

- `Central Standard Time (USA) UT-6:00` 美国标准时间
- `Central Standard Time (Australia) UT+9:30 `澳大利亚标准时间
- `China Standard Time UT+8:00` 中国标准时间
- `Cuba Standard Time UT-4:00` 古巴标准时间

虽然Mysql正确认识了CST是中国标准时间，但是JDBC却没有认识这个时间，**JDBC在解析CST时使用了美国标准时间，这就会导致时区错误**。

具体可以看下面的代码：

```java
public class TimeTest {  
  
    public static void main(String[] args) {  
        // 这里的CST指的是美国中部时间，  
        TimeZone tz = TimeZone.getTimeZone("CST");  
        System.out.println("tz => "+ tz);// 可以看到偏移量是offset=-21600000，-21600000微秒=-6小时，所以这里的CST指美国  
  
    // 建议创建 TimeZone 用 ZoneId，因为ZoneId 不允许 CST、JST 这种简称，能提前预防进坑，如下  
    // ZoneId zoneId = ZoneId.of("CST");// 抛异常  
        ZoneId zoneId = ZoneId.of("GMT+8");// 明确指定，是OK的，或者 "区域/城市" 的写法如 Asia/Shanghai        TimeZone tz1 = TimeZone.getTimeZone(zoneId);  
        System.out.println("tz1 => "+ tz1);  
    }/**运行结果：  
     tz => sun.util.calendar.ZoneInfo[id="CST",offset=-21600000,dstSavings=3600000,useDaylight=true,transitions=235,lastRule=java.util.SimpleTimeZone[id=CST,offset=-21600000,dstSavings=3600000,useDaylight=true,startYear=0,startMode=3,startMonth=2,startDay=8,startDayOfWeek=1,startTime=7200000,startTimeMode=0,endMode=3,endMonth=10,endDay=1,endDayOfWeek=1,endTime=7200000,endTimeMode=0]]  
  
     tz1 => sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]      */}
```
## 3.修改MySQL的时区会影响已经存储的时间类型数据吗？

答案是**只会影响对 timestamp 数据类型的读取**。
## 4.迁移数据时会有导致时间类型数据时区错误的可能吗？

这一点依然是**针对 timestamp 数据类型**，比如使用 mysqldump 导出 csv 格式的数据，默认这种导出方式会使用 UTC 时区读取 `timestamp` 类型数据，这意味导入时必须手工设置 `session.time_zone=’+00:00’`才能保证时间准确。

```mysql
--将 test.t 导出成 csv
mysqldump -S /data/mysql/data/3306/mysqld.sock --single-transaction \
--master-data=2 -t -T /data/backup/test3 --fields-terminated-by=',' test t

--查看导出数据
cat /data/backup/test3/t.txt
2021-12-02 08:45:39,2021-12-02 16:45:39
```

为了避免这些繁琐的操作，mysqldump 也提供了一个参数 `--skip-tz-utc`，意思就是导出数据的那个连接不设置 UTC 时区，使用 MySQL 的 **gloobal time_zone 系统变量值**。

当然这个设置也算是告诉我们，**mysqldump 导出默认也是使用 UTC 时区**，为了确保导出和导入的时区正确，会在导出的 sql 文件头部带有 **session time_zone 信息**。

> 需要注意 `--compact` 参数会去掉 sql 文件的所有头信息，所以`--compact` 参数得和 `--skip-tz-utc` 一起使用。
# 为什么mysql有system_time_zone和time_zone两个?

见 [https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)

1. Mysql 服务的时候会读取Linux宿主机所在的时区，固定值之后这个值不再改变。（简单说 `system_time_zone` 是Mysql系统算出来的值）。
2. 默认情况 `time_zone` 值为 **SYSTEM**，也就是说它跟随`system_time_zone`的值。
3. 注意`system_time_zone`的值固定下来后，数据库宿主机的时区再改变，`time_zone`的值都是不变的，因为它是跟随`system_time_zone`变量的，不是实时跟随操作系统的，如果想要让他跟随操作系统，最简单的方法就是重启Mysql。
4. 有时候我们会发现，**Linux时区是对的，但是mysql的时区是错**，这时候我们把Linux的时区改对，但是发现Mysql还是错的，原因是**Linux时区改对之后没有重启Mysql服务器重新读取Linux系统时区**。

