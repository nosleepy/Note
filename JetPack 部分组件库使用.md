---
title: JetPack 部分组件库使用
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

## Jetpack

Jetpack 是一套组件库，可帮助开发人员遵循最佳实践，减少样板代码并编写可在 Android 版本和设备上一致工作的代码，以便开发人员可以专注于他们关心的代码。

![](https://upload-images.jianshu.io/upload_images/1930161-54784907605fa6a1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## LiveData

LiveData是一个可被观察的数据容器类。什么意思呢？我们可以将LiveData理解为一个数据的容器，它将数据包装起来，使得数据成为“被观察者”，页面成为“观察者”。这样，当该数据发生变化时，页面能够获得通知，进而更新UI。

ViewModel用于存放页面所需的各种数据，它还包括一些业务逻辑等。

```java
class MainViewModel : ViewModel() {
    private var _name = MutableLiveData<String>() // 可变对象
    var name: LiveData<String> = _name // 不可变对象
    fun change() {
        _name.value = "grandstream"
    }
}
```

点击按钮更改livedata的值弹出提示

```java
class MainActivity : AppCompatActivity() {
    private var mainViewModel: MainViewModel by viewModels()
    private lateinit var button: Button
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button = findViewById(R.id.button)
        button.setOnClickListener {
            mainViewModel.change()
        }
        mainViewModel.name.observe(this, {
            Toast.makeText(this, it, 0).show()
        })
    }
}
```

非UI线程使用

```java
mainViewModel.postValue(value)
```

## Room

Room是Google推出的Android架构组件库中的数据持久化组件库, 也可以说是在SQLite上实现的一套ORM解决方案。Room主要包含三个部分：Database : 持有DB和DAO；Entity : 定义POJO类，即数据表结构；DAO(Data Access Objects) : 定义访问数据（增删改查）的接口。

app/build.gradle添加依赖

```java
apply plugin: 'kotlin-kapt'
dependencies {
    def room_version = "2.2.5"
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
    implementation "androidx.room:room-ktx:$room_version"
}
```

entity层

```java
@Entity(tableName = "Book")
class Book{
    @PrimaryKey()
    var id :Int = 0
    @ColumnInfo(name = "book_name")
    var bookName:String? = null
    var isbn:String? = null
    var anchor:String? = null
    @Ignore
    var price :Int = 0
    override fun toString(): String {
        return "Book(id=$id, bookName=$bookName, isbn=$isbn, anchor=$anchor, price=$price)"
    }
}
```

dao层

```java
@Dao
abstract class BookDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    abstract fun insert(book: Book)
    @Delete
    abstract fun delete(book: Book)
    @Update
    abstract fun update(book: Book)
    @Query("select * from Book where id =:id")
    abstract fun queryById(id:Int): Book
    @Query("select * from Book")
    abstract fun queryAll(): List<Book>
    @Insert
    abstract fun insert(books: List<Book>)
    @Delete
    abstract fun delete(vararg books: Book)
    @Query("select book_name from Book")
    abstract fun queryAllBookName(): List<String?>
    @Query("select count(*) from Book")
    abstract fun bookCount(): Int
}
```

database层

```java
@Database(entities = [Book::class],version = 1)
abstract class AppDatabase : RoomDatabase(){
    abstract fun bookDao(): BookDao
}
```

room 测试

```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val appDb = Room.databaseBuilder(this, AppDatabase::class.java, "myDb.db")
            .addMigrations(object : Migration(1, 2){
                override fun migrate(database: SupportSQLiteDatabase) {
                    Log.d(TAG, "migrate")
                }
            })
            .allowMainThreadQueries()
            .build()
        val bookDao = appDb.bookDao()
        val ludingji = Book()
        ludingji.id = 1
        ludingji.bookName = "《鹿鼎记》"
        val tianlongbabu = Book()
        tianlongbabu.id = 2
        tianlongbabu.bookName = "《天龙八部》"
        bookDao.insert(ludingji)
        bookDao.insert(tianlongbabu)
        Log.d(TAG, bookDao.queryAll().toString())
        Log.d(TAG, bookDao.queryById(1).toString())
        bookDao.delete(ludingji)
        Log.d(TAG, bookDao.queryAll().toString())
        tianlongbabu.anchor = "金庸"
        bookDao.update(tianlongbabu)
        Log.d(TAG, bookDao.queryById(2).toString())
    }
    companion object {
        private const val TAG = "wlzhou"
    }
}
```

## Coroutine

协程是 Kotlin 中一个重要的特性支持，而 Kotlin 协程的支持，底层依托于虚拟机的特性。它与线程的关系，依然是 1:1 对应的。而不是类似 Go 语言这种，真的存在更小的执行体，是一种轻量级线程。Kotlin 的协程，可以理解为一种类似线程池的封装，每个协程执行的背后，都依托于一个线程。而它与线程池相比的优势，在于用更精炼的代码，利用阻塞的思想写出非阻塞式的代码。

添加依赖

```xml
dependencies {
    // Kotlin
    implementation "org.jetbrains.kotlin:kotlin-stdlib:1.4.32"
    // 协程核心库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.3"
    // 协程Android支持库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.4.3"
    // 协程Java8支持库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-jdk8:1.4.3"
    // lifecycle对于协程的扩展封装
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
}
```

**suspend 挂起函数**

```java
// 参考 https://blog.csdn.net/cpcpcp123/article/details/111724079
// suspend function方法可以被挂起和在任务完成后恢复，处理耗时任务
// 使用阻塞的思想写出非阻塞式的代码
fun test() {
    GlobalScope.launch { // 子线程运行
        val arg1 = suspendF1()
        var arg2 = suspendF2()
        log("suspend finish arg1:$arg1, arg2:$arg2, result:${arg1 + arg2}")
    }
}
private suspend fun suspendF1(): Int {
    delay(2000) // delay也是挂起函数，模拟耗时操作
    log("suspend fun 1")
    return 2
}
private suspend fun suspendF2(): Int {
    delay(4000)
    log("suspend fun 2")
    return 4
}
```

logcat日至打印结果

```xml
D/wlzhou: suspend fun 1 // 延迟2秒打印
D/wlzhou: suspend fun 2 // 延迟4秒打印
D/wlzhou: suspend finish arg1:2, arg2:4, result:6
```

**创建协程的方式**

```java
CoroutineScope.launch() // 不能返回结果
CoroutineScope.async() // 获取返回值和处理并发
```

launch 方式创建协程

```java
class MainActivity : AppCompatActivity() {
    /**
     * 使用官方库的 MainScope()获取一个协程作用域用于创建协程
     */
    private val mScope = MainScope()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // 创建一个默认参数的协程，其默认的调度模式为Main 也就是说该协程的线程环境是Main线程
        val job1 = mScope.launch {
            // 这里就是协程体
            // 延迟1000毫秒  delay是一个挂起函数
            // 在这1000毫秒内该协程所处的线程不会阻塞
            // 协程将线程的执行权交出去，该线程该干嘛干嘛，到时间后会恢复至此继续向下执行
            delay(1000)
        }
        // 创建一个指定了调度模式的协程，该协程的运行线程为IO线程
        val job2 = mScope.launch(Dispatchers.IO) {
            // 此处是IO线程模式
            // 切线程 将协程所处的线程环境切至指定的调度模式Main
            withContext(Dispatchers.Main) {
                // 现在这里就是Main线程了  可以在此进行UI操作了
            }
        }
        // 下面直接看一个例子： 从网络中获取数据  并更新UI
        // 该例子不会阻塞主线程
        mScope.launch(Dispatchers.IO) {
            // 执行getUserInfo方法时会将线程切至IO去执行
            val userInfo = getUserInfo()
            // 获取完数据后 切至Main线程进行更新UI
            withContext(Dispatchers.Main) {
                // 更新UI
            }
        }
    }
    /**
     * 获取用户信息 该函数模拟IO获取数据
     * @return String
     */
    private suspend fun getUserInfo(): String {
        return withContext(Dispatchers.IO) {
            delay(2000)
            "Kotlin"
        }
    }
    override fun onDestroy() {
        super.onDestroy()
        // 取消协程 防止协程泄漏  如果使用lifecycleScope则不需要手动取消
        mScope.cancel()
    }
}
```

async 方式创建协程

```java
fun asyncTest1() {
    mScope.launch {
        // 开启一个IO模式的线程 并返回一个Deferred，Deferred可以用来获取返回值
        // 代码执行到此处时会新开一个协程 然后去执行协程体  父协程的代码会接着往下走
        val deferred = async(Dispatchers.IO) {
            // 模拟耗时
            delay(2000)
            // 返回一个值
            "Quyunshuo"
        }
        // 等待async执行完成获取返回值 此处并不会阻塞线程  而是挂起 将线程的执行权交出去
        // 等到async的协程体执行完毕后  会恢复协程继续往下执行
        val date = deferred.await()
    }
}
```

协程并发能力

```java
fun asyncTest2() {
    mScope.launch {
        // 此处有一个需求  同时请求5个接口  并且将返回值拼接起来
        val job1 = async {
            // 请求1 
            delay(2000)
            log("job1 2s")
            "2"
        }
        val job2 = async {
            // 请求2 
            delay(4000)
            log("job2 4s")
            "4"
        }
        val job3 = async {
            // 请求3 
            delay(6000)
            log("job3 6s")
            "6"
        }
        log("start")
        // 代码执行到此处时  3个请求已经同时在执行了
        // 等待各job执行完 将结果合并
        log("asyncTest2: ${job1.await()} ${job2.await()} ${job3.await()}")
        // 最耗时请求job3执行3秒，所以3个请求处理完成也是耗时3秒
    }
}
```

并发处理结果日至打印

```java
10:48:52.137 D/wlzhou: start
10:48:54.139 D/wlzhou: job1 2s // 2秒后打印
10:48:56.141 D/wlzhou: job2 4s // 2秒后打印
10:48:58.140 D/wlzhou: job3 6s // 2秒后打印
10:48:58.141 D/wlzhou: asyncTest2: 2 4 6 // 总共耗时6秒完成
```

**suspendCancellableCoroutine**

```java
// 在kotlin之前，异步请求往往会采用回调函数，将异步线程的数据回调回主线程。
// 在kotlin中则可以使用suspendCancellableCoroutine将回调函数转换成协程。

MainScope().launch {
  try {
    val user = fetchUser()
    updateUser(user)
  } catch (exception: Exception) {
    // 捕获resumeWithException抛出的异常
  }
}
private suspend fun fetchUser(): User = suspendCancellableCoroutine { 
  cancellableContinuation ->
  // 异步网络请求
  fetchUserFromNetwork(object : Callback {
    override fun onSuccess(user: User) {
      // 回调获得的数据，最后会被fetchUser()方法声明的val user变量接收到
      cancellableContinuation.resume(user)
    }
    override fun onFailure(exception: Exception) {
      //网络请求失败，抛出异常会被上面的try/catch块捕获
      cancellableContinuation.resumeWithException(exception)
    }
  })
}
private fun fetchUserFromNetwork(callback: Callback) {
  Thread {
    Thread.sleep(3_000)
    //模拟网路响应的回调
    callback.onSuccess(User())
  }.start()
}
private fun updateUser(user: User) {
  // 更新ui
}
interface Callback {
  fun onSuccess(user: User)
  fun onFailure(exception: Exception)
}
class User 
```

## Channel

Channel 翻译过来为通道或者管道，实际上就是个队列, 是一个面向多协程之间数据传输的 BlockQueue，用于协程间通信。Channel 允许我们在不同的协程间传递数据。形象点说就是不同的协程可以往同一个管道里面写入数据或者读取数据。

通道定义

```java
val channnel = Channel<String>(Channel.UNLIMITED)
```

写入数据

```java
viewModelScope.launch {
    channel.send("hello")
}
```

读取数据

```java
//way1
lifecycleScope.launch {
    channel.consumeAsFlow().collect { // 
        log(it)
  }
}

// way2
lifecycleScope.launch {
    val result = channel.receive()
}
```

## LifeCycle

Lifecycle可以让某一个类变成Activity、Fragment的生命周期观察者类，监听其生命周期的变化并可以做出响应。Lifecycle使得代码更有条理性、精简、易于维护。 

定义生命周期观察者 MyLifecycleObserver 实现  DefaultLifecycleObserver 接口

```java
class MyLifecycleObserver : DefaultLifecycleObserver {
    override fun onCreate(owner: LifecycleOwner) {
        super.onCreate(owner)
        Log.d(TAG, "onCreate")
    }
    override fun onStart(owner: LifecycleOwner) {
        super.onStart(owner)
        Log.d(TAG, "onStart")
    }
    override fun onResume(owner: LifecycleOwner) {
        super.onResume(owner)
        Log.d(TAG, "onResume")
    }
    override fun onPause(owner: LifecycleOwner) {
        super.onPause(owner)
        Log.d(TAG, "onPause")
    }
    override fun onStop(owner: LifecycleOwner) {
        super.onStop(owner)
        Log.d(TAG, "onStop")
    }
    override fun onDestroy(owner: LifecycleOwner) {
        super.onDestroy(owner)
        Log.d(TAG, "onDestroy")
    }
}
```

生命周期拥有者 Activity/Fragment 已实现 LifecycleOwner 接口

```java
lifecycle.addObserver(new MyLifecycleObserver())
```

## Flow

Kotlin 协程中使用挂起函数可以实现非阻塞地执行任务并将结果返回回来，但是只能返回单个计算结果。但是如果希望有多个计算结果返回回来，则可以使用 Flow。

flow 冷流

```java
// 冷流，只有调用collect方法 flow{} 中的代码才会执行
private fun createFlow(): Flow<Int> = flow {
    delay(1000)
    emit(1) // 发射数据
    delay(1000)
    emit(2)
    delay(1000)
    emit(3)
}

fun main() = runBlocking {
    createFlow().collect { // 收集结果
        println(it)
    }
}

// 运行结果
2022-06-23 11:20:25.616 D/wlzhou: 1
2022-06-23 11:20:26.617 D/wlzhou: 2
2022-06-23 11:20:27.622 D/wlzhou: 3
```

flow 热流

```java
// 热流，只要上游产生了数据，就会立即发射给下游的收集者。
StateFlow // 只会向订阅者发射最新的值，适用于对状态的监听
SharedFlow // 可以配置对历史发射的数据进行订阅，适合用来处理对于事件的监听
```

StateFlow 使用

```java
private val state = MutableStateFlow<String>("hello") // 有初始值

lifecycleScope.launch {     
    launch(Dispatchers.IO) {
        delay(3000)
        state.emit("world")
        state.emit("android")
    }
    launch(Dispatchers.IO) {
        state.collect {
            log("${Thread.currentThread().name} -> $it")
        }
    }
    launch(Dispatchers.IO) {
        state.collect {
            log("${Thread.currentThread().name} -> $it")
        }
    }
}

// 运行结果
2022-06-23 14:28:43.831 D/wlzhou: DefaultDispatcher-worker-2 -> hello
2022-06-23 14:28:43.831 D/wlzhou: DefaultDispatcher-worker-1 -> hello
// 延迟3秒，没有打印world
2022-06-23 14:28:46.828 D/wlzhou: DefaultDispatcher-worker-6 -> android
2022-06-23 14:28:46.828 D/wlzhou: DefaultDispatcher-worker-2 -> android
```

SharedFlow 使用

```java
private val state = MutableSharedFlow<String>() // 没有初始值

lifecycleScope.launch {
    launch(Dispatchers.IO) {
        delay(3000)
        state.emit("hello")
        state.emit("world")
    }
    launch(Dispatchers.IO) {
        state.collect {
            log("${Thread.currentThread().name} -> $it")
        }
    }
    launch(Dispatchers.IO) {
        state.collect {
            log("${Thread.currentThread().name} -> $it")
        }
    }
}

// 运行结果，发射的数据都会打印
2022-06-23 14:32:50.192 D/wlzhou: DefaultDispatcher-worker-4 -> hello
2022-06-23 14:32:50.192 D/wlzhou: DefaultDispatcher-worker-3 -> hello
2022-06-23 14:32:50.193 D/wlzhou: DefaultDispatcher-worker-3 -> world
2022-06-23 14:32:50.193 D/wlzhou: DefaultDispatcher-worker-4 -> world
```

## 参考

+ [《安卓-深入浅出MVVM教程》应用篇-03 Cache （本地缓存）](https://www.jianshu.com/p/cf9482d71241)
+ [一文带你了解Room数据库_三分钟Code的博客-CSDN博客_room数据库](https://blog.csdn.net/weixin_38261570/article/details/111500338)
+ [Kotlin flow实践总结](https://blog.csdn.net/YoungOne2333/article/details/123018573)
+ [Android-Transformations.switchMap 理解](https://blog.csdn.net/weixin_43928944/article/details/119897897)
+ [《安卓-深入浅出MVVM教程》应用篇-02 Repository （数据仓库）](https://juejin.cn/post/6844903505635835911)
+ [kotlin Suspend Function](https://zhuanlan.zhihu.com/p/362604283)
+ [一篇文章带你了解——Kotlin协程](https://zhuanlan.zhihu.com/p/427092689)
+ [Android必学之数据适配器BaseAdapter](https://www.cnblogs.com/caobotao/p/5061627.html)
