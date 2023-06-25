---
title: RxJava2使用
date: 2023-06-25 00:00:00
tags:
- rxjava
categories:
- 安卓
---

#### 简介

什么是rxjava？ 是一种事件驱动的基于异步数据流的编程模式，整个数据流就像一条河流，它可以被观测（监听），过滤，操控或者与其他数据流合并为一条新的数据流。

#### 三要素

+ 被观察者（Observable）
+ 观察者（Observer）
+ 订阅（subscribe）

#### 简单使用

1. 添加依赖

```groovy
implementation 'io.reactivex.rxjava2:rxjava:2.1.4'
implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
```

2. 创建被观察者

```java
Observable observable =  Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("msg1");
        emitter.onNext("msg2");
        emitter.onNext("msg3");
        emitter.onComplete();
    }
});
```

3. 创建观察者

```java
Observer observer = new Observer<String>(){
    @Override
    public void onSubscribe(Disposable d) {
        Log.i("wlzhou", "subscribe");
    }

    @Override
    public void onNext(String s) {
        Log.i("wlzhou", s);
    }

    @Override
    public void onError(Throwable e) {
        Log.i("wlzhou", "error");
    }

    @Override
    public void onComplete() {
        Log.i("wlzhou", "complete");
    }
};
```

4. 订阅

```java
 observable.subscribe(observer);
```

运行结果

```
2023-06-25 14:02:17.824  6039-6039  wlzhou                  com.example.myapplication            I  subscribe
2023-06-25 14:02:17.825  6039-6039  wlzhou                  com.example.myapplication            I  msg1
2023-06-25 14:02:17.825  6039-6039  wlzhou                  com.example.myapplication            I  msg2
2023-06-25 14:02:17.825  6039-6039  wlzhou                  com.example.myapplication            I  msg3
2023-06-25 14:02:17.825  6039-6039  wlzhou                  com.example.myapplication            I  complete
```

#### 操作符

+ 转换操作符
+ 组合操作符
+ 功能操作符
+ 过滤操作符
+ 条件操作符

#### 参考

+ [史上最全的Rxjava2讲解（使用篇）](https://juejin.cn/post/6844903957496643597)