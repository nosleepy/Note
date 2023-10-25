---
title: 使用源码编译AndroidStudio工程
date: 2023-10-25 00:00:00
tags:
categories:
- 安卓
---

使用 AndroidStudio 新建一个 MyApp 工程

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/new_myapp.png)

复制 MyApp 工程到安卓源码的 packages/apps 目录下

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/myapp_position.png)

MyApp 根目录创建 Android.mk 文件

```makefile
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := $(call all-java-files-under, app/src/main/java)

LOCAL_MANIFEST_FILE := app/src/main/AndroidManifest.xml

LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/app/src/main/res

LOCAL_PACKAGE_NAME := MyApp

LOCAL_CERTIFICATE := platform

LOCAL_SDK_VERSION := current

LOCAL_DEX_PREOPT := false

LOCAL_STATIC_ANDROID_LIBRARIES := \
androidx.appcompat_appcompat \
com.google.android.material_material \
androidx-constraintlayout_constraintlayout

include $(BUILD_PACKAGE)
```

当前项目目录结构, 此时 MyApp 工程同时支持源码编译和 AndroidStudio 编译

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/myapp_project.png)

源码 MyApp 目录下执行 mm 命令开始编译

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/media/wlzhou/bak/GXV33X5/vendor/grandstream/packages/apps/MyApp$ mm
```

+ 错误1

```
/media/wlzhou/bak/GXV33X5/out/target/common/obj/APPS/MyApp_intermediates/manifest/AndroidManifest.xml.fixed Error:
	Main AndroidManifest.xml at AndroidManifest.xml.fixed manifest:package attribute is not declared
```

AndroidManifest.xml 文件中需要添加 packge 属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:sharedUserId="android.uid.system"
    package="com.gs.myapp">

</manifest>
```

+ 错误2

```
error: resource style/Theme.Material3.DayNight.NoActionBar (aka com.gs.myapp:style/Theme.Material3.DayNight.NoActionBar) not found.
error: resource style/Theme.Material3.DayNight.NoActionBar (aka com.gs.myapp:style/Theme.Material3.DayNight.NoActionBar) not found.
error: failed linking references.
```

主题资源找不到, Theme.Material3.DayNight.NoActionBar 主题在 com.google.android.material_material 库中, 修改 themes.xml 文件中的主题

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <style name="Base.Theme.MyApp" parent="android:Theme.Light">
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowActionBar">false</item>
    </style>

    <style name="Theme.MyApp" parent="Base.Theme.MyApp" />
</resources>
```

+ 错误3

```
out/target/common/obj/APPS/MyApp_intermediates/manifest/AndroidManifest.xml:10: error: attribute android:dataExtractionRules not found.
error: failed processing manifest.
```

直接删除 android:dataExtractionRules 这个属性

**编译成功**

```
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/media/wlzhou/bak/GXV33X5/vendor/grandstream/packages/apps/MyApp$ mm
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=11
TARGET_PRODUCT=fox8
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-a
TARGET_CPU_VARIANT=cortex-a55
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv8-2a
TARGET_2ND_CPU_VARIANT=cortex-a55
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.4.0-137-generic-x86_64-Ubuntu-18.04.6-LTS
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=RQ3A.210705.001
OUT_DIR=out
============================================
11:14:21 AllowBuildBrokenUsesNetwork: true
11:14:21 BuildBrokenUsesNetwork: true
11:14:21 AllowBuildBrokenUsesNetwork: true
11:14:21 BuildBrokenUsesNetwork: true
11:14:22 AllowBuildBrokenUsesNetwork: true
11:14:22 BuildBrokenUsesNetwork: true
11:14:22 AllowBuildBrokenUsesNetwork: true
11:14:22 BuildBrokenUsesNetwork: true
[100% 11/11] Install: out/target/product/fox8/system/app/MyApp/MyApp.apk

#### build completed successfully (22 seconds) ####

Start looking for installable targets...
/media/wlzhou/bak/GXV33X5/out/target/product/fox8/system/app/MyApp/MyApp.apk
```

使用 adb install

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/media/wlzhou/bak/GXV33X5/vendor/grandstream/packages/apps/MyApp$ adb install /media/wlzhou/bak/GXV33X5/out/target/product/fox8/system/app/MyApp/MyApp.apk
Performing Streamed Install
Success
```

MyApp 应用已成功安装, 运行闪退报错

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.gs.myapp/com.gs.myapp.MainActivity}: java.lang.IllegalStateException: You need to use a Theme.AppCompat theme (or descendant) with this activity.
```

MainActivity 继承 AppCompatActivity, 但没有使用 Theme.AppCompat 主题, 修改 MainActivity 继承 Activity

```java
package com.gs.myapp;

import android.app.Activity;
import android.os.Bundle;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

再次 mm 编译, abd install 安装运行

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/myapp_page.png)

通过 adb shell 查看, com.gs.myapp 已为系统 app

```
云视讯D33Plus:/ $ ps -ef | grep gs
root            181      1 0 17:49:25 ?     00:00:00 gs-dbus-daemon --system --nofork
system          437      1 0 17:49:28 ?     00:00:06 gslogd
system          775    280 0 17:49:38 ?     00:00:03 com.gs.service
system          965    280 2 17:49:39 ?     00:26:00 com.gs.service.phone
system         1106    280 0 17:49:39 ?     00:00:01 com.android.settings
system         1156    280 0 17:49:39 ?     00:00:00 com.android.launcher3gsrecents
root           1396   1357 0 17:49:43 ?     00:00:01 gs_lwM2M
root           1431      1 30 17:49:43 ?    05:21:34 gs_avs
system         1515   1340 0 17:49:45 ?     00:00:16 gs_cpe
root           1518   1343 0 17:49:45 ?     00:00:07 gs_cpe_54
system         1572    280 0 17:49:45 ?     00:00:00 com.gs.together.zoom
system         1649    280 0 17:49:45 ?     00:00:00 com.gs.sink.service
system         2486    280 0 17:49:51 ?     00:00:00 com.base.module.gsaffinityservice
system         2690    280 4 11:28:29 ?     00:00:00 com.gs.myapp
system         2761    280 0 17:49:53 ?     00:00:01 com.gs.airplay.service
shell          2762   2742 2 11:28:35 pts/2 00:00:00 grep gs
system         2812    280 0 17:49:54 ?     00:00:00 com.gs.enterprise.contacts
system         2944    280 0 17:50:02 ?     00:00:00 com.gs.snmp4j
system         2968    280 0 17:50:03 ?     00:00:00 com.gs.ui
```

**参考**

+ [2022-02-22 Android 源码里面添加一个系统app，最简单的app demo实例，附加源码](https://blog.csdn.net/qq_37858386/article/details/123062639)
+ [Android源码中添加APP](https://www.cnblogs.com/CoderTian/p/5890942.html)
+ [Android Framework开发之新加一个app源码到packages/apps编译Android.mk配置](https://juejin.cn/post/7238446447175565367?searchId=202310241127220EF3723767C4B5CEE617)
+ [Android Studio项目放到源码编译](https://www.jianshu.com/p/3cce39f25312)
+ [Android查看源码编译中存在的库](https://blog.csdn.net/wenzhi20102321/article/details/122889502)
+ [Android 编译过程介绍，Android.mk 和 Android.bp 分析， 在源码中编译 AndroidStudio 构建的 App](https://blog.csdn.net/qq_43880417/article/details/128311478)