---
title: Android系统启动
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

![Android系统启动流程](https://raw.githubusercontent.com/nosleepy/picture/master/img/android_system_startup_process.png)

**启动流程**

1. 启动电源以及引导程序加载
2. 引导程序BootLoader启动
3. Linux内核启动
4. init进程启动
5. Zygote进程启动
6. SystemServer进程启动
7. Launcher启动

**参考**

+ [Android 系统启动流程](https://github.com/yoyiyi/SoleilNotes/blob/master/Android/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8.md)
+ [Android源码分析--Android系统启动](https://juejin.cn/post/6844903781457461261)
