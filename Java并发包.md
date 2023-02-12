---
title: Java并发包
date: 2020-05-29 19:42:28
tags:
categories:
- Java并发
---

## Vector

*Vector* 和 *ArrayList* 底层通过数组实现，实现 *List* 接口。

*Vector* 是**线程安全**的类

*ArrayList* 是**线程不安全**的类

### add()

```java
public synchronized boolean add(E e) { // 使用了synchronized关键字
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```

### get()

```java
public synchronized E get(int index) { // 使用了synchronized关键字
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

## HashTable

*HashTable* 和 *HashMap* 底层通过数组加链表实现，实现了 *Map* 接口

*HashTable* **线程安全**

*HashMap* **线程不安全**

### put()

```java
public synchronized V put(K key, V value) { // 使用了synchronized关键字
}
```

### get()

```java
public synchronized V get(Object key) { // 使用了synchronized关键字
}
```

**Collections.synchronizedMap()**

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) { // 将不安全的Map集合转成安全的Map集合
    return new SynchronizedMap<>(m);
}
```

## ConcurrentHashMap

HashMap是线程不安全的类，HashTable线程安全，但每次操作都使用synchronized关键字，对全局加锁性能太低。

ConcurrentHashMap大量使用volatile，final，CAS等无锁技术来减少锁竞争对于性能的影响。避免了对全局加锁，使用对局部加锁，极大的提高了在并发环境下的操作速度。

### jdk1.7

底层实现使用 Segment数组 + HashEntry数组

<img src="https://www.javadoop.com/blogimages/map/3.png" width="480px"/>

### jdk1.8

底层实现使用 数组 + 链表 + 红黑树

<img src="https://www.javadoop.com/blogimages/map/4.png" width="480px"/>

## 原子类

### 类属性和成员变量

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```
1、AtomicInteger类使用Unsafe，直接操作内存来保证原子性。
2、long类型的valueOffset，在类加载时，初始化了值。
3、用volatile修饰的int类型的成员变量value，是AtomicInteger所包装的值。

### valueOffset

初始化赋值的方法

```java
public native long objectFieldOffset(Field var1); // 是Unsafe的一个native方法
```

`objectFieldOffset()`方法返回成员属性在内存中的地址相对于对象内存地址的偏移量

对于每个对象来说，偏移量都是固定的，所以作为一个类变量。

对象的内存地址+偏移量就可以知道成员变量value在内存中的具体地址。

### Unsafe

Unsafe是一个用于直接操作内存的类。

```java
// AtomicInteger对自增自减的非原子性操作的实现

public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
```

unsafe的方法

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

```java
getIntVolatile(var1, var2);
获取成员变量value的值，从主内存中加载，总能确保获取到的是有效的值
```

```java
compareAndSwapInt(var1, var2, var5, var5 + var4);
首先找出Object var1在内存中的位置p, 然后偏移var2个字节, 设p+var2处的这个int值为y,
如果y == var5, 则执行赋值操作y = var5+var4, 返回true
如果y != var5, 则不执行赋值操作, 返回false
```

1、取到对象var1的偏移量var2下的成员变量的值，读取值后，作为期望值。

2、在赋值操作的时候，先从内存中取到值和期望值比较，如果相等，则进行运算赋值操作，返回成功，结束。

3、否则，循环第一步。

### 其它方法

```java
public final int get() {
    return value;
}

public final void set(int newValue) {
    value = newValue;
}
```

set()和get()方法因为成员变量是volatile修饰，保证了内存的可见性

## CAS

### 介绍

CAS全称compare-and-swap，是计算机科学中一种实现多线程原子操作的指令，它比较内存中当前存在的值和外部给定的期望值，只有两者相等时，才将这个内存值修改为新的给定值。CAS操作包含三个操作数，需要读写的内存位置（V）、拟比较的预期原值（A）和拟写入的新值（B），如果V的值和A的值匹配，则将V的值更新为B，否则不做任何操作。多线程尝试使用CAS更新同一变量时，只有一个线程可以操作成功，其他的线程都会失败，失败的线程不会被挂起，只是在此次竞争中被告知失败，下次可以继续尝试CAS操作的。

### 存在的问题

#### ABA问题

因为CAS需要在操作值的时候，检查值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A、变成了B、又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但实际上却变化了。

增加一个版本号，当内存位置V的值每次被修改后，版本号都加1。

从jdk1.5开始，jdk的Atomic包里就提供了一个类AtomicStampedReference来解决ABA问题，这个类中的compareAndSet方法的作用就是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值更新为指定的新值

AtomicStampedReference内部维护了对象值和版本号，在创建AtomicStampedReference对象时，需要传入初始值和初始版本号，当AtomicStampedReference设置对象值时，对象值以及状态戳都必须满足期望值，写入才会成功。

#### 循环时间长开销大

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

#### 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。从java1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

## CountDownLatch

**介绍**

可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

**示例代码**

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(2);
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "-start");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-end");
                countDownLatch.countDown(); // 每次减去1
            }
        }, "t1").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "-start");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-end");
                countDownLatch.countDown(); // 每次减去1
            }
        }, "t2").start();
        countDownLatch.await(); // 主线程阻塞，countDown结果为0, 阻塞变为运行状态
        System.out.println("main" + "-start");
        for (int i = 0; i < 3; i++) {
            System.out.println("main i = " + i);
        }
        System.out.println("main" + "-end");
    }
}
```

**运行结果**

```java
t1-start
t2-start
t1-end
t2-end
main-start
main i = 0
main i = 1
main i = 2
main-end
```

## CyclicBarrier

**介绍**

CyclicBarrier的字面意思是可循环使用(Cyclic)的屏障(Barrier)。CyclicBarrier的作用是让一组线程之间相互等待，任何一个线程到达屏障点后就阻塞，直到最后一个线程到达，才都继续往下执行。个人理解：CyclicBarrier可以看成是一道大门或者关卡，先到的线程会被阻塞在大门口，直到最后一个线程到达屏障时，大门才被打开，所有被阻塞的线程才会继续干活。就像是朋友聚餐，只有最后一个朋友到达时，才会开吃！

**示例代码**

```java
class MyThread extends Thread {
    private CyclicBarrier cyclicBarrier;

    public MyThread (CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "-step1");
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "-step2");
            cyclicBarrier.await(); // 到达屏障，所有线程到达才向下执行
            System.out.println(Thread.currentThread().getName() + "-end");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
        for (int i = 0; i < 3; i++) {
            new MyThread(cyclicBarrier).start();
        }
    }
}
```

**运行结果**

```
Thread-0-step1
Thread-1-step1
Thread-2-step1
Thread-1-step2
Thread-0-step2
Thread-2-step2
Thread-2-end
Thread-0-end
Thread-1-end
```

## Semaphore

**介绍**

信号量，主要有两个目的,一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

**示例代码**

```java
public class Main {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 8; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire(); // 申请资源
                    System.out.println(Thread.currentThread().getName() + "\t抢到车位");
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t停车3秒后离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release(); // 释放资源
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

**运行结果**

```
2	抢到车位
1	抢到车位
3	抢到车位
2	停车3秒后离开车位
3	停车3秒后离开车位
1	停车3秒后离开车位
4	抢到车位
5	抢到车位
6	抢到车位
6	停车3秒后离开车位
7	抢到车位
4	停车3秒后离开车位
8	抢到车位
5	停车3秒后离开车位
8	停车3秒后离开车位
7	停车3秒后离开车位
```

## AtomicInteger

**介绍**

AtomicInteger是一个提供原子操作的Integer类，通过线程安全的方式操作加减。

**线程不安全示例**

```java
class MyThread implements Runnable {
    public static int count = 0;

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            count++;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        Thread t1 = new Thread(mt, "线程1");
        Thread t2 = new Thread(mt, "线程2");
        t1.start();
        t2.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(mt.count);
    }
}
```

```
198
```

**使用AtomicInteger**

```java
class MyThread implements Runnable {
    public static AtomicInteger ai = new AtomicInteger(0);

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            ai.getAndIncrement();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        Thread t1 = new Thread(mt, "线程1");
        Thread t2 = new Thread(mt, "线程2");
        t1.start();
        t2.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(mt.ai.get());
    }
}
```

```
200
```

## 参考

+ [java并发编程--AtomicInteger](https://www.cnblogs.com/rookie111/p/12622221.html)
+ [Java并发之CAS的三大问题](https://www.cnblogs.com/liukaifeng/p/10052640.html)
