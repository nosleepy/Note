---
title: ViewModel组件原理分析
date: 2023-01-01 19:06:13
tags:
- JetPack
categories:
- 安卓
---

#### 简介

+ ViewModel 类被设计用于以感知生命周期的方式来管理和存储 UI 界面相关的数据，例如当屏幕发生旋转、App权限被动态修改、系统语言发生改变（这些都属于 Configuration changes）的时候，Activity 重建的时候，ViewModel 可以保留界面的数据。
+ 在没有 ViewModel 之前，一般我们是在 onSaveInstanceState() 方法中将数据保存在 Bundle 中，然后在 onCreate() 方法中判断 Bundle 是否为空，不为空则从中获取保存的数据，如果被保存的数据是复杂类型的话还需要实现 Serializable 或 Parcelable 接口，所以这种方式保存和恢复数据比较麻烦也不适合保存大量的数据。
+ 有了 ViewModel 之后，一切都变得简单，我们无需手动的在某个特定的方法中保存数据，保存的动作系统帮我们自动完成了，而且 ViewModel 可以保留任何对象，如果是复杂类型不需要实现Serializable 或 Parcelable 接口。

#### 使用

添加 kotlin 插件，引入相关依赖

```groovy
//project/build.gradle
plugins {
    id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
}
```

```groovy
//project/app/build.gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-android-extensions'
}

dependencies {
    implementation 'androidx.core:core-ktx:1.7.0'
}
```

实现点击按钮 count 数量加一、横竖屏切换数量不变的案例

<img src="https://raw.githubusercontent.com/nosleepy/picture/master/img/viewmodel_add_count_demo.png" style="zoom:20%;" />

布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/count"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"/>

    <Button
        android:id="@+id/add"
        android:text="Add"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        tools:ignore="HardcodedText" />

</RelativeLayout>
```

新建 MainViewModel 继承 ViewModel

```kotlin
class MainViewModel(var count: Int) : ViewModel() {
}
```

MainViewModel 不能通过 new 的方式创建，需要由 ViewModelProvider.Factory 工厂去生成

新建 MainViewModelFactory 继承 ViewModelProvider.Factory，重写 create 方法生成 ViewModel 对象

```kotlin
class MainViewModelFactory(private val count: Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return MainViewModel(count) as T
    }
}
```

示例代码

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var mainViewModel: MainViewModel
    private lateinit var sp: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        sp = getPreferences(Context.MODE_PRIVATE)
        val countReserved = sp.getInt("count_reserved", 0)
        mainViewModel = ViewModelProvider(this, MainViewModelFactory(countReserved)).get(MainViewModel::class.java)
        add.setOnClickListener {
            count.text = "${++mainViewModel.count}"
        }
        count.text = "${mainViewModel.count}"
    }

    override fun onPause() {
        super.onPause()
        sp.edit().putInt("count_reserved", mainViewModel.count).apply()
    }
}
```

#### 原理分析

ViewModel 对象通过 ViewModelProvider 对象的 get 方法来获取，先来看一下 ViewModelProvider 的构造方法

```kotlin
public open class ViewModelProvider
@JvmOverloads
constructor(
    private val store: ViewModelStore,
    private val factory: Factory,
    private val defaultCreationExtras: CreationExtras = CreationExtras.Empty,
) {
    public constructor(
        owner: ViewModelStoreOwner
    ) : this(owner.viewModelStore, defaultFactory(owner), defaultCreationExtras(owner))
    
    public constructor(owner: ViewModelStoreOwner, factory: Factory) : this(
        owner.viewModelStore,
        factory,
        defaultCreationExtras(owner)
    )
}
```

owner.viewModelStore 得到 ViewModelStore 对象，内部封装了一个 HashMap 来存储 ViewModel

```java
public class ViewModelStore {
    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    Set<String> keys() {
        return new HashSet<>(mMap.keySet());
    }

    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.clear();
        }
        mMap.clear();
    }
}
```

最终调用三参构造方法，ComponentActivity 实现了 ViewModelStoreOwner 接口，AppCompatActivity 间接继承 ComponentActivity，可以使用 this 关键字传入

```java
public interface ViewModelStoreOwner {
    @NonNull
    ViewModelStore getViewModelStore();
}

public class ComponentActivity extends androidx.core.app.ComponentActivity implements ViewModelStoreOwner {
    private ViewModelStore mViewModelStore;
    
	@NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        ensureViewModelStore();
        return mViewModelStore;
    }
    
    void ensureViewModelStore() {
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
    }
    
    static final class NonConfigurationInstances {
        Object custom;
        ViewModelStore viewModelStore;
    }
}
```

getLastNonConfigurationInstance() 方法获取 NonConfigurationInstances 对象，如果不为空，mViewModelStore 进行赋值，如果还是为空，则实例化 ViewModelStore 对象

```java
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback,
        ContentCaptureManager.ContentCaptureClient {
    
    NonConfigurationInstances mLastNonConfigurationInstances;
        
    @Nullable
    public Object getLastNonConfigurationInstance() {
        return mLastNonConfigurationInstances != null
                ? mLastNonConfigurationInstances.activity : null;
    }
            
    static final class NonConfigurationInstances {
        Object activity;
        HashMap<String, Object> children;
        FragmentManagerNonConfig fragments;
        ArrayMap<String, LoaderManager> loaders;
        VoiceInteractor voiceInteractor;
    }
 }
```

NonConfigurationInstances 就是 mLastNonConfigurationInstances.activity，通过 onRetainNonConfigurationInstance() 方法返回

```java
//ComponentActivity.java
    @Override
    @Nullable
    @SuppressWarnings("deprecation")
    public final Object onRetainNonConfigurationInstance() {
        // Maintain backward compatibility.
        Object custom = onRetainCustomNonConfigurationInstance();

        ViewModelStore viewModelStore = mViewModelStore;
        if (viewModelStore == null) {
            // No one called getViewModelStore(), so see if there was an existing
            // ViewModelStore from our last NonConfigurationInstance
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                viewModelStore = nc.viewModelStore;
            }
        }

        if (viewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        nci.viewModelStore = viewModelStore;
        return nci;
    }
```

而 mLastNonConfigurationInstances 变量在 attach 方法进行赋值

```java
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback,
        ContentCaptureManager.ContentCaptureClient {
    NonConfigurationInstances mLastNonConfigurationInstances;
        
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
            IBinder shareableActivityToken) {
            mLastNonConfigurationInstances = lastNonConfigurationInstances;
    }
 }
```

Activity 重建的时候会调用 attach 方法，将 lastNonConfigurationInstances 变量赋值给 mLastNonConfigurationInstances 成员变量

```java
public final class ActivityThread extends ClientTransactionHandler
        implements ActivityThreadInternal {
        private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        	activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.activityConfigCallback,
                        r.assistToken, r.shareableActivityToken);
        }
    
    	public static final class ActivityClientRecord {
            Activity.NonConfigurationInstances lastNonConfigurationInstances;
        }
 }
```

ActivityThread 的内部类 ActivityClientRecord 来存储 lastNonConfigurationInstances，找到 performDestroyActivity 方法

```java
//ActivityThread.java
void performDestroyActivity(ActivityClientRecord r, boolean finishing,
            int configChanges, boolean getNonConfigInstance, String reason) {
        if (getNonConfigInstance) {
            try {
                r.lastNonConfigurationInstances = r.activity.retainNonConfigurationInstances();
            } catch (Exception e) {
                if (!mInstrumentation.onException(r.activity, e)) {
                    throw new RuntimeException("Unable to retain activity "
                            + r.intent.getComponent().toShortString() + ": " + e.toString(), e);
                }
            }
        }
    }
```

r.lastNonConfigurationInstances 通过 retainNonConfigurationInstances 方法来返回

```java
//Activity.java
NonConfigurationInstances retainNonConfigurationInstances() {
        Object activity = onRetainNonConfigurationInstance();
        HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
        FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();
        mFragments.doLoaderStart();
        mFragments.doLoaderStop(true);
        ArrayMap<String, LoaderManager> loaders = mFragments.retainLoaderNonConfig();

        if (activity == null && children == null && fragments == null && loaders == null
                && mVoiceInteractor == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.activity = activity;
        nci.children = children;
        nci.fragments = fragments;
        nci.loaders = loaders;
        if (mVoiceInteractor != null) {
            mVoiceInteractor.retainInstance();
            nci.voiceInteractor = mVoiceInteractor;
        }
        return nci;
    }
```

界面销毁的时候通过 retainNonConfigurationInstances 来保存数据

重新回到 ViewModelProvider#get 方法

```java
	//ViewModelProvider.kt
    @MainThread
    public open operator fun <T : ViewModel> get(modelClass: Class<T>): T {
        val canonicalName = modelClass.canonicalName
            ?: throw IllegalArgumentException("Local and anonymous classes can not be ViewModels")
        return get("$DEFAULT_KEY:$canonicalName", modelClass)
    }
    
    @Suppress("UNCHECKED_CAST")
    @MainThread
    public open operator fun <T : ViewModel> get(key: String, modelClass: Class<T>): T {
        val viewModel = store[key]
        if (modelClass.isInstance(viewModel)) {
            (factory as? OnRequeryFactory)?.onRequery(viewModel)
            return viewModel as T
        } else {
            @Suppress("ControlFlowWithEmptyBody")
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        val extras = MutableCreationExtras(defaultCreationExtras)
        extras[VIEW_MODEL_KEY] = key有就返回，没有通过 Factory 去生成
        return try {
            factory.create(modelClass, extras)
        } catch (e: AbstractMethodError) {
            factory.create(modelClass)
        }.also { store.put(key, it) }
    }
```

以 $DEFAULT_KEY:$canonicalName 作为 key 去 ViewModelStore 中查询对应的 ViewModel，有就返回，没有通过 Factory 去生成

```java
	//ViewModelProvider.Factory.java
    public interface Factory {
        public fun <T : ViewModel> create(modelClass: Class<T>): T {
            throw UnsupportedOperationException(
                "Factory.create(String) is unsupported.  This Factory requires " +
                    "`CreationExtras` to be passed into `create` method."
            )
        }

        public fun <T : ViewModel> create(modelClass: Class<T>, extras: CreationExtras): T =
            create(modelClass)

        companion object {
            @JvmStatic
            fun from(vararg initializers: ViewModelInitializer<*>): Factory =
                InitializerViewModelFactory(*initializers)
        }
    }
```

#### 原理总结

+ 在 configuration changes 的时候，在界面 destroy 的时候（ActivityThread.performDestroyActivity）通过 onRetainNonConfigurationInstance 方法将 NonConfigurationInstances 对象（里面包含 viewModelStore） 保存起来。
+ 在重建 Activity 的时候在 attach 方法中（ActivityThread.performLaunchActivity）将上次保存的 NonConfigurationInstances 对象赋值给 Activity 的成员变量 mLastNonConfigurationInstances，然后在 onCreate 中通过 getLastNonConfigurationInstance 方法获取上次保存的 NonConfigurationInstances 对象。

#### 参考

+ [ViewModel组件原理剖析](https://juejin.cn/post/6930195735719870471#heading-3)
