---
title: Java并发编程题
date: 2020-08-07 11:20:18
tags:
categories:
- Java并发
---

## Java写一个死锁

**示例代码**

```java
public class DeadLockDemo {
    private static final String A = "A";
    private static final String B = "B";

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (A) {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("A is lock");
                    synchronized (B) {
                        System.out.println("1");
                    }
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (B) {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("B is lock");
                    synchronized (A) {
                        System.out.println("2");
                    }
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

**运行结果**

```
A is lock
B is lock
```

## 如何实现三个窗口并发卖票安全

**使用synchronized**

```java
class TicketWindow implements Runnable {
    private static int total_count = 100;

    @Override
    public void run() {
        while (true) {
            if (total_count > 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sale();
            } else {
                break;
            }
        }
        System.out.println(Thread.currentThread().getName() + "的票已售空");
    }

    private void sale() {
        synchronized (this) {
            if (total_count > 0) {
                System.out.println(Thread.currentThread().getName() + " 卖出第 " + (100 - total_count + 1) + " 张票");
                total_count--;
            }
        }
    }
}

public class SellTicketTest1 {
    public static void main(String[] args) {
        TicketWindow tw = new TicketWindow();

        Thread t1 = new Thread(tw, "窗口1");
        Thread t2 = new Thread(tw, "窗口2");
        Thread t3 = new Thread(tw, "窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

```
窗口1 卖出第 95 张票
窗口2 卖出第 96 张票
窗口2 卖出第 97 张票
窗口3 卖出第 98 张票
窗口1 卖出第 99 张票
窗口1 卖出第 100 张票
窗口2的票已售空
窗口3的票已售空
窗口1的票已售空
```

**使用reentrantLock**

```java
class TicketWindow implements Runnable {
    private static int total_count = 100;

    private final ReentrantLock lock = new ReentrantLock(true);

    @Override
    public void run() {
        while (true) {
            if (total_count > 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sale();
            } else {
                break;
            }
        }
        System.out.println(Thread.currentThread().getName() + "的票已售空");
    }

    private void sale() {
        lock.lock();
        try {
            if (total_count > 0) {
                System.out.println(Thread.currentThread().getName() + " 卖出第 " + (100 - total_count + 1) + " 张票");
                total_count--;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class SellTicketTest2 {
    public static void main(String[] args) {
        TicketWindow tw = new TicketWindow();

        Thread t1 = new Thread(tw, "窗口1");
        Thread t2 = new Thread(tw, "窗口2");
        Thread t3 = new Thread(tw, "窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

```
窗口1 卖出第 94 张票
窗口2 卖出第 95 张票
窗口3 卖出第 96 张票
窗口1 卖出第 97 张票
窗口3 卖出第 98 张票
窗口2 卖出第 99 张票
窗口1 卖出第 100 张票
窗口1的票已售空
窗口3的票已售空
窗口2的票已售空
```

## 3个线程打印ABC

+ 竞争型打印

```java
public class DemoTask implements Runnable {
    // 这里将lock对象换成 Lock(ReentrantLock) 进行lock/unlock也是可以的
    private static final Object lock = new Object();
    private static final int MAX = 30;
    private static int current = 0;
    private final int index;

    public DemoTask(int index) {
        this.index = index;
    }

    @Override
    public void run() {
        while (current < MAX) {
            if (current % 3 == index) {
                synchronized (lock) {
                    if (current < MAX) {
                        System.out.println((char) ('A' + current % 3)); // thread0打印'A',thread1打印'B',thread2打印'C'
                        current++;
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        List<Thread> threadList = Arrays.asList(
                new Thread(new DemoTask(0)),
                new Thread(new DemoTask(1)),
                new Thread(new DemoTask(2))
        );
        threadList.forEach(Thread::start);
    }
}
```

+ 协同型打印

```java
public class Main {
    //定义一个共享变量,用来在线程中进行通信
    private static final Object obj = new Object();
    //定义一个变量来记录打印的次数,控制打印条件
    private static int count = 1;

    public static void main(String[] args) {
        //创建三个线程,然后把三个任务分别放入这三个线程执行
        //创建线程1执行任务1
        new Thread(new Task1()).start();
        //创建线程2执行任务2
        new Thread(new Task2()).start();
        //创建线程3执行任务4
        new Thread(new Task3()).start();
    }

    //任务1
    private static class Task1 implements Runnable {
        @Override
        public void run() {
            synchronized (obj) {
                //打印十次A
                for (int i = 0; i < 10; i++) {
                    //一直轮询,如果条件不是要打印A的条件,那么直接释放锁
                    while (count % 3 != 1) {
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    //通知其他所有线程可以来抢占锁了,但是发现现在线程1在持有锁,其他线程还抢不到,只有等到线程1释放锁之后,才可以抢到锁
                    obj.notifyAll();
                    System.out.println("A");
                    count++;
                }
            }
        }
    }

    //任务2
    private static class Task2 implements Runnable {
        @Override
        public void run() {
            synchronized (obj) {
                //打印十次B
                for (int i = 0; i < 10; i++) {
                    //一直轮询,如果条件不是要打印B的条件,那么直接释放锁
                    while (count % 3 != 2) {
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    //通知其他所有线程可以来抢占锁了,但是发现现在线程2在持有锁,其他线程还抢不到,只有等到线程2释放锁之后,才可以抢到锁
                    obj.notifyAll();
                    System.out.println("B");
                    count++;
                }
            }
        }
    }

    //任务3
    private static class Task3 implements Runnable {
        @Override
        public void run() {
            synchronized (obj) {
                //打印十次C
                for (int i = 0; i < 10; i++) {
                    //一直轮询,如果条件不是要打印C的条件,那么直接释放锁
                    while (count % 3 != 0) {
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    //通知其他所有线程可以来抢占锁了,但是发现现在线程3在持有锁,其他线程还抢不到,只有等到线程3释放锁之后,才可以抢到锁
                    obj.notifyAll();
                    System.out.println("C");
                    count++;
                }
            }
        }
    }
}
```

## 参考

+ [如何优雅的让3个线程打印ABC](https://juejin.cn/post/6940274452559036453)
+ [三个线程交叉打印ABC（synchronized、wait/notify）](https://www.cnblogs.com/rao11/p/13728941.html)
