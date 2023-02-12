---
title: Activity相关
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

**异常生命周期**

1. 资源相关的系统配置发生改变导致Activity被杀死并重新创建。
2. 资源内存不足导致低优先级的Activity被杀死。

**说说Binder的大体实现**

aidl工具会根据aidl文件里面的接口定义，自动生成java文件，其中就包括客户端代理Proxy和服务端代理Stub，并且自动生成了与Binder驱动通信的代码。

**aidl接口文件是 java 用于跨进程通信的工具，那如果是在同一个进程中 aidl 还能用吗**

可以用于同一个进程中。

**transact 和 onTransact 两个方法有什么关系？**

client 端调用了 transact 方法发起一次跨进程通信的请求，回调 server 端的 onTransact 方法获取返回结果。

**Service中的 onBind 方法需要返回的 IBinder 对象和 ServiceConnection 的回调方法 onServiceConnected 中的 IBinder 有什么关系？**

如果 Service 和 client 是跨进程的，那么此时的 IBinder 对象就是 BinderProxy。如果是在一个进程，那么这个 IBinder 对象就是 Service 中定义的 Stub 对象。

**手写AIDL**

+ [手撸一个 aidl](https://blog.csdn.net/haoxl1994/article/details/103816377)
+ [Android四大组件——Service篇](https://zhuanlan.zhihu.com/p/334657346)

**广播中怎么进行网络请求**

开启Service进行网络请求。

**Android应用保活** https://juejin.cn/post/6924604980057882638

+ 白色保活

1. 用startForeground()启动前台服务，这是官方提供的后台保活方式，不足的就是通知栏会常驻一条通知，像360的状态栏。

+ 灰色保活

1. 开启前台Service，开启另一个Service将通知栏移除，其oom_adj值还是没变的，这样用户就察觉不到app在后台保活。
2. 用广播唤醒自启，像开机广播、网络切换广播等，但在国产Rom中几乎都被堵上了。
3. 多个app关联唤醒，就像BAT的全家桶，打开一个App的时候会启动、唤醒其他App，包括一些第三方推送也是，对于大多数单独app，比较难以实现。

+ 黑色保活

1. 像素activity保活方案，监听息屏事件，在息屏时启动个一像素的activity，提升自身优先级；
2. Service中循环播放一段无声音频，伪装音乐app，播放音乐中的app优先级还是蛮高的，也能很大程度保活效果较好，但耗电量高，谨慎使用；
3. 双进程守护，这在国产rom中几乎没用，因为划掉app会把所有相关进程都杀死。

**Finalize机制**

+ 当GcRoots判断这个对象是否可达，如果不可达被第一次标记。
+ 当没有重写finalize()方法,那么被第二次标记，可以回收这个对象。
+ 当重写了finalize()方法，先吧这个对象放入F-Queue队列中，然后虚拟机创建一个finalizer线程来,来调用finalize()方法，如果这个方法内的对象被GcRoot上的任意一个对象引用了，那么复活一次。当在次调用的时候,就不会在复活了，因为只会调用一次finalize()方法，这个对象就被垃圾回收掉了。

**Android启动优化**

1. 闪屏页将白屏替代
2. 减少方法耗时（初始化SDK、读写存储数据）

**Java中的ClassLoader**

BootStrapClassLoader -> ExtClassLoader -> AppClassLoader -> CustomClassLoader

**Android中的ClassLoader**

BootClassLoader -> PathClassLoader -> DexClassLoader
