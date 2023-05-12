---
title: android混淆相关
date: 2023-05-12 00:00:00
tags:
categories:
- 安卓
---

#### 开启混淆

app/build.gradle 文件配置

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/app_proguard_project.png)

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true //开启混淆
            shrinkResources true //无用资源去除
            zipAlignEnabled true //字节码优化
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

其中 proguard-android-optimize.txt 文件是 android 中默认的混淆文件，位于 /Android/Sdk/tools/proguard 目录

```shell
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/Android/Sdk/tools/proguard$ ls
README  ant  bin  docs  examples  lib  license.html  proguard-android-optimize.txt  proguard-android.txt  proguard-project.txt
```

android 默认混淆文件内容

```
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html
#
# This file is no longer maintained and is not used by new (2.2+) versions of the
# Android plugin for Gradle. Instead, the Android plugin for Gradle generates the
# default rules at build time and stores them in the build directory.

# Optimizations: If you don't want to optimize, use the
# proguard-android.txt configuration file instead of this one, which
# turns off the optimization flags.  Adding optimization introduces
# certain risks, since for example not all optimizations performed by
# ProGuard works on all versions of Dalvik.  The following flags turn
# off various optimizations known to have issues, but the list may not
# be complete or up to date. (The "arithmetic" optimization can be
# used if you are only targeting Android 2.0 or later.)  Make sure you
# test thoroughly if you go this route.
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
-dontpreverify

# The remainder of this file is identical to the non-optimized version
# of the Proguard configuration file (except that the other file has
# flags to turn off optimization).

-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose

-keepattributes *Annotation*
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
-keepclasseswithmembernames class * {
    native <methods>;
}

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

-keepclassmembers class **.R$* {
    public static <fields>;
}

# The support library contains references to newer platform versions.
# Don't warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.
-dontwarn android.support.**

# Understand the @Keep support annotation.
-keep class android.support.annotation.Keep

-keep @android.support.annotation.Keep class * {*;}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
```

proguard-rules.pro 文件是 app 模块自定义混淆规则的文件，可以使用基本的混淆模板

```
# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 指定不去忽略非公共库的成员
-dontskipnonpubliclibraryclassmembers
# 混淆时不做预校验
-dontpreverify
# 忽略警告
-ignorewarnings
# 保留代码行号，方便异常信息的追踪
-keepattributes SourceFile,LineNumberTable
# appcompat库不做混淆
-keep class androidx.appcompat.**
#保留 AndroidManifest.xml 文件：防止删除 AndroidManifest.xml 文件中定义的组件
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View

#保留序列化，例如 Serializable 接口
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

#带有Context、View、AttributeSet类型参数的初始化方法
-keepclasseswithmembers class * {
    public <init>(android.content.Context);
}
-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
}
-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet, int);
}
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

#保留资源R类
-keep class **.R$* {*;}

#避免回调函数 onXXEvent 混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
    void *(**on*Changed);
}

#业务实体不做混淆，避免gson解析错误
-dontwarn com.grandstream.convergentconference.entity.**
-keep class com.grandstream.convergentconference.entity.** { *;}

#Rxjava、RxAndroid，官方ReadMe文档中说明无需特殊配置
-dontwarn java.util.concurrent.Flow*
#okhttp3、okio、retrofit，jar包中已包含相关proguard规则，无需配置
#其他一些配置
```

#### 移除material库

androidstudio 创建项目默认会添加 material 依赖

```xml
dependencies {
    implementation 'com.google.android.material:material:1.8.0'
}
```

默认配置 material 相关的主题

```xml
<style name="Theme.MyApplication" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
	<item name="colorPrimary">@color/purple_500</item>
</style>
```

开启混淆配置前 apk 的大小为 1.9 MB

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/not_use_material.png)

项目中如果没有用到 material 组件可以直接移除，同时修改 material 主题为 appcompat 主题

```xml
<style name="Theme.MyApplication" parent="Theme.AppCompat.DayNight.DarkActionBar">
	<item name="colorPrimary">@color/purple_500</item>
</style>
```

经测试优化后的 apk 体积为 785.5 KB

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/use_material.png)

#### 混淆堆栈还原

新建 MyUtil.java 文件模拟堆栈异常

```java
public class MyUtil {
    public void method1() {
        Log.d("wlzhou", "---------method1----------");
        method2();
    }

    public void method2() {
        Log.d("wlzhou", "---------method2----------");
        method3();
    }

    public void method3() {
        Log.d("wlzhou", "---------method3----------");
        exceptionMethod();
    }

    public void exceptionMethod() {
        int res = 10 / 0;
        Log.d("wlzhou", "res = " + res);
    }
}
```

MainActivity 中调用 MyUtil#method1 方法

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new MyUtil().method1();
    }
}
```

未混淆前 logcat 日志信息，很容易定位异常位置

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/not_proguard_exception_info.png)

混淆后 logcat 日志信息，只能看到异常发生在 MainActivity#onCreate 方法中

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/use_proguard_exception_info.png)

使用 android 自带的混淆堆栈还原工具 proguardgui 解决

```shell
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/Android/Sdk/tools/proguard/bin$ ls
proguard.sh  proguardgui.sh  retrace.sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/Android/Sdk/tools/proguard/bin$ ./proguardgui.sh
```

脚本运行后会打开 ProGuard 调试界面

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/proguard_debug.png)

填写混淆前后的映射文件 mapping.txt 和需要还原的 logcta 异常日志，点击 Retrace 按钮就能还原混淆前的异常信息

#### 参考

+ [Android混淆配置（含androidx、kotlin）](https://blog.csdn.net/weixin_42602900/article/details/127628442)
+ [想提高应用程序的用户满意度——APK体积包优化少不了](https://zhuanlan.zhihu.com/p/625071359)
+ [安卓编译混淆与堆栈还原](https://juejin.cn/post/7053323596101320735)