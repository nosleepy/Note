---
title: 源码编译Compose项目
date: 2023-11-15 00:00:00
tags:
categories:
- 安卓
---

使用 AndroidStudio 新建一个 Compose 工程, Name 为 MyCompose

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_project.png)

AndroidManifest.xml 文件中添加包名

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.gs.mycompose">

</manifest>
```

将 MyCompose 工程拷贝到 /media/wlzhou/bak/gsc3575/vendor/grandstream/packages/apps/ 目录下

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/mycompose_position.png)

参考 SystemUI 的编译在 MyCompose 目录下新建一个 Android.bp 文件

```makefile
android_app {
    name: "MyCompose",

    srcs: ["app/src/**/*.kt"],

    manifest: "app/src/main/AndroidManifest.xml",

    resource_dirs: ["app/src/main/res"],

    certificate: "platform",

    platform_apis: true,

    static_libs: [
        "androidx.compose.runtime_runtime",
        "androidx.core_core-ktx",
        "androidx.lifecycle_lifecycle-runtime-ktx",
        "androidx.activity_activity-compose",
        "androidx.compose.ui_ui",
        "androidx.compose.material3_material3",
    ],

    kotlincflags: ["-Xjvm-default=enable"],

    dxflags: ["--multi-dex"],
}
```

源码目录下使用 mm 命令开始编译

```sh
/media/wlzhou/bak/gsc3575/vendor/grandstream/packages/apps/MyCompose$ mm
```

编译完成后会在 out 目录生成一个 apk 文件

```shell
[15/15] Install out/target/product/gsc3575/system/app/MyCompose/MyCompose.apk

#### build completed successfully (01:28 (mm:ss)) ####
```

使用 adb install 安装 apk

```shell
adb install MyCompose.apk
```
