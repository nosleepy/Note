---
title: Hilt和依赖注入
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

### 什么是依赖注入

简单的说，依赖注入就是内部的类在外部实例化了。也就是不需要自己去做实例化工作了，而是交给外部容器来完成，最后注入到调用者这边，形成依赖注入。

### 引入Hilt

```java
# 项目根目录的 build.gradle 文件中配置 Hilt 的插件路径
buildscript {
    dependencies {
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}

# app/build.gradle 文件中，引入 Hilt 的插件并添加 Hilt 的依赖库，启用 Java 8 的功能
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
}

android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

### 不使用依赖注入

以卡车送货为例子，在 MainActivity 中如果想使用 Truck 卡车类来运送货物，需要手动实例化 Truck 对象，再调用对象的方法来完成送货功能。

```java
class MainActivity : AppCompatActivity() {
    var truck: Truck = Truck()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        truck.deliver()
    }
}

class Truck {
    fun deliver() {
        Log.d("wlzhou", "Truck is delivering cargo.")
    }
}
```

### Hilt的简单用法

使用 hilt 不需要手动实例化 Truck 对象，而是通过以下注解声明来进行依赖注入。

```java
// 使用依赖注入功能，入口点添加注解声明

@HiltAndroidApp // Application 入口点
class MyApplication : Application() {
}

@AndroidEntryPoint // Activity 入口点
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var truck: Truck
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        truck.deliver()
    }
}

// 告诉 Hilt，通过构造函数来实例化对象
class Truck @Inject constructor() {
    fun deliver() {
        Log.d("wlzhou", "Truck is delivering cargo.")
    }
}
```

### 带参数的依赖注入

```java
// 司机类
class Driver @Inject constructor() {
}

// 卡车依赖于司机
class Truck @Inject constructor(val driver: Driver) {
    fun deliver() {
        Log.d("wlzhou", "Truck is delivering cargo. Driven by $driver")
    }
}
```

### 接口的依赖注入

卡车需要用到引擎，定义 Engine 引擎接口，提供燃油引擎 GasEngine 和电动引擎 ElectricEngine 的实现。

```java
// 引擎接口
interface Engine {
    fun start()
    fun shutdown()
}

// 燃油引擎
class GasEngine @Inject constructor() : Engine {
    override fun start() {
        Log.d("wlzhou", "Gas engine start.")
    }
    override fun shutdown() {
        Log.d("wlzhou", "Gas engine shutdown.")
    }
}

// 电动引擎
class ElectricEngine @Inject constructor() : Engine {
    override fun start() {
        Log.d("wlzhou", "Electric engine start.")
    }
    override fun shutdown() {
        Log.d("wlzhou", "Electric engine shutdown.")
    }
}

// BindGasEngine 注解区分燃油引擎
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BindGasEngine

// BindElectricEngine 注解区分电动引擎
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BindElectricEngine

@Module
@InstallIn(ActivityComponent::class)
abstract class EngineModule {

    @BindGasEngine
    @Binds
    abstract fun bindGasEngine(gasEngine: GasEngine): Engine
    
    @BindElectricEngine
    @Binds
    abstract fun bindElectricEngine(electricEngine: ElectricEngine): Engine
}

class Truck @Inject constructor(val driver: Driver) {

    @BindGasEngine
    @Inject
    lateinit var gasEngine: Engine
    
    @BindElectricEngine
    @Inject
    lateinit var electricEngine: Engine
    
    fun deliver() {
        gasEngine.start()
        electricEngine.start()
        Log.d("wlzhou", "Truck is delivering cargo. Driven by $driver")
        gasEngine.shutdown()
        electricEngine.shutdown()
    }
}
```

### 第三方类的依赖注入

### 
MainActivity 中使用 OkHttpClient 和 Retrofit 发起网络请求。

```java
@Module
@InstallIn(ActivityComponent::class)
class NetworkModule {

    @Provides
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(20, TimeUnit.SECONDS)
            .readTimeout(20, TimeUnit.SECONDS)
            .writeTimeout(20, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .baseUrl("http://example.com/")
            .client(okHttpClient)
            .build()
    }
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var okHttpClient: OkHttpClient

    @Inject
    lateinit var retrofit: Retrofit
}
```

### 参考

[简单谈谈Hilt——依赖注入框架 ](https://www.cnblogs.com/jimuzz/p/13930844.html)

[Android Jetpack--Hilt使用详解](https://zhuanlan.zhihu.com/p/413691642)

[Jetpack新成员，一篇文章带你玩转Hilt和依赖注入](https://blog.csdn.net/guolin_blog/article/details/109787732#commentBox)
