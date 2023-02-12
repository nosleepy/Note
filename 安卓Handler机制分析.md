---
title: 安卓Handler机制分析
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

一般项目中子线程处理耗时任务，需要使用 Handler 来切换到主线程进行 UI 更新操作。

```java
public class MainActivity extends AppCompatActivity {
    private final Handler mHandler = new Handler() {
        @Override
        public void handleMessage(@NonNull Message msg) {
            if (msg.what == 0) {
                //切换到主线程
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000); //模拟耗时操作
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                mHandler.sendEmptyMessage(0);
            }
        }).start();
    }
}
```

先看下 Handler 的构造方法

```java
//Handler.java
    public Handler(@Nullable Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper(); //返回 Looper对象
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue; // 保存Looper中的MessageQueue
        mCallback = callback;
        mAsynchronous = async;
    }
```

```java
//Looper.java
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get(); // 从ThreadLocal中获取Looper对象
    }
```

APP 进程创建后会执行 ActivityThread 的 main 方法，主线程的 Looper 对象在这里创建。

```java
//ActivityThread.java
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        Looper.prepareMainLooper(); //创建主线程的Looper对象
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }
        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop(); // 开启loop循环
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

Looper.prepareMainLooper() 方法用于创建主线程的 Looper 对象，构造方法中还会创建 MessageQueue 对象。

```java
//Looper.java
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
    
  private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
  private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed); // Looper创建时会实例化 MessageQueue对象
        mThread = Thread.currentThread();
    }
```

Looper.loop()方法从 MessageQueue 队列中取出 Message 消息，执行相应的任务。

```java
//Looper.java
    public static void loop() {
        final Looper me = myLooper();
        for (;;) {
            if (!loopOnce(me, ident, thresholdOverride)) {
                return;
            }
        }
    }
    
    private static boolean loopOnce(final Looper me,
            final long ident, final int thresholdOverride) {
        Message msg = me.mQueue.next(); // might block，取出消息
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return false;
        }
        try {
            msg.target.dispatchMessage(msg); //处理消息，执行任务，msg.target属性为发送Message消息的Handler对象
            if (observer != null) {
                observer.messageDispatched(token, msg);
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } catch (Exception exception) {
            if (observer != null) {
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
    }
```

回到 Handler.sendEmptyMessage() 方法中

```java
//Handler.java
    public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis); //消息入队操作
    }
    
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
        msg.target = this; //绑定target属性为Handler对象，由Handler来处理消息
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

处理消息时执行 Handler.dispatchMessage() 方法

```java
//Handler.java
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg); //这个方法就是Activity中重写的方法
        }
    }
```

msg.callback mCallback 都为 null 的情况下，Message 消息在自己重写的 handleMessgae 方法中处理，完成子线程切换到主线程。
