---
title: ThreadLocal原理
date: 2020-08-02 10:54:14
tags:
categories:
- Java并发
---

## ThreadLocal是什么

ThreadLocal是一种变量类型，我们称之为"线程局部变量"；每个线程访问这种变量的时候都会创建该变量的副本，这个变量副本为线程私有；ThreadLocal类型的变量一般用private static加以修饰。

## ThreadLocal示例

**示例代码**

```java
public class ThreadLocalTest {
    private static ThreadLocal<List<String>> threadLocal = new ThreadLocal<>();

    public void setThreadLocal(List<String> list) {
        threadLocal.set(list);
    }

    public void getThreadLocal() {
        threadLocal.get().forEach(str -> System.out.println(Thread.currentThread().getName() + "###" + str));
    }

    public static void main(String[] args) {
        final ThreadLocalTest test = new ThreadLocalTest();

        new Thread(new Runnable() {
            @Override
            public void run() {
                List<String> list = new ArrayList<String>();
                list.add("1");
                list.add("2");
                list.add("3");
                test.setThreadLocal(list);
                test.getThreadLocal();
            }
        },"t1").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                List<String> list = new ArrayList<>();
                list.add("a");
                list.add("b");
                list.add("c");
                test.setThreadLocal(list);
                test.getThreadLocal();
            }
        },"t2").start();
    }
}
```

**运行结果**

```
t1###1
t1###2
t1###3
t2###a
t2###b
t2###c
```

## ThreadLocal原理

**set方法**

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

**get方法**

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/ThreadLocal原理.png" width="400px"/>

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/ThreadLocal原理2.png" width="400px" style="margin-top: 20px"/>

**总结**

每个线程自身都维护着一个 ThreadLocalMap ，用来存储线程本地的数据，可以简单理解成 ThreadLocalMap 的 key 是 ThreadLocal 变量， value 是线程本地的数据。这样就实现了线程本地数据存储和交互访问。
