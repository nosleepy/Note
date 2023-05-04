---
title: Glide学习
date: 2023-04-28 19:06:13
tags:
categories:
- 安卓
---

#### 介绍

Glide 是一个快速高效的 Android 图片加载库，可以自动加载网络、本地文件，app 资源中的图片，注重于平滑的滚动。

#### 基本使用

添加依赖

```groovy
dependencies {
    implementation 'com.github.bumptech.glide:glide:4.12.0'
}
```

添加网络权限

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/iv"
        android:layout_width="128dp"
        android:layout_height="128dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

使用方法

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ImageView iv = findViewById(R.id.iv);
        Glide.with(this).load("https://raw.githubusercontent.com/nosleepy/picture/master/img/fruit/apple.png").into(iv);
    }
}
```

结果截图

![](https://raw.githubusercontent.com/nosleepy/picture/main/img/glide_load_apple.png)

#### 手写实现

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/glide_process.png)

+ [手写Glide框架-csdn](https://blog.csdn.net/tiangaopan/article/details/105316596)
+ [手写Glide框架-github](https://github.com/tianyalu/NeGlide2)
+ [Android 基于 Bitmap 的图片压缩方式探究](https://juejin.cn/post/6959840823261265957)
+ [Android进阶宝典 -- 学会Bitmap内存管理，你的App内存还会暴增吗？](https://blog.csdn.net/m0_71506521/article/details/130081589)
