---
title: ffmpeg编解码学习
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### ffmpeg命令

官网下载ffmpeg源代码 http://www.ffmpeg.org/releases/

转码命令

```shell
ffmpeg -i input.avi -b:v 640k output.ts
```

播放命令

```shell
ffplay input.avi
```

#### ffmpeg库简介

+ avcodec：编解码
+ avformat：封装格式处理
+ avfilter：滤镜特效处理
+ avdevice：各种设备的输入输出
+ avutil：工具库
+ postproc：后加工
+ swresample：音频采样数据格式转换
+ swscale：视频像素数据格式转换

#### 参考

+ [[Ubuntu]编译 安装 FFmpeg](https://zhuanlan.zhihu.com/p/80895966)
+ [ubuntu21.04 安装ffmpeg库并在Clion中编写cpp使用ffmpeg](https://blog.csdn.net/qq_45953454/article/details/128509757)
+ [Linux下编译安装SDL2](https://cloud.tencent.com/developer/article/1932877)
+ [C/C++静态库与动态库（Clion下创建及调用静态库）](https://blog.csdn.net/Phantom_matter/article/details/121082906)
+ [FFmpeg_Android_Demo](https://github.com/weekend-y/FFmpeg_Android_Demo)
+ [FFmpeg视频解码流程详解及demo](https://blog.csdn.net/weekend_y45/article/details/125168344)
+ [Android NDK学习--jni访问java层方法](https://www.jianshu.com/p/4d7124503dd0)
+ [Android FFmpeg 编译.so库](https://blog.csdn.net/JohanMan/article/details/81565834)