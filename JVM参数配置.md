---
title: JVM参数配置
date: 2020-04-24 16:00:29
tags:
categories:
- Java虚拟机
---

## Trace跟踪参数

1、打印GC的简要信息

```
-verbose:gc
-XX:+printGC
```

解释：可以打印GC的简要信息。

2、打印GC的详细信息

```
-XX:+PrintGCDetails
```
解释：打印GC详细信息。

```
-XX:+PrintGCTimeStamps
```
解释：打印GC发生的时间戳。

3、指定GC log的位置：

```
-Xloggc:log/gc.log
```
解释：指定GC log的位置，以文件输出。帮助开发人员分析问题。

```
-XX:+PrintHeapAtGC
```
解释：每一次GC前和GC后，都打印堆信息。

```
-XX:+TraceClassLoading
```
解释：监控类的加载。

```
-XX:+PrintClassHistogram
```
解释：按下Ctrl+Break后，打印类的信息。

## 堆的分配参数

1、-Xmx –Xms：指定最大堆和最小堆

```
-Xmx20m -Xms5m
```
最大堆内存大小为20m，最小堆内存大小为5m

2、-Xmn、-XX:NewRatio、-XX:SurvivorRatio：

```
-Xmn（新生代）
```
设置新生代大小

```
-XX:NewRatio
```
设置新生代（eden+2*s）和老年代（不包含永久区）的比值

例如：4，表示新生代:老年代=1:4，即新生代占整个堆的1/5

```
-XX:SurvivorRatio（幸存代）
```
设置两个Survivor区和eden的比值

例如：8，表示两个Survivor:eden=2:8，即一个Survivor占年轻代的1/10

3、-XX:+HeapDumpOnOutOfMemoryError、-XX:+HeapDumpPath

-XX:+HeapDumpOnOutOfMemoryError

```
-XX:+HeapDumpOnOutOfMemoryError
```
OOM时导出堆到文件

根据这个文件，我们可以看到系统dump时发生了什么。

```
-XX:+HeapDumpPath
```

导出OOM的路径

4、-XX:OnOutOfMemoryError：

```
-XX:OnOutOfMemoryError
```
在OOM时，执行一个脚本。可以在OOM时，发送邮件，甚至是重启程序。

5、堆的分配参数总结：

+ 根据实际事情调整新生代和幸存代的大小
+ 官方推荐新生代占堆的3/8
+ 幸存代占新生代的1/10
+ 在OOM时，记得Dump出堆，确保可以排查现场问题

6、永久区分配参数：

```
-XX:PermSize  -XX:MaxPermSize
```
设置永久区的初始空间和最大空间。

## 栈的分配参数

```
-Xss
```
设置栈空间的大小。

## 参考

+ [JVM学习八：常用JVM配置参数](https://www.cnblogs.com/pony1223/p/8661219.html)