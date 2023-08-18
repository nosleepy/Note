---
title: C语言相关
date: 2022-01-01 19:06:13
tags:
categories:
---

#### CMakeLists.txt文件配置

+ c项目

```
cmake_minimum_required(VERSION 3.26)
project(c_project C)

set(CMAKE_C_STANDARD 11)

add_executable(c_project main.c)
```

+ cpp项目

```
cmake_minimum_required(VERSION 3.26)
project(cpp_project)

set(CMAKE_CXX_STANDARD 17)

add_executable(cpp_project main.cpp)
```

#### cpp文件引入c头文件

add.h

```h
#ifndef UNTITLED15_ADD_H
#define UNTITLED15_ADD_H

int add(int a, int b);

#endif //UNTITLED15_ADD_H
```

add.c

```c
int add(int a, int b) {
    return a + b;
}
```

main.cpp

```cpp
#include <iostream>
using namespace std;

extern "C" {
    #include "../util/add.h"
}

int main(){
    cout << "add(1, 2) = " << add(1, 2) << endl;
    return 0;
}
```

+ [Chat-Room](https://github.com/fangsong0517/Chat-Room)
+ [C++ 信号处理](https://www.runoob.com/cplusplus/cpp-signal-handling.html)
+ [C++ 多线程](https://www.runoob.com/cplusplus/cpp-multithreading.html)
