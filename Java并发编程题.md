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


