---
title: shell编程
date: 2023-06-09 00:00:00
tags:
categories:
- shell
---

#### 第一个shell脚本

创建一个 test.sh 文件

```sh
#!/bin/bash
echo "Hello World !"
```

+ 作为可执行程序运行

```sh
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

+ 作为解释器参数运行

```sh
/bin/sh test.sh
sh test.sh
/bin/bash test.sh
bash test.sh
source test.sh
. test.sh # . 等于 source 命令
```

运行结果

```sh
helloworld
```

#### 函数返回值

+ 使用 echo

```sh
#!/bin/bash

function getName() {
  echo "wlzhou"
}

echo "name = $(getName)"
```

运行结果

```
name = wlzhou
```

+ 使用 return

```sh
#!/bin/bash

function getAge() {
  return 100
}

getAge

#$?返回上条语句的执行结果,正常是0
echo "age = $?"

#ls
#echo "ls = $?"
#abc
#echo "abc = $?"
```

运行结果

```
age = 100
```

#### case 语法

```sh
read answer
case $answer in
  0) echo "input 0";;
  1|2|3) echo "input 1|2|3";;
  a|A) echo "input a|A";;
  ?) echo "input single character";;
  *) echo "input other";;
esac
```

#### 函数相关

定义一个 show 函数

```sh
function show() {
    echo "---hello,world---"
}

show #执行 ---hello,world---
```

#### 循环语句

for 循环

```sh
arr="1 2 3 4 5"
for i in $arr # for i in 1 2 3 4 5, for i in {1..5}, for i in `seq 1 5, for i in `seq 5`
do
  echo $i
done
```

运行结果

```
1
2
3
4
5
```

while 循环

```sh
var=0
while [ $var -lt 3 ]
do
  echo "var = $var"
  var=$(($var+1))
done
```

运行结果

```
var = 0
var = 1
var = 2
```

break 和 continue

```sh
var=0

while [ $var -lt 10 ]; do
  var=$(($var+1))
  if [ $var -eq 3 ]; then
    continue
  fi
  if [ $var -eq 5 ]; then
    break
  fi
  echo "var = $var"
done
```

运行结果

```
var = 1
var = 2
var = 4
```

#### 条件判断

判断字符串是否为空

```sh
# -z 为空, -n 不为空
str=""

if [ -z "$str" ]; then
  echo "str is null"
else
  echo "str is not null"
fi

# str is null
```

判断字符串是否相等

```sh
str="wlzhou"

if [ $str = "wlzhou" ]; then
  echo "equal"
else
  echo "not equal"
fi

# equal
```

判断数字是否相等

```sh
# -eq(=) -gt(>) -lt(<) -ge(>=) -le(<=)
a=5
b=10
if [ $a -eq $b ]; then
  echo "equal"
else
  echo "not equal"
fi

# not equal
```

文件判断

```sh
if [ -f test.sh ]; then
  echo "is file"
else
  echo "is not file"
fi

# is file
```

目录判断

```sh
if [ -d system ]; then
  echo "is dir"
else
  echo "is not dir"
fi

# is dir
```

逻辑操作

```sh
# -o -a
if [ -d system -a -f test.sh ]; then
  echo "is ok"
else
  echo "not ok"
fi

# is ok
```

#### 其他操作

变量的定义和初始化

```sh
name="wlzhou"
echo "name = $name" # name = wlzhou
echo "${name}ok" # wlzhouok
```

shell 中执行 linux 命令

```sh
PWD=`pwd`
echo "PWD = $PWD" # PWD = /media/wlzhou/bak/gwn73xx
```

(())处理数字运算

```sh
for((i = 0;i<3;i++))
do
  echo "i = $i"
done
```

运行结果

```
i = 0
i = 1
i = 2
```

#### 参考

+ [Shell 教程](https://www.runoob.com/linux/linux-shell.html)
