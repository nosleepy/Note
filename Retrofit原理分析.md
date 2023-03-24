---
title: Retrofit原理分析
date: 2023-03-24 19:06:13
tags:
- 开源框架
categories:
- 安卓
---

Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装，网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责 网络请求接口的封装。在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit根据用户的需求对结果进行解析。

#### 基础使用

添加依赖

```groovy
implementation "com.squareup.retrofit2:converter-gson:2.9.0" //json转换
implementation "com.squareup.retrofit2:retrofit:2.9.0"
```

添加网络访问权限

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

使用 mock 模拟测试数据

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/mock_user_list.png)

定义 UserService 接口

```kotlin
interface UserService {
    @GET("users")
    fun getUsers(): Call<List<User>>

    companion object {
        private const val BASE_URL = "https://getman.cn/mock/"

        fun create(): UserService {
            val client = OkHttpClient.Builder().build()
            return Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build()
                .create(UserService::class.java)
        }
    }
}
```

示例代码

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val call = UserService.create().getUsers()
        call.enqueue(object : Callback<List<User>> {
            override fun onResponse(call: Call<List<User>>, response: Response<List<User>>) {
                val list = response.body()
                Log.d(TAG, "list = $list")
                Log.d(TAG, "user[0] = ${list!![0]}")
            }

            override fun onFailure(call: Call<List<User>>, t: Throwable) {
                Log.d(TAG, "failure")
            }
        })
    }

    companion object {
        private const val TAG = "RetrofitTest"
    }
}
```

运行日志打印

```
list = [User(id=1, email=george.bluth@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/1-image.jpg), User(id=2, email=janet.weaver@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/2-image.jpg), User(id=3, email=emma.wong@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/3-image.jpg), User(id=4, email=eve.holt@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/4-image.jpg), User(id=5, email=charles.morris@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/5-image.jpg), User(id=6, email=tracey.ramos@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/6-image.jpg)]
user[0] = User(id=1, email=george.bluth@reqres.in, firstName=null, lastName=null, avatar=https://reqres.in/img/faces/1-image.jpg)
```

#### 原理分析

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/retrofit_process.png)

retrofit 的创建使用构建者模式

```kotlin
val BASE_URL = "https://getman.cn/mock/"
val client = OkHttpClient.Builder().build()
val retrofit = Retrofit.Builder()
	.baseUrl(BASE_URL)
	.client(client)
	.addCallAdapterFactory(Java8CallAdapterFactory.create()) //请求适配工厂
	.addConverterFactory(GsonConverterFactory.create()) //格式转换工厂
	.build()
```

使用 retrofit 示例

```kotlin
val userService = retrofit.create(UserService::class.java)
val call = userService.getUsers() 
```

retrofit 的 create 方法传入一个 Class 泛型类对象，返回 UserService 接口类型对象

```java
  public < T > T create(final Class < T > service) {
      validateServiceInterface(service);
      return (T)
      Proxy.newProxyInstance(
          service.getClassLoader(),
          new Class <? > [] {
              service
          },
          new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public@ Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
                  // If the method is a method from Object then defer to normal invocation.
                  if (method.getDeclaringClass() == Object.class) {
                      return method.invoke(this, args);
                  }
                  args = args != null ? args : emptyArgs;
                  return platform.isDefaultMethod(method) ? platform.invokeDefaultMethod(method, service, proxy, args) : loadServiceMethod(method).invoke(args);
              }
          });
  }
```

Proxy.newProxyInstance 方法是 JDK 动态代理的使用，通过反射生成一个实现 UserService 接口的对象

```java
  ServiceMethod <? > loadServiceMethod(Method method) {
      ServiceMethod <? > result = serviceMethodCache.get(method);
      if (result != null) return result;

      synchronized(serviceMethodCache) {
          result = serviceMethodCache.get(method);
          if (result == null) {
              result = ServiceMethod.parseAnnotations(this, method);
              serviceMethodCache.put(method, result);
          }
      }
      return result;
  }
```

loadServiceMethod 方法会先判断缓存中是否存在 ServiceMethod 对象，没有调用 ServiceMethod.parseAnnotations 方法解析方法注解信息

```java
abstract class ServiceMethod<T> {
  static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
    RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);

    Type returnType = method.getGenericReturnType();
    if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(
          method,
          "Method return type must not include a type variable or wildcard: %s",
          returnType);
    }
    if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
    }

    return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
  }

  abstract @Nullable T invoke(Object[] args);
}
```

RequestFactory.parseAnnotations 方法解析注解信息构建 RequestFactory 请求工厂对象，封装请求信息，HttpServiceMethod 继承 ServiceMethod

```java
  static < ResponseT, ReturnT > HttpServiceMethod < ResponseT, ReturnT > parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
      boolean isKotlinSuspendFunction = requestFactory.isKotlinSuspendFunction;
      boolean continuationWantsResponse = false;
      boolean continuationBodyNullable = false;

      Annotation[] annotations = method.getAnnotations(); //返回方法所有注解
      Type adapterType;
      if (isKotlinSuspendFunction) {
          Type[] parameterTypes = method.getGenericParameterTypes();
          Type responseType =
              Utils.getParameterLowerBound(
                  0, (ParameterizedType) parameterTypes[parameterTypes.length - 1]);
          if (getRawType(responseType) == Response.class && responseType instanceof ParameterizedType) {
              // Unwrap the actual body type from Response<T>.
              responseType = Utils.getParameterUpperBound(0, (ParameterizedType) responseType);
              continuationWantsResponse = true;
          } else {
              // TODO figure out if type is nullable or not
              // Metadata metadata = method.getDeclaringClass().getAnnotation(Metadata.class)
              // Find the entry for method
              // Determine if return type is nullable or not
          }

          adapterType = new Utils.ParameterizedTypeImpl(null, Call.class, responseType);
          annotations = SkipCallbackExecutorImpl.ensurePresent(annotations);
      } else {
          adapterType = method.getGenericReturnType(); //方法返回类型，包含泛型参数
      }

      CallAdapter < ResponseT, ReturnT > callAdapter =
          createCallAdapter(retrofit, method, adapterType, annotations); //创建CallAdapter
      Type responseType = callAdapter.responseType();
      if (responseType == okhttp3.Response.class) {
          throw methodError(
              method,
              "'" + getRawType(responseType).getName() + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      if (responseType == Response.class) {
          throw methodError(method, "Response must include generic type (e.g., Response<String>)");
      }
      // TODO support Unit for Kotlin?
      if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
          throw methodError(method, "HEAD method must use Void as response type.");
      }

      Converter < ResponseBody, ResponseT > responseConverter =
          createResponseConverter(retrofit, method, responseType); //创建响应结果转换器

      okhttp3.Call.Factory callFactory = retrofit.callFactory; //
      if (!isKotlinSuspendFunction) {
          return new CallAdapted < > (requestFactory, callFactory, responseConverter, callAdapter);
      } else if (continuationWantsResponse) {
          //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
          return (HttpServiceMethod < ResponseT, ReturnT > )
          new SuspendForResponse < > (
              requestFactory,
              callFactory,
              responseConverter, (CallAdapter < ResponseT, Call < ResponseT >> ) callAdapter);
      } else {
          //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
          return (HttpServiceMethod < ResponseT, ReturnT > )
          new SuspendForBody < > (
              requestFactory,
              callFactory,
              responseConverter, (CallAdapter < ResponseT, Call < ResponseT >> ) callAdapter,
              continuationBodyNullable);
      }
  }
```

+ 获取返回类型

```java
Type adapterType = method.getGenericReturnType();//就是Call<List<User>>
```

+ CallAdapter 的创建

```java
  private static <ResponseT, ReturnT> CallAdapter<ResponseT, ReturnT> createCallAdapter(
      Retrofit retrofit, Method method, Type returnType, Annotation[] annotations) {
    try {
      //noinspection unchecked
      return (CallAdapter<ResponseT, ReturnT>) retrofit.callAdapter(returnType, annotations);
    } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create call adapter for %s", returnType);
    }
  }
```

交给 retrofit 去创建

```java
  public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
    return nextCallAdapter(null, returnType, annotations);
  }
  
  public CallAdapter<?, ?> nextCallAdapter(
      @Nullable CallAdapter.Factory skipPast, Type returnType, Annotation[] annotations) {
    Objects.requireNonNull(returnType, "returnType == null");
    Objects.requireNonNull(annotations, "annotations == null");

    int start = callAdapterFactories.indexOf(skipPast) + 1;
    for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
      CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
      if (adapter != null) {
        return adapter;
      }
    }

    StringBuilder builder =
        new StringBuilder("Could not locate call adapter for ").append(returnType).append(".\n");
    if (skipPast != null) {
      builder.append("  Skipped:");
      for (int i = 0; i < start; i++) {
        builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
      }
      builder.append('\n');
    }
    builder.append("  Tried:");
    for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
      builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
    }
    throw new IllegalArgumentException(builder.toString());
  }
```

nextCallAdapter 方法根据返回类型在 callAdapterFactories 中找到合适的 CallAdapter

+ responseConverter 的创建

```java
  private static <ResponseT> Converter<ResponseBody, ResponseT> createResponseConverter(
      Retrofit retrofit, Method method, Type responseType) {
    Annotation[] annotations = method.getAnnotations();
    try {
      return retrofit.responseBodyConverter(responseType, annotations);
    } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create converter for %s", responseType);
    }
  }
```

同样交给 retrofit 去创建

```java
  public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
    return nextResponseBodyConverter(null, type, annotations);
  }
  
    public <T> Converter<ResponseBody, T> nextResponseBodyConverter(
      @Nullable Converter.Factory skipPast, Type type, Annotation[] annotations) {
    Objects.requireNonNull(type, "type == null");
    Objects.requireNonNull(annotations, "annotations == null");

    int start = converterFactories.indexOf(skipPast) + 1;
    for (int i = start, count = converterFactories.size(); i < count; i++) {
      Converter<ResponseBody, ?> converter =
          converterFactories.get(i).responseBodyConverter(type, annotations, this);
      if (converter != null) {
        //noinspection unchecked
        return (Converter<ResponseBody, T>) converter;
      }
    }

    StringBuilder builder =
        new StringBuilder("Could not locate ResponseBody converter for ")
            .append(type)
            .append(".\n");
    if (skipPast != null) {
      builder.append("  Skipped:");
      for (int i = 0; i < start; i++) {
        builder.append("\n   * ").append(converterFactories.get(i).getClass().getName());
      }
      builder.append('\n');
    }
    builder.append("  Tried:");
    for (int i = start, count = converterFactories.size(); i < count; i++) {
      builder.append("\n   * ").append(converterFactories.get(i).getClass().getName());
    }
    throw new IllegalArgumentException(builder.toString());
  }
```

从 converterFactories 中返回合适的 responseConverter

最终返回一个 CallAdapted 对象，CallAdapted 继承 HttpServiceMethod

```java
return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
```

回到 invoke 方法中，调用的是 CallAdapted 的 invoke 方法

```java
  @Override
  final@ Nullable ReturnT invoke(Object[] args) {
      Call < ResponseT > call = new OkHttpCall < > (requestFactory, args, callFactory, responseConverter);
      return adapt(call, args);
  }

  @Override
  protected ReturnT adapt(Call < ResponseT > call, Object[] args) {
      return callAdapter.adapt(call);
  }
```

retrofit 在构造的时候默认会添加 Android 平台的 CallAdapter

```java
    public Retrofit build() {
        // Make a defensive copy of the adapters and add the default Call adapter.
        List < CallAdapter.Factory > callAdapterFactories = new ArrayList < > (this.callAdapterFactories);
        callAdapterFactories.addAll(platform.defaultCallAdapterFactories(callbackExecutor));
    }

    List <? extends CallAdapter.Factory > defaultCallAdapterFactories(@Nullable Executor callbackExecutor) {
        DefaultCallAdapterFactory executorFactory = new DefaultCallAdapterFactory(callbackExecutor);
        return hasJava8Types ? asList(CompletableFutureCallAdapterFactory.INSTANCE, executorFactory) : singletonList(executorFactory);
    }
```

DefaultCallAdapterFactory 的 get 方法返回的就是具体的 CallAdapter 了

```java
  @Override
  public@ Nullable CallAdapter <? , ?> get(
      Type returnType, Annotation[] annotations, Retrofit retrofit) {
      if (getRawType(returnType) != Call.class) {
          return null;
      }
      if (!(returnType instanceof ParameterizedType)) {
          throw new IllegalArgumentException(
              "Call return type must be parameterized as Call<Foo> or Call<? extends Foo>");
      }
      final Type responseType = Utils.getParameterUpperBound(0, (ParameterizedType) returnType);

      final Executor executor =
          Utils.isAnnotationPresent(annotations, SkipCallbackExecutor.class) ? null : callbackExecutor;

      return new CallAdapter < Object, Call <? >> () {
         @Override
          public Type responseType() {
              return responseType;
          }

          @Override
          public Call < Object > adapt(Call < Object > call) {
              return executor == null ? call : new ExecutorCallbackCall < > (executor, call);
          }
      };
  }
```

继续调用 callAdapter 的 adapt 方法，传入 OkHttpCall 对象，返回的是 ExecutorCallbackCall 对象

```java
static final class ExecutorCallbackCall < T > implements Call < T > {
    final Executor callbackExecutor;
    final Call < T > delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call < T > delegate) {
        this.callbackExecutor = callbackExecutor;
        this.delegate = delegate;
    }

    @Override
    public void enqueue(final Callback < T > callback) {
        Objects.requireNonNull(callback, "callback == null");

        delegate.enqueue(
            new Callback < T > () {
            @Override
                public void onResponse(Call < T > call, final Response < T > response) {
                    callbackExecutor.execute(
                        () - > {
                            if (delegate.isCanceled()) {
                                // Emulate OkHttp's behavior of throwing/delivering an IOException on
                                // cancellation.
                                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
                            } else {
                                callback.onResponse(ExecutorCallbackCall.this, response);
                            }
                        });
                }

                @Override
                public void onFailure(Call < T > call, final Throwable t) {
                    callbackExecutor.execute(() - > callback.onFailure(ExecutorCallbackCall.this, t));
                }
            });
    }
}
```

UserService 的 getUsers 方法最后返回的是 ExecutorCallbackCall 对象，调用 enqueue 执行异步请求内部调用的是 OkHttpCall 代理对象的方法去实现

```java
//OkHttpCall.java
@Override
public void enqueue(final Callback < T > callback) {
    Objects.requireNonNull(callback, "callback == null");

    okhttp3.Call call;
    Throwable failure;

    synchronized(this) {
        if (executed) throw new IllegalStateException("Already executed.");
        executed = true;

        call = rawCall;
        failure = creationFailure;
        if (call == null && failure == null) {
            try {
                call = rawCall = createRawCall();
            } catch (Throwable t) {
                throwIfFatal(t);
                failure = creationFailure = t;
            }
        }
    }

    if (failure != null) {
        callback.onFailure(this, failure);
        return;
    }

    if (canceled) {
        call.cancel();
    }

    call.enqueue(
        new okhttp3.Callback() {
            @Override
            public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse) {
                Response < T > response;
                try {
                    response = parseResponse(rawResponse);
                } catch (Throwable e) {
                    throwIfFatal(e);
                    callFailure(e);
                    return;
                }

                try {
                    callback.onResponse(OkHttpCall.this, response);
                } catch (Throwable t) {
                    throwIfFatal(t);
                    t.printStackTrace(); // TODO this is not great
                }
            }

            @Override
            public void onFailure(okhttp3.Call call, IOException e) {
                callFailure(e);
            }

            private void callFailure(Throwable e) {
                try {
                    callback.onFailure(OkHttpCall.this, e);
                } catch (Throwable t) {
                    throwIfFatal(t);
                    t.printStackTrace(); // TODO this is not great
                }
            }
        });
}

private okhttp3.Call createRawCall() throws IOException {
    okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
    if (call == null) {
        throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}
```

createRawCall 方法中通过 OkHttpClient 调用 newCall 方法构造 Call 对象

parseResponse 方法中通过 json 格式转换器将 OkHttp 返回的 response 内容转换为接口返回类型中的泛型类型

```java
  Response < T > parseResponse(okhttp3.Response rawResponse) throws IOException {
      ResponseBody rawBody = rawResponse.body();

      // Remove the body's source (the only stateful object) so we can pass the response along.
      rawResponse =
          rawResponse
          .newBuilder()
          .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
          .build();

      int code = rawResponse.code();
      if (code < 200 || code >= 300) {
          try {
              // Buffer the entire body to avoid future I/O.
              ResponseBody bufferedBody = Utils.buffer(rawBody);
              return Response.error(bufferedBody, rawResponse);
          } finally {
              rawBody.close();
          }
      }

      if (code == 204 || code == 205) {
          rawBody.close();
          return Response.success(null, rawResponse);
      }

      ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
      try {
          T body = responseConverter.convert(catchingBody);
          return Response.success(body, rawResponse);
      } catch (RuntimeException e) {
          // If the underlying source threw an exception, propagate that rather than indicating it was
          // a runtime exception.
          catchingBody.throwIfCaught();
          throw e;
      }
  }
```

通过自定义的 Callback 实现结果回调，完成对 OkHttp 的封装

#### 实现Retrofit

Retrofit 主要用到了动态代理、反射、注解（未完成json转换这一过程）

https://github.com/nosleepy/MyRetrofit

#### 参考

+ [Retrofit动态代理+注解+反射简析](https://blog.csdn.net/niuyongzhi/article/details/125862776)
+ [Android技能树 — 网络小结(7)之 Retrofit源码详细解析](https://juejin.cn/post/6844903746414067719#heading-6)
+ [JDK动态代理](https://blog.csdn.net/weixin_46666822/article/details/125334448)
+ [Retrofit 源码深度解析-（3）Invoke](https://www.cnblogs.com/gonzo/articles/15267861.html)
+ [ParameterizedType理解笔记](https://www.cnblogs.com/Meiwah/p/10434893.html)
