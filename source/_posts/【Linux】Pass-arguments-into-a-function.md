---
title: 【Linux】Pass arguments into a function
subtitle: Linux函数参数
author: lazytime
url_suffix: passarguments
tags:
  - Linux
categories:
  - Linux
keywords: 函数,linux
description: Linux函数
copyright: true
date: 2023-09-13 13:47:10
---
# Source

[Pass arguments into a function - Linux Bash Shell Scripting Tutorial Wiki (cyberciti.biz)](https://bash.cyberciti.biz/guide/Pass_arguments_into_a_function)

Let us see how to pass parameters to a Bash function.

让我们看看如何向 Bash 函数传递参数。

- A shell function is nothing but a set of one or more commands/statements that act as a complete routine.
- Each function must have a unique name.
- Shell functions have their own command line argument or parameters.
- Use shell variable [$1](https://bash.cyberciti.biz/guide/$1 "$1"), [$2](https://bash.cyberciti.biz/guide/$2 "$2"),..$n to access argument passed to the function.

- shell 函数是由一条或多条命令/语句组成的一个完整例程。
- 每个函数都必须有一个唯一的名称。
- shell 函数有自己的命令行参数。
- 使用 shell 变量 [$1](https://bash.cyberciti.biz/guide/$1"$1")、[$2](https://bash.cyberciti.biz/guide/$2"$2")...$n 访问传递给函数的参数。



<!-- more -->

# Passing parameters to a Bash function向 Bash 函数传递参数

- The syntax is as follows to create user-defined functions in a shell script:

在 shell 脚本中创建用户定义函数的语法如下：

```sh
function_name(){
   command_block_here
}

## OR ##

function function_name_here(){
   command_line_block
}

## passing parameters to a Bash function ##

my_function_name(){
  arg1=$1
  arg2=$2
  command on $arg1
}
```
# Invoke function

- To invoke the the function use the following syntax:

要调用该函数，请使用以下语法：

```
my_function_name foo bar
```

Where,

1. **my_function_name** = Your function name.
2. **foo** = Argument # 1 passed to the function (positional parameter # 1).
3. **bar** = Argument # 2 passed to the function.

1. **my_function_name** = 您的函数名称。
2. **foo** = 传递给函数的参数 # 1（位置参数 # 1）。
3. 3. **bar** = 传递给函数的参数 # 2。

# Examples

Create a function called fresh.sh:

创建一个名为 fresh.sh 的函数：

```sh
#!/bin/bash
 
# write a function
fresh(){
   # t stores $1 argument passed to fresh()
   t=$1
   echo "fresh(): \$0 is $0"
   echo "fresh(): \$1 is $1"
   echo "fresh(): \$t is $t"
   echo "fresh(): total args passed to me $#"
   echo "fresh(): all args (\$@) passed to me -\"$@\""
   echo "fresh(): all args (\$*) passed to me -\"$*\""
}
 
# invoke the function with "Tomato" argument
echo "**** calling fresh() 1st time ****"
fresh Tomato
 
# invoke the function with total 3 arguments
echo "**** calling fresh() 2nd time ****"
fresh Tomato Onion Paneer
```

Save and close the file. Run it as follows:

保存并关闭文件。按以下步骤运行：

```sh
chmod +x fresh.sh
./fresh.sh
```

Sample outputs:

样本输出：

```sh
**** calling fresh() 1st time ****
fresh(): $0 is ./fresh.sh
fresh(): $1 is Tomato
fresh(): $t is Tomato
fresh(): total args passed to me 1
fresh(): all args ($@) passed to me -"Tomato"
fresh(): all args ($*) passed to me -"Tomato"
**** calling fresh() 2nd time ****
fresh(): $0 is ./fresh.sh
fresh(): $1 is Tomato
fresh(): $t is Tomato
fresh(): total args passed to me 3
fresh(): all args ($@) passed to me -"Tomato Onion Paneer"
fresh(): all args ($*) passed to me -"Tomato Onion Paneer"
```

Let us try one more example. Create a new shell script to determine if given name is file or directory (cmdargs.sh):

让我们再试一个例子。创建一个新的 shell 脚本来判断给定的名称是文件还是目录（cmdargs.sh）：

```sh
#!/bin/bash
# Name - cmdargs.sh
# Purpose - Passing positional parameters to user defined function 
# 目的 - 将位置参数传递给用户定义函数 
# -----------------------------------------------------------------
file="$1"

# User-defined function
is_file_dir(){
        # $f is local variable
	local f="$1"
        # file attributes comparisons using test i.e. [ ... ]
	[ -f "$f" ] && { echo "$f is a regular file."; exit 0; }
	[ -d "$f" ] && { echo "$f is a directory."; exit 0; }
	[ -L "$f" ] && { echo "$f is a symbolic link."; exit 0; }
	[ -x "$f" ] && { echo "$f is an executeble file."; exit 0; }
}

# make sure filename supplied as command line arg else die
# 确保文件名作为命令行参数提供，否则 die
[ $# -eq 0 ] && { echo "Usage: $0 filename"; exit 1; }

# invoke the is_file_dir and pass $file as arg
# 调用 is_file_dir，并将 $file 作为参数传入
is_file_dir "$file
```

Run it as follows:

运行方法如下

```sh
./cmdargs.sh /etc/resolv.conf
./cmdargs.sh /bin/date
./cmdargs.sh $HOME
./cmdargs.sh /sbin
```

Sample outputs:

样本输出：

```sh
/etc/resolv.conf is a regular file.
/bin/date is a regular file.
/home/vivek is a directory.
/sbin is a directory.
```

# Function shell variables

- All function parameters or arguments can be accessed via $1, $2, $3,..., $N.
- [$0](https://bash.cyberciti.biz/guide/$0 "$0") always point to the shell script name.
- [$*](https://bash.cyberciti.biz/guide/$* "$*") or [$@](https://bash.cyberciti.biz/guide/$@ "$@") holds all parameters or arguments passed to the function.
- [$#](https://bash.cyberciti.biz/guide/$ "$") holds the number of positional parameters passed to the function.

- 所有函数参数或参数都可以通过` $1, $2, $3,..., $N` 访问。
- [$0](https://bash.cyberciti.biz/guide/$0 "$0")总是指向 shell 脚本名。
- [$* ](https://bash.cyberciti.biz/guide/$* "$*")  或  [$@](https://bash.cyberciti.biz/guide/$@ "$@") 保存传递给函数的所有参数。
- [$#](https://bash.cyberciti.biz/guide/$ "$") 保存传递给函数的位置参数的个数。

## How do I display function name? 如何显示功能名称？

[$0](https://bash.cyberciti.biz/guide/$0 "$0") always point to the shell script name. However, you can use an array variable called [FUNCNAME](https://bash.cyberciti.biz/wiki/index.php?title=FUNCNAME&action=edit&redlink=1 "FUNCNAME (page does not exist)") which contains the names of all shell functions currently in the execution call stack. The element with index 0 is the name any currently-executing shell function.This variable _exists only_ when a shell function is executing.

[$0](https://bash.cyberciti.biz/guide/$0 "$0")总是指向 shell 脚本名。不过，您可以使用一个名为 [FUNCNAME](https://bash.cyberciti.biz/wiki/index.php?title=FUNCNAME&action=edit&redlink=1 "FUNCNAME (page does not exist)")的数组变量，它包含当前执行调用堆栈中所有 shell 函数的名称。索引为 0 的元素是当前正在执行的 shell 函数的名称。

**FUNCNAME in action**

Create a shell script called funcback.sh:

创建名为 funcback.sh 的 shell 脚本：

```sh
#!/bin/bash
#  funcback.sh : Use $FUNCNAME
backup(){
	local d="$1"
	[[ -z $d ]] && { echo "${FUNCNAME}(): directory name not specified"; exit 1; }
	echo "Starting backup..."
}

backup $1
```

Save and close the file. Run it as follows:

保存并关闭文件。按以下步骤运行：

```sh
chmod +x funcback.sh
funcback.sh /home
funcback.sh
```

Sample outputs:

样本输出：

```sh
backup(): directory name not specified
```

Return values

返回值

It is possible to pass a value from the function back to the bash using the [return command](https://bash.cyberciti.biz/guide/Return_command "Return command"). The return statement terminates the function. The syntax is as follows:

可以使用 [return command](https://bash.cyberciti.biz/guide/Return_command "Return command")将函数值传回 bash。return 语句终止函数。语法如下

```sh
return
return [value]
```

One can force script to exit with the return value specified by [value]. If [value] is omitted, the return status is that of the last command executed within the function or script.

可以强制脚本以 [value] 指定的返回值退出。如果省略 [value]，返回状态将是函数或脚本中最后执行的命令的状态。
## Examples

Create a new file called math.sh:

新建一个名为 math.sh 的文件：

```sh
#!/bin/bash
#!/bin/bash
# Name - math.sh
# Purpose - Demo return value 
# ------------------------------------

## user define function
math(){
	local a=$1
	local b=$2
	local sum=$(( a + b))
	return $sum
}

## call the math function with 5 and 10 as  arguments 
math 5 10

## display back result (returned value) using $?
echo "5 + 10 = $?"
```

Run it as follows:

运行方法如下

```sh
chmod +x math.sh
./math.sh
```

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230830135648.png)

The above example is straightforward. The [return command](https://bash.cyberciti.biz/guide/Return_command "Return command") returns any given value between **zero (0) and 255**. By default, the value passed by the return statement is the current value of the [exit status](https://bash.cyberciti.biz/guide/The_exit_status_of_a_command "The exit status of a command") variable. For example, the following code will return wrong values as it is not between **zero (0) and 255**.

上面的例子很简单。[return 命令](https://bash.cyberciti.biz/guide/Return_command "返回命令") 返回介于**零 (0) 和 255** 之间的任意给定值。默认情况下，return 语句传递的值是 [exit status](https://bash.cyberciti.biz/guide/The_exit_status_of_a_command "命令的退出状态") 变量的当前值。例如，以下代码将返回错误的值，因为它不在**零（0）和 255** 之间。

```sh
math 500 100
```

Here is the proper use of [return command](https://bash.cyberciti.biz/guide/Return_command "Return command") (bash-task.sh):

下面是 [return command](https://bash.cyberciti.biz/guide/Return_command )的正确用法（bash-task.sh）：

```sh
#!/bin/bash
# Name: bash-task.sh
# Purpose: Check if a file exists or not using custom bash function and return value
# -----------------------------------------------------------------------------------

# set values 
readonly TRUE=0
readonly FALSE=1

# get input from the CLI
file="$1"

# return $TRUE (0) if file found 
# return $FALSE (1) if file not found
is_file_found(){
	[ -f "$1" ] && return $TRUE || return $FALSE
}

# show usage info if $1 not passed to our script
if [ $# -eq 0 ]
then
	echo "Usage: $0 filename"
	exit 1
fi

# let us call our function 
is_file_found "$file"

# take action based upon return value
if [ $? -eq 0 ]
then
	echo "$file added to backup task"
else
	echo "$file not found."
fi
```

Run it as follows:

运行方法如下

```sh
chmod +x bash-task.sh
./bash-task.sh
./bash-task.sh /etc/passwd
./bash-task.sh /foo/bar
```

![image.png](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/20230830140338.png)

> Passing parameters to a Bash function and return true or false value
> 向 Bash 函数传递参数并返回 true 或 false 值
# How to return custom value from a user-defined function?

As I said earlier, you can't use [return command](https://bash.cyberciti.biz/guide/Return_command "Return command"). However, the workaround is as following using the [printf command](https://bash.cyberciti.biz/guide/Printf_command "Printf command")/[echo command](https://bash.cyberciti.biz/guide/Echo_command "Echo command"):

正如前面所说，不能使用 [return command](https://bash.cyberciti.biz/guide/Return_command "Return command")。不过，变通方法如下：使用 [printf 命令](https://bash.cyberciti.biz/guide/Printf_command "Printf 命令")/[echo命令](https://bash.cyberciti.biz/guide/Echo_command "Echo 命令")：

```sh
#!/bin/bash
#!/bin/bash
# Name - math.sh
# Purpose - Demo how to return custom value 
# -----------------------------------------

## user define function that use echo/printf 
math(){
	local a=$1
	local b=$2
	local sum=$(( a + b))
	echo $sum
}

## call the math function with 5 and 10 as  arguments 
total=$(math 500 42)

## display back result (returned value) using $?
echo "500 + 42 = $total"
```
# Listing functions 列出函数

Use the [typeset command](https://bash.cyberciti.biz/wiki/index.php?title=Typeset_command&action=edit&redlink=1 "Typeset command (page does not exist)") or [declare command](https://bash.cyberciti.biz/guide/Declare_command "Declare command").

使用 [typeset command](https://bash.cyberciti.biz/wiki/index.php?title=Typeset_command&action=edit&redlink=1 "Typeset command (page does not exist)")或 [declare command](https://bash.cyberciti.biz/guide/Declare_command "Declare command")。
# Show the known functions and their code/definitions显示已知函数及其代码/定义

Open the terminal and then run:

打开终端，然后运行：

```sh
declare -f
declare -f function_name_here
## or ##
typeset -f
typeset -f function_name_here
```
# How to remove or unset functions 如何删除或取消设置函数

Use the [unset command](https://bash.cyberciti.biz/wiki/index.php?title=Unset_command&action=edit&redlink=1 "Unset command (page does not exist)") to unset values and attributes of [shell variables](https://bash.cyberciti.biz/guide/Variables "Variables") and [functions](https://bash.cyberciti.biz/guide/Functions "Functions"):

使用 [unset 命令](https://bash.cyberciti.biz/wiki/index.php?title=Unset_command&action=edit&redlink=1 "Unset 命令（页面不存在）") 取消设置 [shell 变量](https://bash.cyberciti.biz/guide/Variables "变量") 和 [函数](https://bash.cyberciti.biz/guide/Functions "函数") 的值和属性：

```sh
unset -f function_name_here
unset -f foo
unset -f bar
```
