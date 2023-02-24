---
title: 安卓ndk开发记录
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### 开发过程中的问题

**android ndk abiFilters 和 cmake abiFilters 的区别**

```gradle

android {
    defaultConfig {
        applicationId "com.grandstream.myapplication"
        minSdk 21
        targetSdk 33
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                abiFilters 'arm64-v8a', 'armeabi-v7a', 'x86', 'x86_64'
            }
        }
        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a', 'x86', 'x86_64'
        }
    }
 }
```

cmake 中 //指定编译 c、[c++] 代码时生成哪种类型的 so 库
ndk 中 //指定打包 apk 时只将哪种类型的 so 库打包进 apk 里面

**Android Studio 如何调用 so 库**

+ JNI规范so库调用

使用 native-lib.cpp 对命名进行规范化，通过相同包名类名进行方法调用

Android Studio 新建 Native C++ 项目，新增 MyUtils.java 类

```java
package com.grandstream.utils;

public class MyUtils {
    public static native int add(int a, int b);
}
```

native-lib.cpp 实现该 native 方法

```cpp
#include <jni.h>
#include <string>

extern "C"
JNIEXPORT jint JNICALL
Java_com_grandstream_utils_MyUtils_add(JNIEnv *env, jobject thiz, jint a, jint b) {
    return a + b;
}
```

编译项目，复制 arm64-v8a 目录下生成的 libmyutils.so 库

Android Studio 新建 Empty Activity 项目，同样创建 com.grandstream.utils 包，新增 MyUtils.java 类；拷贝 so 库文件到 app/src/main/jniLibs/arm64-v8a 目录下，无需配置 build.gradle

```java
package com.grandstream.utils;

public class MyUtils {
    static {
        System.loadLibrary("myutils");
    }

    public static native int add(int a, int b);
}
```

Activity 中直接调用 MyUtils.add(1, 2) 即可

也可以将 so 库文件放到 app/libs 目录下，build.gradle 增加配置

```gradle
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

+ 非JNI规范的so库调用

测试没有使用 native-lib.cpp 对命名进行规范化，如何直接调用 cpp 中的方法

Android Studio 新建 Native C++ 项目，创建 mytool.h、mytool.cpp 文件

```h
#include "mytool.h"
#include <iostream>

std::string getName() {
    return "1024";
}
```

```cpp
#include "tool.h"
#include <iostream>

std::string getName() {
    return "1024";
}
```

CMakeLists.txt 修改配置

```txt
cmake_minimum_required(VERSION 3.18.1)

project("mytool")

add_library(
        mytool
        SHARED
        native-lib.cpp
        mytool.h #add
        mytool.cpp #add
        )

find_library(
        log-lib
        log)

target_link_libraries(
        mytool
        ${log-lib})
```

编译后 arm64-v8a 目录生成 libmytool.so 动态库

创建另外一个 Native C++ 工程，在这个工程中调用非 JNI 规范的 so 库

app/libs 目录新建 arm64-v8a 目录，存放 so 库文件，新建 includes 目录存放头文件

拷贝上个工程生成的 libmytool.so 库和头文件 mytool.h

新建 MyUtils.java 文件用于调用方法

```java
package com.grandstream.utils;

public class MyUtils {
    public static native String tip();
}
```

CMakeLists.txt 文件加入如下配置

```txt
cmake_minimum_required(VERSION 3.18.1)

project("sotest")

add_library(
        sotest
        SHARED
        native-lib.cpp)

find_library(
        log-lib
        log)

include_directories(${CMAKE_SOURCE_DIR}/../../../libs/includes)
set(DIR ${CMAKE_SOURCE_DIR}/../../../libs/arm64-v8a)
add_library(mytool
        SHARED
        IMPORTED)
set_target_properties(mytool
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/libmytool.so)

target_link_libraries(
        sotest
        mytool
        ${log-lib})
```

build.gradle 修改配置

```gradle
android {
    namespace 'com.grandstream.testtooltool'
    compileSdk 33

    defaultConfig {
        applicationId "com.grandstream.testtooltool"
        minSdk 21
        targetSdk 33
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                abiFilters 'arm64-v8a' #只编译arm64-v8a类型的so库
            }
        }
    }
}
```

native-lib.cpp 方法中引入 mytool.h 头文件，调用 getName() 方法

```cpp
#include <jni.h>
#include <string>
#include "mytool.h"

extern "C"
JNIEXPORT jstring JNICALL
Java_com_grandstream_utils_MyUtils_tip(JNIEnv *env, jobject thiz) {
    return env->NewStringUTF(getName().c_str()); //调用c++实现的getName()方法
}
```

Activity 中加载 so 库，调用 MyUtils.tip() 方法显示信息

```java
public class MainActivity extends AppCompatActivity {
    static {
        System.loadLibrary("sotest");
    }

    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        TextView tv = binding.sampleText;
        tv.setText(MyUtils.tip());
    }
}
```

运行工程，TextView 上显示 mytool.cpp 实现中返回的 "1024"

复制编译生成的 libmytool.so 和 libsotest.so 库，就可以完成规范 JNI 调用 so 动态库了

```java
public class MyUtils {
    static {
    	//需要加载两个so库,sotest依赖于mytool
        System.loadLibrary("sotest");
        System.loadLibrary("mytool");
    }

    public static native String tip();
}
```

#### Clion编译静态库和动态库

Clion 新建 C++ Library 工程，Library type 选择 shared 动态库

新建 tool.h、tool.cpp

```h
#ifndef UNTITLED_TOOL_H
#define UNTITLED_TOOL_H

#include <iostream>

std::string getName();

#endif //UNTITLED_TOOL_H
```

```cpp
#include "tool.h"
#include <iostream>

std::string getName() {
    return "1024";
}
```

CMakeLists.txt 配置如下

```txt
cmake_minimum_required(VERSION 3.24)
project(tool)

set(CMAKE_CXX_STANDARD 17)

add_library(tool SHARED library.cpp tool.cpp tool.h)
```

编译工程得到 libtool.so 动态库

去掉 `add_library(tool SHARED library.cpp tool.cpp tool.h)` 中的 SHARED 编译可以得到 libtool.a 静态库