---
title: OkHttp原理分析
date: 2023-03-21 00:00:00
tags:
- 开源框架
categories:
- 安卓
---

#### 基础使用

添加依赖

```groovy
implementation 'com.squareup.okhttp3:okhttp:3.14.9'
```

使用mock模拟数据

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/mock_test_data.png)

示例代码

```kotlin
val client = OkHttpClient()
val request = Request.Builder().url("https://getman.cn/mock/test").build()
//1.异步调用
client.newCall(request).enqueue(object: Callback {
        override fun onFailure(call: Call, e: IOException) {
            Log.d(TAG, "fail: ${e.printStackTrace()}")
        }

        override fun onResponse(call: Call, response: Response) {
            Log.d(TAG, "response = ${response.body()?.string()}")
        }
    })
//2.同步调用
Thread {
    val response = client.newCall(request).execute()
    Log.d(TAG, "response = ${response.body()?.string()}")
}.start()
```

#### 初始化相关

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/okhttp_process.png)

OkHttpClient 源码

```java
public class OkHttpClient {
    final Dispatcher dispatcher;
    final boolean retryOnConnectionFailure;
    final int connectTimeout;

    public OkHttpClient() {
        this(new Builder());
    }

    OkHttpClient(Builder builder) {
        this.dispatcher = builder.dispatcher;
        this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
        this.connectTimeout = builder.connectTimeout;
    }

    public Builder newBuilder() {
        return new Builder(this);
    }

    public static final class Builder {
        Dispatcher dispatcher;
        boolean retryOnConnectionFailure;
        int connectTimeout;

        public Builder() {
            dispatcher = new Dispatcher();
            protocols = DEFAULT_PROTOCOLS;
        }

        Builder(OkHttpClient okHttpClient) {
            this.dispatcher = okHttpClient.dispatcher;
            this.retryOnConnectionFailure = okHttpClient.retryOnConnectionFailure;
            this.connectTimeout = okHttpClient.connectTimeout;
        }

        public OkHttpClient build() {
            return new OkHttpClient(this);
        }
    }
}
```

OkHttpClient 的创建最终调用 `OkHttpClient(Builder builder)` 构造方法，传入一个 Builder 对象

```kotlin
val client = OkHttpClient.Builder()..build()
```

Request 源码

```java
public final class Request {
  final HttpUrl url;
  final String method;
  final Headers headers;
  final @Nullable RequestBody body;
  final Map<Class<?>, Object> tags;
  private volatile @Nullable CacheControl cacheControl;
  //...
}
```

Response 源码

```java
public final class Response implements Closeable {
  final Request request;
  final Protocol protocol;
  final int code;
  final String message;
  final @Nullable Handshake handshake;
  final Headers headers;
  final @Nullable ResponseBody body;
  final @Nullable Response networkResponse;
  final @Nullable Response cacheResponse;
  final @Nullable Response priorResponse;
  final long sentRequestAtMillis;
  final long receivedResponseAtMillis;
  final @Nullable Exchange exchange;
  private volatile @Nullable CacheControl cacheControl;
  //...
}
```

Request 和 Response 封装了 http 请求和响应相关信息，同样使用建造者模式创建

Call 源码

```java
public class OkHttpClient implements Cloneable, Call.Factory {
	@Override 
    public Call newCall(Request request) {
    	return RealCall.newRealCall(this, request, false /* for web socket */);
  	}
}
```

```java
public interface Call extends Cloneable {
  Request request();

  Response execute() throws IOException; //同步执行

  void enqueue(Callback responseCallback); //异步执行

  void cancel();

  boolean isExecuted();

  boolean isCanceled();

  Timeout timeout();
  
  Call clone();

  interface Factory {
    Call newCall(Request request);
  }
}
```

```java
final class RealCall implements Call {
    static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
        RealCall call = new RealCall(client, originalRequest, forWebSocket);
        call.transmitter = new Transmitter(client, call);
        return call;
    }
}
```

OkHttpClient 的 newCall 方法创建的是 RealCall 对象

#### 请求分发

+ 同步执行

```java
@Override
public Response execute() throws IOException {
    synchronized (this) {
    //' 这里有个同步锁的抛异常操作'
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    try {
      //'1. 执行了dispatcher的executed方法'
      client.dispatcher().executed(this);
      //'2. 调用了getResponseWithInterceptorChain方法'
      return getResponseWithInterceptorChain();
    } finally {
      //'3. 最后一定会执行dispatcher的finished方法'
      client.dispatcher().finished(this);
    }
  }
```

首先将 RealCall 加入队列，然后执行 getResponseWithInterceptorChain 方法，最后将 RealCall 从队列移除

Dispatcher 源码

```java
public final class Dispatcher {
    private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>(); //准备异步队列
    private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>(); //运行异步队列
    private final Deque < RealCall > runningSyncCalls = new ArrayDeque < > (); //运行同步队列

    synchronized void executed(RealCall call) {
        runningSyncCalls.add(call);
    }

    void finished(RealCall call) {
        finished(runningSyncCalls, call);
    }

    private < T > void finished(Deque < T > calls, T call) {
        Runnable idleCallback;
        synchronized(this) {
            if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
            idleCallback = this.idleCallback;
        }
        boolean isRunning = promoteAndExecute();
        if (!isRunning && idleCallback != null) {
            idleCallback.run();
        }
    }
}
```

+ 异步执行

```java
@Override 
public void enqueue(Callback responseCallback) {
    synchronized (this) {
    	//'1. 这里有个同步锁的抛异常操作'
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    transmitter.callStart();
    //'2. 调用Dispatcher里面的enqueue方法'
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```

调用 Dispatcher 的 enqueue 方法，传入 Callback 参数

```java
void enqueue(AsyncCall call) {
    synchronized(this) {
        readyAsyncCalls.add(call);
        if (!call.get().forWebSocket) {
            AsyncCall existingCall = findExistingCallWithHost(call.host());
            if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
        }
    }
    promoteAndExecute();
}

private boolean promoteAndExecute() {
    assert(!Thread.holdsLock(this));

    List < AsyncCall > executableCalls = new ArrayList < > ();
    boolean isRunning;
    synchronized(this) {
        for (Iterator < AsyncCall > i = readyAsyncCalls.iterator(); i.hasNext();) {
            AsyncCall asyncCall = i.next();
            if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
            if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.
            i.remove();
            asyncCall.callsPerHost().incrementAndGet();
            executableCalls.add(asyncCall);
            runningAsyncCalls.add(asyncCall);
        }
        isRunning = runningCallsCount() > 0;
    }
    for (int i = 0, size = executableCalls.size(); i < size; i++) {
        AsyncCall asyncCall = executableCalls.get(i);
        asyncCall.executeOn(executorService());
    }
    return isRunning;
}
```

最后调用 AsyncCall 的 executeOn 方法处理任务，Callback 在异步执行开始被包装成 AsyncCall

```java
public abstract class NamedRunnable implements Runnable {
    protected final String name;

    public NamedRunnable(String format, Object...args) {
        this.name = Util.format(format, args);
    }

    @Override
    public final void run() {
        String oldName = Thread.currentThread().getName();
        Thread.currentThread().setName(name);
        try {
            execute();
        } finally {
            Thread.currentThread().setName(oldName);
        }
    }

    protected abstract void execute();
}
```

```java
  final class AsyncCall extends NamedRunnable {
      private final Callback responseCallback;
      private volatile AtomicInteger callsPerHost = new AtomicInteger(0);

      void executeOn(ExecutorService executorService) {
          assert(!Thread.holdsLock(client.dispatcher()));
          boolean success = false;
          try {
              executorService.execute(this);
              success = true;
          } catch (RejectedExecutionException e) {
              InterruptedIOException ioException = new InterruptedIOException("executor rejected");
              ioException.initCause(e);
              transmitter.noMoreExchanges(ioException);
              responseCallback.onFailure(RealCall.this, ioException);
          } finally {
              if (!success) {
                  client.dispatcher().finished(this); // This call is no longer running!
              }
          }
      }

      @Override 
      protected void execute() {
          boolean signalledCallback = false;
          transmitter.timeoutEnter();
          try {
              Response response = getResponseWithInterceptorChain();
              signalledCallback = true;
              responseCallback.onResponse(RealCall.this, response);
          } catch (IOException e) {
              if (signalledCallback) {
                  Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
              } else {
                  responseCallback.onFailure(RealCall.this, e);
              }
          } catch (Throwable t) {
              cancel();
              if (!signalledCallback) {
                  IOException canceledException = new IOException("canceled due to " + t);
                  canceledException.addSuppressed(t);
                  responseCallback.onFailure(RealCall.this, canceledException);
              }
              throw t;
          } finally {
              client.dispatcher().finished(this);
          }
      }
  }
}
```

由线程池去执行任务，run 方法内部调用 execute 方法，同样回到 getResponseWithInterceptorChain 方法，和同步执行的步骤一样

#### 拦截器

```java
  Response getResponseWithInterceptorChain() throws IOException {
      //创建拦截器list
      List < Interceptor > interceptors = new ArrayList < > ();
      interceptors.addAll(client.interceptors());//应用拦截器
      interceptors.add(new RetryAndFollowUpInterceptor(client));//重试与重定向拦截器
      interceptors.add(new BridgeInterceptor(client.cookieJar()));//内容拦截器
      interceptors.add(new CacheInterceptor(client.internalCache()));//缓存拦截器
      interceptors.add(new ConnectInterceptor(client));//连接拦截器
      if (!forWebSocket) {
          interceptors.addAll(client.networkInterceptors());//网络连接器
      }
      interceptors.add(new CallServerInterceptor(forWebSocket));//请求服务拦截器
	  //创建拦截器链条
      Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
          originalRequest, this, client.connectTimeoutMillis(),
          client.readTimeoutMillis(), client.writeTimeoutMillis());

      boolean calledNoMoreExchanges = false;
      try {
          //调用链条的proceed方法
          Response response = chain.proceed(originalRequest);
          if (transmitter.isCanceled()) {
              closeQuietly(response);
              throw new IOException("Canceled");
          }
          return response;
      } catch (IOException e) {
          calledNoMoreExchanges = true;
          throw transmitter.noMoreExchanges(e);
      } finally {
          if (!calledNoMoreExchanges) {
              transmitter.noMoreExchanges(null);
          }
      }
  }
```

```java
public final class RealInterceptorChain implements Interceptor.Chain {
    //'我们刚才建立的放拦截器的队列'
    private final List < Interceptor > interceptors;
    //'当前执行的第几个拦截器序号'
    private final int index;
    
    public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
        RealConnection connection) throws IOException {
        if (index >= interceptors.size()) throw new AssertionError();
        //'实例化了一个新的RealInterceptorChain对象，并且传入相同的拦截器List，只不过传入的index值+1'
        RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
            connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
            writeTimeout);
        //'获取当前index对应的拦截器里面的具体的某个拦截器，
        Interceptor interceptor = interceptors.get(index);
        //然后执行拦截器的intercept方法,同时传入新的RealInterceptorChain对象(主要的区别在于index+1了)'
        Response response = interceptor.intercept(next);
        return response;
    }
}
```

```java
public final class ConnectInterceptor implements Interceptor {
  public final OkHttpClient client;

  public ConnectInterceptor(OkHttpClient client) {
    this.client = client;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    Transmitter transmitter = realChain.transmitter();
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);
    return realChain.proceed(request, transmitter, exchange);//再次调用Chain的proceed方法让下一个拦截器执行任务
  }
}
```

使用责任链模式执行各个拦截器的 intercept 方法来获取返回结果 Response 对象

#### 责任链模式

实现一个链式返回信息的例子

```java
public class InterceptorChain {
    List<Interceptor> list;
    int index;
    String name;

    public InterceptorChain(String name, List<Interceptor> list, int index) {
        this.list = list;
        this.name = name;
        this.index = index;
    }

    public String proceed() {
        if (index >= list.size()) {
            return name;
        }
        InterceptorChain next = new InterceptorChain(name, list, index + 1);
        Interceptor interceptor = list.get(index);
        return interceptor.intercept(next);
    }
}
```

```java
public interface Interceptor {
    String intercept(InterceptorChain chain);
}

public class AddAddressInterceptor implements Interceptor {
    @Override
    public String intercept(InterceptorChain chain) {
        chain.name = chain.name + " address: China";
        return chain.proceed();
    }
}

public class AddTelephoneInterceptor implements Interceptor {
    @Override
    public String intercept(InterceptorChain chain) {
        String name = chain.proceed();
        chain.name = name + " telephone: 12345678";
        return chain.name;
    }
}
```

客户端调用

```java
public class Main {
    public static void main(String[] args) {
        List<Interceptor> list = new ArrayList<>();
        list.add(new AddTelephoneInterceptor());
        list.add(new AddAddressInterceptor());
        int index = 0;
        String name = "info = ";
        InterceptorChain chain = new InterceptorChain(name, list, index);
        String result = chain.proceed();
        System.out.println(result);
    }
}
```

输出结果

```
info =  address: China telephone: 12345678
```

#### 参考

+ [Android技能树 — 网络小结(6)之 OkHttp超超超超超超超详细解析](https://juejin.cn/post/6844903712809304072#heading-10)
