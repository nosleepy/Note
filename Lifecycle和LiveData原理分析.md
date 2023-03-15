---
title: Lifecycle和LiveData原理分析
date: 2023-01-01 19:06:13
tags:
- JetPack
categories:
- 安卓
---

#### 基本使用

布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/count"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true" />

    <Button
        android:id="@+id/add"
        android:text="Add"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</RelativeLayout>
```

自定义 MainViewModel

```java
class MainViewModel: ViewModel() {
    private val _count = MutableLiveData<Int>(0)
    val count: LiveData<Int> = _count

    fun addCount() {
        _count.value = _count.value?.plus(1)
    }
}
```

测试代码

```java
class MainActivity : AppCompatActivity() {
    private lateinit var mainViewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mainViewModel = ViewModelProvider(this)[MainViewModel::class.java]
        mainViewModel.count.observe(this) {
            Log.d(TAG, "it = $it")
            count.text = "$it"
        }
        add.setOnClickListener { mainViewModel.addCount() }
        lifecycle.addObserver(object : LifecycleEventObserver {
            override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                Log.d(TAG, "event = $event")
            }
        })
    }

    companion object {
        private const val TAG = "MainActivity"
    }
}
```

打印结果

```
MainActivity        D  event = ON_CREATE
MainActivity        D  it = 0
MainActivity        D  event = ON_START
MainActivity        D  event = ON_RESUME
MainActivity        D  it = 1
MainActivity        D  it = 2
MainActivity        D  it = 3
MainActivity        D  event = ON_PAUSE
MainActivity        D  event = ON_STOP
MainActivity        D  event = ON_DESTROY
```

#### Lifecycle原理

LifecycleObserver通过直接注册观察LifecycleOwner，当LifecycleOwner对象生命周期发生变化的时候就可以直接通知observer，Activity以及Fragment都实现了LifecycleOwner接口。这样我们的生命周期相关的回调就可以不用全都写在Activity 或者Fragment的生命周期方法里边，避免代码过度耦合。

#### LivaData原理

LiveData 是一种可观察的数据存储器类。与常规的可观察类不同，LiveData 具有生命周期感知能力，意指它遵循其他应用组件（如 activity、fragment 或 service）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

#### 参考

+ [Lifecycle原理解析，人人都能看得懂](https://blog.csdn.net/chuyouyinghe/article/details/124040555)
+ [Android Jetpack - Lifecycle使用方法及原理分析](https://juejin.cn/post/6844903997346676749#heading-9)
+ [Android Jetpack组件LiveData基本使用和原理分析](https://zhuanlan.zhihu.com/p/321667726)
