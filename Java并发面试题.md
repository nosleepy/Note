---
title: Java并发面试题
date: 2020-04-27 09:24:09
tags:
categories:
- Java面试
---

**创建一个线程的方式有哪几种**
   1. 继承Thread类
   2. 实现Runnable接口
   3. 实现Callable接口

**sleep()会让线程进入什么状态**
   + sleep()方法的作用是让当前线程暂停指定的时间，只是暂时让出CPU的执行权，并不会释放锁。会让线程进入阻塞状态。

**线程池相关参数介绍，原理说明**
   + 参数介绍
      + int corePoolSize：线程池的核心线程数量
      + int maximumPoolSize：线程池的最大线程数
      + long keepAliveTime：当线程数大于核心线程数时，多余的空闲线程存活的最长时间
      + TimeUnit unit：时间单位
      + BlockingQueue<Runnable> workQueue：任务队列，用来储存等待执行任务的队列
      + ThreadFactory threadFactory：线程工厂，用来创建线程，一般默认即可
      + RejectedExecutionHandler handler：拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
   + 原理说明
      1. 线程池中线程数量小于corePoolSize，此时任务不会进等待队列，线程池直接创建一个线程Worker执行提交的任务
      2. 线程池中线程数量不小于corePoolSize并且等待队列未满，任务直接添加到等待队列，等待线程池调度执行
	  3. 线程池中线程数量不小于corePoolSize但是等待队列已满且线程数量小于maximumPoolSize，线程池会进行扩容新创建一个线程Worker执行提交的任务，新创建的Worker会被添加到线程集合workers中
	  4. 等待队列已满并且线程数量已达到maximumPoolSize，这种情况下线程池无法继续执行任务会拒绝任务，执行一个指定的拒接策略
	  5. 线程池已关闭，拒绝任务，执行一个指定的拒接策略

**syncronized的工作原理**
   + 介绍：synchronized关键字用于在线程并发执行时，保证同一时刻，只有一个线程可以执行某个代码块或方法，同时还保证了代码在执行完后所修改的数据对其它线程是可见的。
   + 基本使用
      1. 同步普通方法：锁是当前对象
      2. 同步静态方法：锁是当前类的Class对象
      3. 同步代码块：锁是括号里的对象
   + 工作原理
      + 反编译字节码文件，发现方法的同步是通过指令monitorenter和monitorexit来完成的。
      + Java中的每个对象都有一个监视器锁（monitor），当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时会获取monitor的所有权将计数器加1，执行monitorexit指令时会释放monitor的所有权将计数器减1。

**为什么说syncronized是可重入的**
   + 当线程执行monitorenter指令时会尝试获取monitor的所有权
      1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
	  2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1。
      3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

**reentrantlock和synchronized的区别**
   + synchronized是可重入的，reentrantlock也是可重入。
   + synchronized是关键字，reentrantlock是Java类。
   + synchronized是通过JVM的字节码实现的，每个锁对象都绑定一个monitor，进入一个synchronized同步逻辑时需要获取该monitor并在计数时加1，离开时释放并减1，reentrantlock是通过CAS操作实现的。
   + synchronized的加锁和释放锁是自动的，reentrantlock需要手动加锁和释放锁。
   + reentrantlock有读写锁实现，在有读和写的并发需求时可以实现更有效率的并发。
   + synchronized是不可中断的，reentrantlock支持超时返回和中断。

**什么是线程安全问题**
   + 当多个线程共享同一个全局变量时，在进行写入操作，可能会受到其它线程的干扰，就会让数据有问题，这样就叫做线程安全问题。

**并发编程三要素**
   1. 原子性：指的是一个操作不能再继续拆分，要么一次操作完成，要么就是不执行。
   2. 可见性：指的是一个变量在被一个线程更改后，其它的线程能立即看到最新的值。
   3. 有序性：指的是程序的执行按照代码的先后顺序执行。

**volatile关键字的作用**
   1. 禁止指令重排，保证了有序性。
   2. 保证变量的读写及时从缓存刷新到主存，保证了可见性。

**synchronized关键字的作用**
   1. 保证同一时刻，只有一个线程可以执行某个代码块或方法
   2. 保证了代码在执行完后所修改的数据对其它线程是可见的
   3. 可以保证有序性，可见性，原子性。

**Unsafe类介绍**
   + Unsafe在sun.misc下，顾名思义，这是一个不安全的类，因为Unsafe类所操作的并不属于Java标准，Java的一系列内存操作都是交给jvm的，而Unsafe类却能有像C语言的指针一样直接操作内存的能力，同时也会带来了指针的问题。过度使用Unsafe类的话，会使出错率变得更大，因此官方才命名为Unsafe，并且不建议使用，连注释的没有。而为了安全使用Unsafe，Unsafe类只允许jdk自带的类使用。

**什么是Java内存模型（共享内存模型）**
  + JMM决定一个线程对共享变量的写入何时对另一个线程可见。
  + 从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，本地内存中存储了该线程以读/写共享变量的副本。
  + 线程A与线程B之间通信：线程A把本地内存A中更新过的共享变量刷新到主内存中去，线程B到主内存中去读取线程A之前已更新过的共享变量。

**notify和notifyAll**

notify唤醒一个线程，notifyAll唤醒所有的线程。

**线程的状态**

New（新建状态），Runnable（就绪状态），Running（运行状态），Blocked（阻塞状态），Dead（死亡状态）。

**CAS底层实现**

比较内存中当前存在的值和外部给定的期望值，只有两者相等时，才将这个内存值修改为新的给定值。CAS操作包含三个操作数，需要读写的内存位置（V）、拟比较的预期原值（A）和拟写入的新值（B），如果V的值和A的值匹配，则将V的值更新为B，否则不做任何操作。

**CAS存在的问题**

1. ABA问题。
2. 循环时间长开销大。
3. 只能保证一个共享变量的原子操作。

**什么是可重入锁**

某个线程已经获得某个锁，可以再次获取锁而不会出现死锁。

**什么是自旋锁**

当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

**什么是公平锁**

多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

**什么是非公平锁**

多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

**线程池的阻塞队列（[博客园](https://www.cnblogs.com/aiguona/p/10634043.html)）**

+ ArrayBlockingQueue：基于数组实现的一个阻塞队列，在创建ArrayBlockingQueue对象时必须制定容量大小。
+ LinkedBlockingQueue：基于链表实现的一个阻塞队列，在创建LinkedBlockingQueue对象时如果不指定容量大小，则默认大小为Integer.MAX_VALUE。
+ PriorityBlockingQueue：按照元素的优先级对元素进行排序，按照优先级顺序出队，每次出队的元素都是优先级最高的元素。
+ DelayQueue：基于PriorityQueue，一种延时阻塞队列，DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。

**线程池的4种拒绝策略**

1. AbortPolicy：直接抛出异常。
2. CallerRunsPolicy：直接在execute方法的调用线程中运行被拒绝的任务。
3. DiscardOldestPolicy：丢弃最旧的一个请求，并尝试再次提交当前任务。
4. DiscardPolicy：丢弃被拒绝的任务。

**实现Runnable接口和继承Thread类的区别**

1. 实现Runnable接口可以再继承其它类。
2. 继承Thread类不能再继承其它类。
3. 实现Runnable接口的线程间可以完成资源的共享，同时处理同一资源。
4. 继承Thread类的线程间都是独立运行的，资源不共享。

**为什么要使用线程池**

1. 可以减少资源的消耗。线程的创建和销毁会造成一定的时间和空间上的消耗，而线程池可以让我们重复的利用已经创建好的线程，避免了不必要的浪费。
2. 提高了系统的响应速度。线程池是利用已经创建好的线程，没有线程的创建和销毁，所以响应速度很快。
3. 让线程更加便于管理。线程属于稀缺资源，我们不可以随意创建。运用线程池可以方便统一的管理。

**Java中死锁的理解（[死锁的简单理解及示例](https://www.cnblogs.com/expiator/p/9391092.html)）**

+ 当两个或多个线程正在等待彼此释放所需资源（锁定）并陷入无限时间的阻塞。
+ 两个或多个线程在执行过程中，相互争夺资源而造成的一种互相等待的现象。

**停止一个线程用什么方法**

Thread.interrupt()方法: 作用是中断线程。将会设置该线程的中断状态位，即设置为true，中断的结果线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身。线程会不时地检测这个中断标示位，以判断线程是否应该被中断（中断标示值是否为true）。它并不像stop方法那样会中断一个正在运行的线程。

**run方法和start方法的区别**

调用start方法方可启动线程，而run方法只是thread类中的一个普通方法调用，还是在主线程里执行。

**谈谈你对AQS的理解（[知乎](https://zhuanlan.zhihu.com/p/86072774)）**

AQS就是一个并发包的基础组件，用来实现各种锁，各种同步组件的。它包含了state变量、加锁线程、等待队列等并发中的核心组件。

**在Java中wait和sleep方法的不同**

+ 在等待时wait会释放锁，而sleep一直持有锁。
+ wait通常被用于线程间交互，sleep通常被用于暂停执行。

**什么是线程池**

事先将多个线程对象放到一个容器中，当使用的时候就不用new线程而是直接去池中拿线程即可，节省了开辟子线程的时间，提高了代码的执行效率。

**Java中多线程间的通信怎么实现**

+ 共享变量。
+ wait/notify机制。

**Synchronized原理**

+ 对象锁原理。
+ 代码块加锁解锁过程。
+ 方法体加锁解锁过程。

**对象锁原理**

在Java中，每个锁对象内部都有一个monitor对象（监视器锁）。Java虚拟机中，monitor由ObjectMonitor实现，ObjectMonitor的三个成员变量：_owner（指向获得monitor的线程）、_EntryList（处于block状态的线程，会被加入到entry set）、_WaitSet（处于wait状态的线程，会被加入到wait set）。多个线程同时访问一段同步代码时，首先会进入_EntryList集合，进行阻塞等待，当线程获取到锁对象的monitor之后进入到_owner区域，并把monitor中的_owner变量指向该线程，同时monitor中的计数器count加一，若线程调用同步对象的wait()方法将释放当前持有的monitor，_owner变量重置为null，count置为0，同时该线程进入_WaitSet中等待唤醒，线程执行完同步代码后，也将_owner和count变量重置。

<img src="https://pic1.zhimg.com/80/v2-1b7171e77a484cd094078227982c283a_720w.jpg" width="320px"/>

**代码块加锁解锁过程**

反编译字节码，底层使用monitorenter和monitorexit指令实现。执行monitorenter指令，线程尝试获取锁对象的monitor对象，若monitor的count变量为0，则将count设置为1，_owner设置为当前线程，如果线程已经获取到monitor对象，则可以重入该锁，将count计数器的值加一。执行monitorexit指令，会将计数器count的值减一，当计数器为0时，当前线程释放monitor对象，其它线程有机会获得monitor对象。

**方法体加锁解锁过程**

反编译字节码，底层通过ACC_SYNCHRONIZED访问标志判断一个方法是否同步方法。调用方法时，检测这个标志是否被设置，如果设置了，线程需要先获得锁对象的monitor对象，然后执行方法，最后在方法完成时释放monitor对象。

**线程同步的方式**

+ 使用synchronized关键字同步方法。
+ 使用synchronized关键字同步代码块。
+ 使用特殊域变量(volatile)。
+ 使用重入锁实现线程同步。
+ 使用局部变量实现线程同步。
+ 使用阻塞队列实现线程同步。
+ 使用原子变量实现线程同步。

**参考**

+ [面试官，别挂电话，Synchronized，我还能说上半小时](https://zhuanlan.zhihu.com/p/132742291)

