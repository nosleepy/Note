---
title: android沉浸式状态栏
date: 2023-04-26 19:06:13
tags:
categories:
- 安卓
---

#### 未适配前

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/adapter_before.png)

布局文件内容

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:text="1111"
        android:textSize="22sp"
        android:background="@color/purple_500"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:text="2222"
        android:textSize="22sp"
        android:background="@color/teal_200"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

#### 安卓4.4以上

1. 资源文件中设置

自定义主题

```xml
    <style name="CustomTheme" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:windowTranslucentStatus">true</item>
    </style>
```

配置主题

```xml
<activity
            android:theme="@style/CustomTheme"
            android:name=".MainActivity"
            android:exported="true">
</activity>
```

2. 代码中设置

```kotlin
window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/adapter_4.4_step1.png)

发现状态栏为半透明的颜色, TextView 有占用状态栏的情况,设置 android:fitsSystemWindows 属性为 true

```
<TextView
    android:text="1111"
    android:textSize="22sp"
    android:fitsSystemWindows="true"
    android:background="@color/purple_500"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/adapter_4.4_step2.png)

属性的作用是给 view 添加一个值为状态栏高度的顶部 padding

#### 安卓5.0以上

可以直接设置状态栏的颜色

```kotlin
window.statusBarColor = Color.TRANSPARENT
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/adapter_5.0.png)

#### 安卓6.0以上

设置状态栏字体颜色为黑色,默认为白色,透明状态栏会看不到

```kotlin
window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/adapter_6.0.png)

#### 其他属性

1. 设置背景图片

```xml
    <style name="CustomTheme" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:windowBackground">@drawable/bg</item>
    </style>
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/background_picture.png)

2. 设置背景透明

```xml
    <style name="CustomTheme" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:windowShowWallpaper">true</item>
    </style>
```

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/background_transparent.png)

#### 参考

+ [随手记Android沉浸式状态栏的踩坑之路](https://juejin.cn/post/6844903518982111245)