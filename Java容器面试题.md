---
title: Java容器面试题
date: 2020-05-29 20:36:33
tags:
categories:
- Java面试
---

**ArrayList与Vector的区别是什么**
+ Vector是线程安全的，ArrayList不是线程安全的。
+ ArrayList在底层数组不够用时在原来的基础上扩展0.5倍，Vector是扩展1倍。
+ Vector在方法前面加了synchronized关键字，执行方法会有加锁和释放锁的开销
+ 在单线程环境下Vector的效率要低于ArrayList，多线程环境使用Vector。

**HashTable与HashMap的区别是什么**
+ HashTable是线程安全的，HashMap不是线程安全的。
+ HashTable的方法都加有synchronized关键字，效率低于HashMap。

**HashMap1.7和1.8的区别**
+ HashMap1.7里面是一个数组，然后数组中每个元素是一个单向链表。
+ HashMap1.8进行了一些修改，底层使用数组+链表+红黑树。
+ HashMap1.7在查找的时候，根据hash值能够快速定位到数组的具体下标，之后需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度。为了降低这部分的开销，HashMap1.8当链表中的元素达到了8个时，会将链表转换为黑红树，在这些位置进行查找的时候就可以降低时间复杂度。

**ConcurrentHashMap1.7和1.8的区别**
+ ConcurrentHashMap1.7是一个Segment数组，Segment通过继承ReentrantLock来进行加锁，所以每次需要加锁的操作锁住的是一个segment，这样只要保证每个Segment是线程安全的，也就实现了全局的线程安全。每个segment里面是多个table数组元素加链表的结构。
+ ConcurrentHashMap1.8取消segments字段，直接采用 transient volatile Node<K,V>[] table保存数据，采用table数组元素作为锁，从而实现对每一行数据进行加锁，进一步减少并发冲突的概率。底层使用table数组+链表+红黑树，链表长度超过8个会自动转换为红黑树，从而降低查询的时间复杂度，改进性能。

**线程安全的list**
+ Vector：线程安全，效率低，方法使用synchronized进行加锁。
+ SynchronizedList：把所有List接口的实现类转换成线程安全的List，方法使用synchronized进行加锁。
+ CopyOnWriteArrayList：复制再写入，就是在添加元素的时候，先把原List列表复制一份，再添加新的元素。读取元素时不加锁，写数据时才加锁。

**线程安全的map**
+ Hashtable：方法都加有synchronized关键字。
+ SynchronizedMap：把所有Map接口的实现类转换成线程安全的Map，方法都加有synchronized关键字。
+ ConcurrentHashMap：jdk1.7通过分段锁实现，jdk1.8使用数组元素作为锁。

**ConcurrentHashMap的原理？**

+ jdk1.7：采用Segment数组 + HashEntry数组的方式进行实现，Segment通过继承ReentrantLock来进行加锁，所以每次需要加锁的操作锁住的是一个Segment，这样只要保证每个Segment是线程安全的，也就实现了全局的线程安全。
+ jdk1.8：采用Node数组 + CAS + Synchronized进行实现，使用数组元素作为锁，从而实现对每一行数据进行加锁，进一步减少并发冲突的概率。

**JDK1.8中HashMap在出现hash碰撞时链表长度超过8一定会变成红黑树?**

实际上转换红黑树有个大前提，就是当前hash table的长度也就是HashMap的capacity(不是size)不能小于64，小于64就只是做个扩容。

单个链表的个数大于8时，数组长度小于64就扩容，数组长度大于等于64，则链表会转换为红黑树。

**如何遍历HashMap的键值**

keySet()、entrySet()、values()。