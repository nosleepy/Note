---
title: 贝壳Java后端开发实习
date: 2020-06-05 10:17:44
tags:
categories:
- 面经合集
---

**Java的基本数据类型？**

+ 整形：byte，short，int，long
+ 字符型：char
+ 浮点型：float，double
+ 布尔型：boolean

**Java引用类型有哪些？**

+ 强引用（Strong Reference）：Java中的引用，默认都是强引用。比如new一个对象，对它的引用就是强引用。对于被强引用指向的对象，就算JVM内存不足OOM，也不会去回收它们。
+ 软引用（Soft Reference）：若一个对象只被软引用所引用，那么它将在JVM内存不足的时候被回收，即如果JVM内存足够，则软引用所指向的对象不会被垃圾回收。
+ 弱引用（Weak Reference）：若一个对象只被弱引用所引用，那么它将在下一次GC中被回收掉。
+ 虚引用（Phantom Reference）：虚引用是四种引用中最弱的一种引用。我们永远无法从虚引用中拿到对象，被虚引用引用的对象就跟不存在一样。

**String、StringBuffer、StringBuilder有什么区别？哪个适合多线程？**

1. String类中使用字符数组保存字符串，数组有"final"修饰符，String对象是不可变的，可以理解为常量，线程安全。
2. StringBuilder使用字符数组保存字符串，StringBuilder对象是可变的，没有对方法进行加同步锁，线程不安全。
3. StringBuffer使用字符数组保存字符串，StringBuffer对象是可变的，对方法加了同步锁，线程安全。

**HashMap线程安全吗？为什么？**

1. HashMap不是线程安全的。
2. 在jdk1.7中，多线程环境下，扩容时会造成环形链或数据丢失。
3. 在jdk1.8中，多线程环境下，会发生数据覆盖的情况。

**ArrayList线程安全吗？**

ArrayList线程不安全，Vector线程安全。

**有哪些线程安全的集合类？**

+ Vector：比Arraylist多了个同步化机制。
+ Hashtable：比HashMap多了个线程安全。
+ ConcurrentHashMap：是一种高效但是线程安全的集合。
+ Stack：栈，继承于Vector。

**ConcurrentHashMap的原理？**

1. jdk1.7：采用Segment数组 + HashEntry数组的方式进行实现，Segment通过继承ReentrantLock来进行加锁，所以每次需要加锁的操作锁住的是一个Segment，这样只要保证每个Segment是线程安全的，也就实现了全局的线程安全。
2. jdk1.8：采用Node数组 + CAS + Synchronized进行实现，使用数组元素作为锁，从而实现对每一行数据进行加锁，进一步减少并发冲突的概率。

**为什么要使用线程池**

1. 可以减少资源的消耗。线程的创建和销毁会造成一定的时间和空间上的消耗，而线程池可以让我们重复的利用已经创建好的线程，避免了不必要的浪费。
2. 提高了系统的响应速度。线程池是利用已经创建好的线程，没有线程的创建和销毁，所以响应速度很快。
3. 让线程更加便于管理。线程属于稀缺资源，我们不可以随意创建。运用线程池可以方便统一的管理。

**接口和抽象类的区别？**

1. 抽象类要被子类继承（extends），接口要被类实现（implements）。
2. 接口只能做方法申明，抽象类中可以做方法申明，也可以做方法实现。
3. 接口里定义的变量只能是公共的静态的常量，抽象类中的变量是普通变量。
4. 一个类只能继承一个抽象类，但是可以实现多个接口。

**浏览器输入网址，中间会经历什么**

1. 浏览器的地址栏输入URL并按下回车。
2. 浏览器查找当前URL是否存在缓存，并比较缓存是否过期。
3. DNS解析URL对应的IP。
4. 根据IP建立TCP连接（三次握手）。
5. HTTP发起请求。
6. 服务器处理请求，浏览器接收HTTP响应。
7. 渲染页面，构建DOM树。
8. 关闭TCP连接（四次挥手）。

**tcp/ip在哪一层？http呢？**

1. tcp/udp在传输层。
2. ip在网络层。
3. http在应用层。

**left join和inner join的区别**

+ 内连接：查询的是两张表的交集部分。
+ 左外连接：查询的是左表所有数据以及其交集部分。
+ 右外连接：查询的是右表所有数据以及其交集部分。

**参考**

+ [贝壳Java后端开发实习一二面HR面 | 6.3更新 已oc](https://www.nowcoder.com/discuss/435115)
+ [极致 HashMap（5）1.7 版的缺陷](https://zhuanlan.zhihu.com/p/99314228)
+ [HashMap线程不安全的体现](https://www.cnblogs.com/developer_chan/p/10450908.html)