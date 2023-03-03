---
title: 安卓framework修改
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### FrameWork修改

+ 修改 /android11/frameworks/base 下的代码

```java
// android11/framework/base/core/java/android/app/Activity.java

@MainThread
    @CallSuper
    protected void onCreate(@Nullable Bundle savedInstanceState) {
       	//onCreate方法最后加上一行日志
        android.util.Log.i("wlzhou", "onCreate modify");
    }
```

+ 编译生成 framework.jar

```shell
make -j4 framework-minus-apex

[100% 35/35] Install: out/target/product/generic_x86/system/framework/framework.jar

#### build completed successfully (05:41 (mm:ss)) ####
```

+ 使用 make snod 重新生成 system.img 镜像

```shell
make snod

[100% 7/7] make snod: ignoring dependencies
Target system fs image: out/target/product/generic_x86/system.img

#### build completed successfully (04:19 (mm:ss)) ####
```

+ emulator 命令启动模拟器测试

```shell
emulator -writable-system #加上参数,不然没权限
adb root
adb remount
adb push out/target/product/generic_x86/system/framework/framework.jar /system/framework/
adb remount
```

启动任何一个app,可以在logcat中看到新增的日志信息

#### ninja加速编译

使用 mm 命令或者 make 命令编译过模块之后，可以使用 ninja 工具加快模块编译

```shell
./prebuilts/build-tools/linux-x86/bin/ninja -f out/combined-aosp_x86.ninja framework-minus-apex # 编译framework
./prebuilts/build-tools/linux-x86/bin/ninja -f out/combined-aosp_x86.ninja SystemUI # 编译SystemUI
```

增加删除文件后需要重新执行 mm 或者 make 命令编译一次

#### 其他

+ /android11/frameworks/base/core/res 有内容修改

```shell
./prebuilts/build-tools/linux-x86/bin/ninja -f out/combined-aosp_x86.ninja framework-res # 编译生成 framework-res.apk,push到/system/framework
```

+ /android11/frameworks/base/services 有内容修改

```shell
./prebuilts/build-tools/linux-x86/bin/ninja -f out/combined-aosp_x86.ninja services # 编译生成 services.jar,push到/system/framework
```

+ /android11/frameworks/base/ 有内容修改

```shell
./prebuilts/build-tools/linux-x86/bin/ninja -f out/combined-aosp_x86.ninja framework-minus-apex # 编译生成 framework.jar,push到/system/framework
```

#### 参考

+ [ANDROID Q单编替换framework.jar方法](https://blog.csdn.net/bestwu0666/article/details/123005985)
+ [Android 12 编译framewok和services详解](https://blog.csdn.net/u010345983/article/details/126779697)
+ [Android Studio自带的模拟器无法root解决方法](https://www.jianshu.com/p/d8231fc25a3d)
+ [解决Android Studio ADV模拟器无法使用remount命令记录](https://blog.csdn.net/qq85913323/article/details/125494934)
+ [解决安卓模拟器系统中已经是root用户，mount仍然报错：Permission denied](https://www.codeleading.com/article/95173368340/)
+ [AOSP Android 10.0单编替换framework.jar刷入手机](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1719606)