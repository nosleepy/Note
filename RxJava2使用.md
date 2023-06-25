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

#### 手写RxJava

上游

```java
public class Observable<T> {
    private ObservableOnSubscribe subscribe;

    public Observable(ObservableOnSubscribe subscribe) {
        this.subscribe = subscribe;
    }

    public static <T> Observable<T> create(ObservableOnSubscribe<T> subscribe) {
        return new Observable<T>(subscribe);
    }

    public static <T> Observable<T> just(final T t) {
        return new Observable<T>(new ObservableOnSubscribe<T>() {
            @Override
            public void subscribe(Observer<T> observer) {
                observer.onNext(t);
                observer.onComplete();
            }
        });
    }

    public static <T> Observable<T> just(final T... t) {
        return new Observable<T>(new ObservableOnSubscribe<T>() {
            @Override
            public void subscribe(Observer<T> observer) {
                for (T t1 : t) {
                    observer.onNext(t1);
                }
                observer.onComplete();
            }
        });
    }

    public <R> Observable<R> map(Function<T, R> function) {
        ObservableMap<T, R> map = new ObservableMap(subscribe, function);
        return new Observable<R>(map);
    }

    // 切换IO线程
    public Observable<T> subscribeOnIO() {
        return create(new ObservableIO<T>(subscribe));
    }

    // 切换main线程
    public Observable<T> subscribeOnMain() {
        return create(new ObservableMain(subscribe));
    }

    // 订阅
    public void subscribe(Observer<T> observer) {
        observer.onSubscribe();
        subscribe.subscribe(observer);
    }
}
```

ObservableOnSubscribe

```java
public interface ObservableOnSubscribe<T> {
    void subscribe(Observer<T> observer);
}
```

下游（接口）

```java
public interface Observer<T> {
    void onSubscribe();
    void onNext(T t);
    void onError(Throwable e);
    void onComplete();
}
```

Function

```java
public interface Function<T, R> {
    R apply(T t);
}
```

map操作

```java
public class ObservableMap<T, R> implements ObservableOnSubscribe<R> {
    private ObservableOnSubscribe<T> subscribe;
    private Function<T, R> function;
    private Observer<R> observer;

    public ObservableMap(ObservableOnSubscribe subscribe, Function<T, R> function) {
        this.subscribe = subscribe;
        this.function = function;
    }

    @Override
    public void subscribe(Observer<R> observer) {
        this.observer = observer;
        subscribe.subscribe(new MapPackage(observer, function));
    }

    class MapPackage<T> implements Observer<T> {
        private Observer<R> observer;
        private Function<T, R> function;

        public MapPackage(Observer<R> observer, Function<T, R> function) {
            this.observer = observer;
            this.function = function;
        }

        @Override
        public void onSubscribe() {
            observer.onSubscribe();
        }

        @Override
        public void onNext(T t) {
            R next = function.apply(t);
            observer.onNext(next);
        }

        @Override
        public void onError(Throwable e) {
            observer.onError(e);
        }

        @Override
        public void onComplete() {
            observer.onComplete();
        }
    }
}
```

IO线程

```java
public class ObservableIO<T> implements ObservableOnSubscribe<T> {
    // 线程池
    private static final ExecutorService EXECUTOR_SERVICE = Executors.newCachedThreadPool();

    private ObservableOnSubscribe<T> subscribe;

    public ObservableIO(ObservableOnSubscribe<T> subscribe) {
        this.subscribe = subscribe;
    }

    @Override
    public void subscribe(final Observer<T> observer) {
        // 开启线程
        EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                subscribe.subscribe(observer);
            }
        });
    }
}
```

main线程

```java
public class ObservableMain<T> implements ObservableOnSubscribe<T> {
    private ObservableOnSubscribe<T> subscribe;

    public ObservableMain(ObservableOnSubscribe<T> subscribe) {
        this.subscribe = subscribe;
    }

    @Override
    public void subscribe(Observer<T> observer) {
        new Handler(Looper.getMainLooper()).post(new Runnable() {
            @Override
            public void run() {
                subscribe.subscribe(observer);
            }
        });
    }
}
```

使用

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                runRxJava();
            }
        }).start();
    }

    private void runRxJava() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(Observer<Integer> observer) {
                observer.onNext(1);
                observer.onNext(2);
                observer.onNext(3);
                observer.onComplete();
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) { return "[" + integer + "]"; }
        })
        .subscribeOnMain() // 切换主线程
        .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe() {
                Log.d("wlzhou", "start, thread = " + Thread.currentThread().getName());
            }
            @Override
            public void onNext(String s) {
                Log.d("wlzhou", "onNext - " + s + ", thread = " + Thread.currentThread().getName());
            }
            @Override
            public void onError(Throwable e) {
                Log.d("wlzhou", "error - " + e.getMessage() + ", name = " + Thread.currentThread().getName());
            }
            @Override
            public void onComplete() {
                Log.d("wlzhou", "done, thread = " + Thread.currentThread().getName());
            }
        });
    }
}
```

运行结果

```
2023-06-25 18:27:24.508  8534-8563  wlzhou                  com.example.myapplication            D  start, thread = Thread-2
2023-06-25 18:27:24.625  8534-8534  wlzhou                  com.example.myapplication            D  onNext - [1], thread = main
2023-06-25 18:27:24.625  8534-8534  wlzhou                  com.example.myapplication            D  onNext - [2], thread = main
2023-06-25 18:27:24.625  8534-8534  wlzhou                  com.example.myapplication            D  onNext - [3], thread = main
2023-06-25 18:27:24.625  8534-8534  wlzhou                  com.example.myapplication            D  done, thread = main
```

#### 参考

+ [手写RxJava（简化版）](https://www.jianshu.com/p/deab6ec8aeea)
+ [史上最全的Rxjava2讲解（使用篇）](https://juejin.cn/post/6844903957496643597)
