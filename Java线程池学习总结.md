---
title: Java线程池学习总结
date: 2020-04-17 19:59:06
tags:
categories: 
- Java并发
---

## 前言

处理器早已迈入多核心时代，为了充分利用cpu多核资源，应用都会采用多线程并行/并发计算，最大限度的利用多核提升应用程序性能。然而线程的创建是有代价的，一方面需要申请内存资源，另一方面需要操作系统内核把线程加入调度队列，开销是比较大的，这在高并发系统中性能隐患非常大，另一方面线程需要消耗内存空间，如果进程创建的线程数量不加以控制，很有可能会耗尽进程的内存空间。

## 线程池的作用

1、不同请求之间重复利用线程，无需频繁的创建和销毁线程，降低系统开销；
2、控制线程数量上限，避免创建过多的线程耗尽进程内存空间，同时减少线程上下文切换次数。

## 线程池实现原理

> jdk在java5版本中增加了内置线程池实现ThreadPoolExecutor，本文通过ThreadPoolExecutor的源码分析jdk中线程池的实现原理。

**线程池由两个核心数据结构组成：**

1. 线程集合（workers）：存放执行任务的线程，是一个HashSet；
2. 任务等待队列（workQueue）：存放等待线程池调度执行的任务，是一个阻塞式队列BlockingQueue；

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/线程池的组成.jpg" width="360px">

**线程池有几个核心参数：**

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/线程池的核心参数.png" width="480px"/>

**任务执行流程**

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/线程池的任务执行流程.png" width="500px"/>

任务由execute方法提交到线程池中调度，在提交任务时会有下面几种场景：

1. 线程池中线程数量小于corePoolSize，此时任务不会进等待队列，线程池直接创建一个线程Worker执行提交的任务；

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/任务执行流程-1.jpg" width="360px"/>

2. 线程池中线程数量不小于corePoolSize并且等待队列未满，任务直接添加到等待队列，等待线程池调度执行；

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/任务执行流程-2.jpg" width="360px"/>

3. 线程池中线程数量不小于corePoolSize但是等待队列已满且线程数量小于maximumPoolSize，线程池会进行扩容新创建一个线程Worker执行提交的任务，新创建的Worker会被添加到线程集合workers中；

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/任务执行流程-3.jpg" width="360px"/>

4. 等待队列已满并且线程数量已达到maximumPoolSize，这种情况下线程池无法继续执行任务会拒绝任务，执行一个指定的拒接策略。

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/任务执行流程-4.jpg" width="360px"/>

5. 线程池已关闭，拒绝任务，执行一个指定的拒接策略。

线程创建之后，会不停从等待队列workQueue中拉取任务，workQueue是一个线程安全的阻塞队列，所以不存在线程安全问题，拉取到任务之后，执行任务逻辑。拉取任务时有两种情况：

1. 线程池设置了keepAliveTime参数，并且此时线程池中的线程数量超过核心数量corePoolSize，从队列中拉取任务时会设置keepAliveTime为超时时间，超过这个时间之后，该线程不再等待任务，直接跑完run方法体，线程被回收；
2. 否则线程会无限等待任务队列直到有任务到来。

**拒绝策略**

当线程集合和等待队列都满时线程无法调度任务，这时线程池会执行一个默认的或使用者指定的拒绝策略。

JDK内置的拒绝策略主要有下面几种：

1. 调用线程执行（CallerRunsPolicy），任务被线程池拒绝后，任务会被调用线程执行；
2. 终止执行（AbortPolicy），任务被拒绝时，抛出RejectedExecutionException异常报错
3. 丢弃任务（DiscardPolicy），任务被直接丢弃，不会抛异常报错；
4. 丢失老任务（DiscardOldestPolicy），把等待队列中最老的任务删除，删除后重新提交当前任务。

除了这些内置的拒绝策略，使用者还可以实现RejectedExecutionHandler接口自定义拒绝策略

**关闭线程池**

关闭线程池时有两个关键步骤：

1. 修改线程池状态到SHUTDOWN，这时新提交到线程池的任务都会被直接拒绝；
2. 中断线程池中的所有线程，中断任务执行回收线程集合中所有线程。

**按时调度线程**

JDK还内置了延时/定时调度任务的线程池，能够延时/定时执行提交的任务，它和普通线程池实现上的区别是，任务队列使用了定制的阻塞队列DelayedWorkQueue，该队列会对添加到队列中的任务按时间排序，在从队列中拉取任务时，只有队头任务指定的时间超过了延时时间时才会有任务出队，否则会一直等待。

## Executor框架

**结构图**

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/Executor.jpg" width="320px"/>

+ Executor：一个接口，其定义了一个接收 Runnable 对象的方法 executor，其方法签名为 executor(Runnable command)。
+ ExecutorService：是一个比 Executor 使用更广泛的子类接口，其提供了生命周期管理的方法，以及可跟踪一个或多个异步任务执行状况返回 Future 的方法。
+ AbstractExecutorService：ExecutorService 执行方法的默认实现。
+ ScheduledExecutorService：一个可定时调度任务的接口。
+ ScheduledThreadPoolExecutor：ScheduledExecutorService 的实现，一个可定时调度任务的线程池。
+ ThreadPoolExecutor：线程池，可以通过调用 Executors 以下静态工厂方法来创建线程池并返回一个 ExecutorService 对象。

**ThreadPoolExecutor类分析**

```java
/**
 * 用给定的初始参数创建一个新的ThreadPoolExecutor。
 */
public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                          int maximumPoolSize,//线程池的最大线程数
                          long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                          ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                          RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                           ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

## 4种线程池

**1.newSingleThreadExecutor**

创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

**2.newFixedThreadPool**

创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

**3.newCachedThreadPool**

创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

**4.newScheduledThreadPool**

创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

## 自定义线程池

**实现代码**

```java
class TaskThread implements Runnable {

    private String threadName;

    public TaskThread(String threadName) {
        this.threadName = threadName;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + threadName);
    }
}

public class MyThreadPool {

    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 2,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(3));
        threadPoolExecutor.execute(new TaskThread("任务1"));
        threadPoolExecutor.execute(new TaskThread("任务2"));
        threadPoolExecutor.execute(new TaskThread("任务3"));
        threadPoolExecutor.execute(new TaskThread("任务4"));
        threadPoolExecutor.execute(new TaskThread("任务5"));
    }

}
```

**运行结果**

```
pool-1-thread-1任务1
pool-1-thread-2任务5
pool-1-thread-1任务2
pool-1-thread-2任务3
pool-1-thread-1任务4
```

## 参考

+ [面试必问：java线程池实现原理](https://baijiahao.baidu.com/s?id=1641469444994560637&wfr=spider&for=pc)
+ [Java几种线程池的分析和使用](https://zhuanlan.zhihu.com/p/22882522)
+ [java线程池学习总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93.md)

