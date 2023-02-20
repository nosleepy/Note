---
title: NDK学习
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### C和C++基础

学习链接
+ [c语言教程](https://www.runoob.com/cprogramming/c-tutorial.html)
+ [c++教程](https://www.runoob.com/cplusplus/cpp-tutorial.html)

c语音编程模板

```c
#include <stdio.h>

int main()
{
   /*  Write C code in this online editor and run it. */
   printf("Hello, World! \n");
   
   return 0;
}
```

c++编程模板

```c++
#include <iostream>
using namespace std;

int main()
{
   cout << "Hello World";
   return 0;
}
```

动态库编译

创建test.h和test.c文件

```c
//test.h
#include <stdio.h>
void test();

//test.c
#include <stdio.h>
void test() {
    printf("test hello world");
}
```

编译so动态库

```shell
gcc test.c -fPIC -shared -o test.so # -o编译动态库
```

#### 编译过程

源文件Main.cpp

```shell
#include <iostream>
using namespace std;

int main() {
	cout << "helloworld\n";
	return 0;
}
```

1. 预处理

```shell
g++ -E test.cpp -o test.i # 生成预处理后的文件
```

2. 编译

```shell
g++ -S test.cpp -o test.s # 生成汇编代码文件
```

3. 汇编

```shell
g++ -c test.s -o test.o # 生成二进制机器码文件
```

4. 链接

```shell
g++ test.o -o test # 生成可执行文件
```

执行 **./test** 后输出 **helloworld**。

一次性生成可执行文件

```shell
g++ test.cpp # c文件用gcc命令，c++文件用g++命令
```

#### 静态库和动态库

+ 静态库：一些目标文件（.o结尾）的集合，一般以.a结尾，只用于生成可执行文件阶段。

```shell
# 首先生成目标文件
g++ -c test.c -o test.o
# 使用ar命令将目标文件打包成静态库
ar rcs libtest.a test.o
```

创建tool.h、tool.cpp、test.cpp测试静态库

tool.h

```c++
int add(int a, int b);
```

tool.cpp

```c++
#include "tool.h"

int add(int a, int b) {
	return a + b;
}
```

test.cpp

```c++
#include <iostream>
#include "tool.h"
using namespace std;

int main() {
	int res = add(1, 2);
	cout << "res = " << res;
}
```

生成静态库.a文件

```shell
g++ -c tool.cpp # 生成tool.o文件
ar rcs libtool.a tool.o # 打包libtool.a静态库文件
g++ -o main test.cpp -L. -ltool # 生成main可执行程序
./main # 输出 res = 3
```

+ 动态库：在链接阶段没有复制到程序中，而是在程序运行时由系统动态加载到内存中供程序使用。

```shell
# 首先生成目标文件
g++ -c test.c -o test.o
# 使用-fPIC和-shared生成动态库
g++ -shared -fPIC -o libtest.so test.o
```

创建tool.h、tool.cpp、test.cpp测试动态库

```shell
g++ -shared -fPIC -o libtool.so tool.o # 生成 libtool.so 动态库文件
g++ -o main test.cpp -L. -ltool # 生成main可执行程序
LD_LIBRARY_PATH=. ./main # 输出 res = 3
```

#### makefile相关

+ makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要重新编译如何进行链接等操作。
+ makefile就是"自动化编译"，告诉make命令如何编译和链接。

当前目录存在test.c、tool.c、tool.c三个文件，命令方法生成main可执行文件。

```shell
g++ -c tool.cpp
g++ -c test.cpp
g++ -o main test.o tool.o
./main
```

下面是makefile文件内容

```
define func
$(info "hello")
endef

$(call func)

define func1
$(info $(1) $(2))
endef

$(call func1,hello,world)

objects = main.o tool.o
main: $(objects)
	g++ -o main $(objects)
.PHONY: clean
clean:
	-rm main $(objects)
```

执行**make**命令自动生成test.o、tool.o、main文件。

#### Android.mk基础

是一个向Android NDK构建系统描述NDK项目的GNU makefile片段。主要用来编译生成APK程序、Java程序、C/C++应用程序、C/C++静态库、C/C++共享库。

app/src/main目录下创建ndkBuild文件夹，创建hello-jni.c、Android.mk文件

```c
#include <jni.h>

int test() {
    return 123;
}

jint Java_com_grandstream_myapplication_MainActivity_nativeTest() {
    return test();
}
```

```mk
# 定义模块当前路径
LOCAL_PATH := $(call my-dir)
# 清空当前环境变量
include $(CLEAR_VARS)
# 当前模块名
LOCAL_MODULE := hello-jni
# 当前模块包含的源代码文件
LOCAL_SRC_FILES := hello-jni.c
# 生成一个动态库
include $(BUILD_SHARED_LIBRARY)
```

MainActivity添加nativeTest()方法

```java
public class MainActivity extends AppCompatActivity {

    {
        System.loadLibrary("hello-jni");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toast.makeText(this, "nativeTest = " + nativeTest(), Toast.LENGTH_SHORT).show();
    }

    public native int nativeTest();
}
```

app/build.gradle文件修改

```gradle
android {
    defaultConfig {
        //指定源文件的编译方式以及配置编译选项
        externalNativeBuild {
            ndkBuild {
                abiFilters "arm64-v8a"
            }
        }
    }

    //配置native的编译脚本路径
    externalNativeBuild {
        ndkBuild {
            path "src/main/ndkBuild/Android.mk"
        }
    }
}
```

#### Shell语法

创建demo.sh文件

```sh
#!/bin/bash
#File by
echo "grandstream"

A=10
echo $A
echo $PWD
echo "##############"
echo $0
echo $1
echo $2
echo "this \$? is $?" # 执行结果
echo "this \$* is $*" # 全部参数
echo "this \$# is $#" # 参数个数
```

执行demo.sh

```shell
chmod 777 demo.sh
./demo.sh #输出helloworld 或者 /bin/bash demo.sh 不需要加权限
```

for循环

```shell
#!/bin/bash
for i in `seq 1 15`
do
	echo "数字 $i"
done

echo "------------"
j=0
for((i=0;i<=100;i++))
do
	j=`expr $i + $j`
done
echo "j = $j"

for i in `find ~/test -name "*.java"`
do
	tar -czf all.tgz $i
done

j=0
while((j<5))
do
	echo "数字 $j"
	j=`expr $j + 1`
done

k=0
while [[ $k -lt 100 ]]
do
	echo "数字 $k"
	k=`expr $k + 1`
done
```

读取文件

```shell
#!/bin/bash
while read line
do
	echo $line
done</home/wlzhou/test/test.txt
```

if和else语句

```shell
#!/bin/bash
num1=100
num2=200
if(($num1>$num2));then
	echo "num1>num2"
else
	echo "num1<num2"
fi

if [ ! -d ~/test/android ]; then
	mkdir -p ~/test/android
else 
	echo "目录已经存在"
fi

PATH=~/test/wlzhou.java

if [ ! -f $PATH ]; then
	touch ~/test/wlzhou.java
else 
	echo "文件已经存在"
	cat $PATH
fi
```

函数

```shell
#!/bin/bash
name="java"
function test() {
	a=1
	echo "woshitest函数"
	echo $a
	echo $name
	echo "1111-----$1"
}

sum=1
function f() {
	for((i=1;i<$1;i++))
	do
		sum=$[$sum * $i]
	done
	echo "$1 = $sum"
}

echo $1

f $1

echo "----------------"

func2() {
	read -p "请输入" num
	echo $[2*num]
}
res=`func2`
echo "func2 return value: $?"
echo "func2 return value: $res"
```

#### FFmpeg

#### CMake

是一个跨平台的支持产出各种不同的构建脚本的一个工具。

#### makefile补充

MakeFile基本语法

```mk
target:depend
	command
```

等价代码

```mk
#calc:
#	g++ add.cpp sub.cpp multi.cpp calc.cpp -o calc

calc:add.o sub.o multi.o
	g++ add.o sub.o multi.o calc.cpp -o calc

add.o:add.cpp
	g++ -c add.cpp -o add.o

sub.o:sub.cpp
	g++ -c sub.cpp -o sub.o

multi.o:multi.cpp
	g++ -c multi.cpp -o multi.o
```

使用变量改造

```mk
#calc:
#	g++ add.cpp sub.cpp multi.cpp calc.cpp -o calc

.PHONY: clean show
TARGET=calc
OBJ=add.o sub.o multi.o calc.o

$(TARGET):$(OBJ)
	$(CXX) $^ -o $@

add.o:add.cpp
	$(CXX) -c $^ -o $@

sub.o:sub.cpp
	$(CXX) -c $^ -o $@

multi.o:multi.cpp
	$(CXX) -c $^ -o $@

calc.o:calc.cpp
	$(CXX) -c $^ -o $@

clean:
	$(RM) *.o $(TARGET)

show:
	echo $(AS)
	echo $(CC)
	echo $(CPP)
	echo $(CXX)
	echo $(RM)
```

更加简洁语法

```mk
.PHONY: clean show
TARGET=calc
OBJ=add.o sub.o multi.o calc.o

$(TARGET):$(OBJ)
	$(CXX) $^ -o $@

%.o:%.cpp
	$(CXX) -c $^ -o $@

clean:
	$(RM) *.o $(TARGET)
```

打印shell命令

```mk
A:=$(shell ls)
```
