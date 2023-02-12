---
title: Redis面试题
date: 2020-07-14 22:43:28
tags:
categories:
- Java面试
---

**为什么一定要有redis？（[知乎](https://zhuanlan.zhihu.com/p/59168140)）**

**redis常用命令**

**基本的数据结构有哪些**

+ string（字符串）
+ hash（哈希）
+ list（列表）
+ set（集合）
+ zset（有序集合）

**redis为什么这么快**

纯内存操作、单线程操作，避免了频繁的上下文切换、采用了非阻塞I/O多路复用机制。