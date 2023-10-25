---
title: Android.mk和Android.bp
date: 2023-10-25 00:00:00
tags:
categories:
- 安卓
---

MyApp 项目下的 Android.mk 文件

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

使用 androidmk 命令将 mk 文件转换成 bp 文件

```shell
androidmk Android.mk > Android.bp
```

生成的 Android.bp 文件 

```makefile
android_app {
    name: "MyApp",

    srcs: ["app/src/main/java/**/*.java"],

    manifest: "app/src/main/AndroidManifest.xml",

    resource_dirs: ["app/src/main/res"],

    certificate: "platform",

    sdk_version: "current",

    dex_preopt: {
        enabled: false,
    },

    static_libs: [
        "androidx.appcompat_appcompat",
        "com.google.android.material_material",
        "androidx-constraintlayout_constraintlayout",
    ],

}
```

参考

+ [Android 基础 | Android.mk 语法浅析](https://juejin.cn/post/6844904127395282951?searchId=202310251156325A0DEBA430445AA24468)
+ [Android.bp学习笔记](https://juejin.cn/post/6844904089659113486?searchId=20231025151537B84BE043BE3A46B0207D)