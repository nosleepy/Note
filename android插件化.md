---
title: android插件化
date: 2023-07-11 00:00:00
tags:
categories:
- 安卓
---

新建 module 名为 plugin, 创建 Calculate 类, 包名设置为 com.gs.plugin

```java
public class Calculate {
    int add(int a, int b) {
        return a + b;
    }
}
```

打包成 plugin.apk, push 到宿主 apk 的 /data/data/com.gs.myapplication 目录

```
adb push plugin.apk /data/data/com.gs.myapplication
```

创建插件管理器 PluginManager

```java
/**
 * 插件化框架核心类
 */
public class PluginManager {

    // 类加载器,用于加载插件包 apk 中的 classes.dex 文件中的字节码对象
    private DexClassLoader mDexClassLoader;

    // 从插件包 apk 中加载的资源
    private Resources mResources;

    // 插件包信息类
    private PackageInfo mPackageInfo;

    // 加载插件的上下文对象
    private Context mContext;

    // PluginManager 单例
    private static PluginManager instance;

    private PluginManager() {

    }

    // 获取单例类
    public static PluginManager getInstance(){
        if (instance == null) {
            instance = new PluginManager();
        }
        return instance;
    }

    // 加载插件
    public void loadPlugin(Context context, String loadPath) {
        this.mContext = context;
        // 创建 DexClassLoader
        mDexClassLoader = new DexClassLoader(
                loadPath, // 加载路径
                null,
                null,
                context.getClassLoader() // DexClassLoader 加载器的父类加载器
        );
        PackageInfo pInfo = context.getPackageManager().getPackageArchiveInfo(loadPath, PackageManager.GET_ACTIVITIES);
        pInfo.applicationInfo.sourceDir = loadPath;
        pInfo.applicationInfo.publicSourceDir = loadPath;
        mPackageInfo = pInfo;
        // 获取皮肤包的 Resources
        Resources skinRes = null;
        try {
            skinRes = context.getPackageManager().getResourcesForApplication(pInfo.applicationInfo);
        } catch (PackageManager.NameNotFoundException e) {
            throw new RuntimeException(e);
        }
        // 获取 app 的 Resources 用于获取 displayMetrics、configuration
        Resources res = context.getResources();
        // 构造出新的皮肤包 Resources； skinRes.assets 是皮肤包的 AssetManager
        mResources = new Resources(skinRes.getAssets(), res.getDisplayMetrics(), res.getConfiguration());
    }

    public DexClassLoader getDexClassLoader() {
        return mDexClassLoader;
    }

    public Resources getResources() {
        return mResources;
    }

    public PackageInfo getPackageInfo() {
        return mPackageInfo;
    }

    public Context getContext() {
        return mContext;
    }
}
```

#### 加载插件中的类

通过反射调用 Calculate 类的 add 方法

```java
PluginManager.getInstance().loadPlugin(getApplicationContext(), "/data/data/com.gs.myapplication/plugin.apk");
try {
    Class<?> clazz = PluginManager.getInstance().getDexClassLoader().loadClass("com.gs.plugin.Calculate");
    Method addMethod = clazz.getMethod("add", int.class, int.class);
    int result = (int) addMethod.invoke(clazz.newInstance(), 1, 2);
    Log.d("wlzhou", "add(1, 2) = " + result);
} catch (Exception e) {
    e.printStackTrace();
}

// 2023-07-10 18:22:55.016 22733-22733 wlzhou                  com.gs.myapplication                 D  add(1, 2) = 3
```

#### 加载插件中的资源

plugin 模块 strings.xml 文件中 app_name 的值是 plugin

```xml
<resources>
    <string name="app_name">plugin</string>
</resources>
```

通过 Resources 加载 R.string.app_name 对应的字符串值

```java
PluginManager.getInstance().loadPlugin(getApplicationContext(), "/data/data/com.gs.myapplication/plugin.apk");
String appName = PluginManager.getInstance().getResources().getString(R.string.app_name); Log.d("wlzhou", "appName = " + appName);

// 2023-07-10 18:37:37.734 25149-25149 wlzhou                  com.gs.myapplication                 D  appName = plugin
```

#### 宿主apk和插件apk合并dex文件

创建一个 BugClass 类显示 bug 信息

```java
public class BugClass {
    public String getTitle(){
        return "这是个Bug";
    }
}
```

新建 module, 创建同包下的 BugClass 类显示正确信息

```java
public class BugClass {
    public String getTitle(){
        return "修复成功";
    }
}
```

打包成 plugin.apk,  push 到 /data/data/com.gs.myapplication 目录

```
adb push plugin.apk /data/data/com.gs.myapplication
```

合并 dex 文件

```java
public class TinkerUtil {
    public void loadDexAndInject(Context appContext, String dexPath, String dexOptPath) {
        try {
            // 加载应用程序dex的Loader
            PathClassLoader pathLoader = (PathClassLoader) appContext.getClassLoader();
            //dexPath 补丁dex文件所在的路径
            //dexOptPath 补丁dex文件被写入后存放的路径
            DexClassLoader dexClassLoader = new DexClassLoader(dexPath, dexOptPath, null, pathLoader);
            //利用反射获取DexClassLoader和PathClassLoader的pathList属性
            Object dexPathList = getPathList(dexClassLoader);
            Object pathPathList = getPathList(pathLoader);
            //同样用反射获取DexClassLoader和PathClassLoader的dexElements属性
            Object leftDexElements = getDexElements(dexPathList);
            Object rightDexElements = getDexElements(pathPathList);
            //合并两个数组，且补丁包的dex文件在数组的前面
            Object dexElements = combineArray(leftDexElements, rightDexElements);
            //反射将合并后的数组赋值给PathClassLoader的pathList.dexElements
            Object pathList = getPathList(pathLoader);
            Class<?> pathClazz = pathList.getClass();
            Field declaredField = pathClazz.getDeclaredField("dexElements");
            declaredField.setAccessible(true);
            declaredField.set(pathList, dexElements);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static Object getPathList(Object classLoader) throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
        Class<?> cl = Class.forName("dalvik.system.BaseDexClassLoader");
        Field field = cl.getDeclaredField("pathList");
        field.setAccessible(true);
        return field.get(classLoader);
    }


    private static Object getDexElements(Object pathList) throws NoSuchFieldException, IllegalAccessException {
        Class<?> cl = pathList.getClass();
        Field field = cl.getDeclaredField("dexElements");
        field.setAccessible(true);
        return field.get(pathList);
    }

    private static Object combineArray(Object arrayLeft, Object arrayRight) {
        Class<?> clazz = arrayLeft.getClass().getComponentType();
        int i = Array.getLength(arrayLeft);
        int j = Array.getLength(arrayRight);
        int k = i + j;
        Object result = Array.newInstance(clazz, k);// 创建一个类型为clazz，长度为k的新数组
        System.arraycopy(arrayLeft, 0, result, 0, i);
        System.arraycopy(arrayRight, 0, result, i, j);
        return result;
    }
}
```

Application 的 onCreate 方法中判断补丁文件是否存在, 然后执行 dex 合并

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        String apkPath = "/data/data/com.gs.myapplication/plugin.apk";
        File file = new File(apkPath);
        if (file.exists()) {
            new TinkerUtil().loadDexAndInject(getApplicationContext(), apkPath, null);
        }
    }
}
```

Activity 中通过反射获取 BugClass 类中 getTitle 方法返回的信息

```java
public class MainActivity extends AppCompatActivity {
    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = findViewById(R.id.tv);
        try {
            Class<?> clazz = Class.forName("com.gs.myapplication.BugClass");
            Method addMethod = clazz.getMethod("getTitle");
            String result = (String) addMethod.invoke(clazz.newInstance());
            textView.setText(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

替换前

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/fix_before.png)

替换后

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/fix_after.png)

#### 加载插件中的Activity

插件 module 中创建一个 PluginActivity

```java
public class PluginActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_plugin);
    }
}
```

1. 使用占坑 Activity

宿主 app 的清单文件注册这个插件 Activity 信息

```xml
<activity android:name="com.gs.plugin.PluginActivity"/>
```

使用包名加类名的方式启动 activity

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        PluginManager.getInstance().loadPlugin(getApplicationContext(), "/data/data/com.gs.myapplication/plugin.apk");
        Class<?> clazz = null;
        try {
            clazz = PluginManager.getInstance().getDexClassLoader().loadClass("com.gs.plugin.PluginActivity");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.gs.plugin","com.gs.plugin.PluginActivity"));
        startActivity(intent);
    }
}
```

2. 使用 hook 方式

+ Hook AMS

```java
public static void HookAMS() {

    try {
        Class<?> atmClazz = Class.forName("android.app.ActivityTaskManager");
        Field iActivityTaskManagerSingletonField = atmClazz.getDeclaredField("IActivityTaskManagerSingleton");
        iActivityTaskManagerSingletonField.setAccessible(true);
        Object atmSingleton = iActivityTaskManagerSingletonField.get(null);

        Class<?> singletonClazz = Class.forName("android.util.Singleton");
        Field mInstanceField = singletonClazz.getDeclaredField("mInstance");
        mInstanceField.setAccessible(true);
        final Object atmService = mInstanceField.get(atmSingleton);

        Class<?> proxyClazz = Class.forName("android.app.IActivityTaskManager");
        Object newProxyInstance = Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{proxyClazz}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                if (method.getName().equals("startActivity")) {
                    int targetIndex = 0;
                    //修改Intent的值
                    for (int i = 0; i < args.length; i++) {
                        if (args[i] instanceof Intent) {
                            targetIndex = i;
                            break;
                        }
                    }
                    Intent oldIntent = (Intent) args[targetIndex];
                    Intent proxyIntent = new Intent();
                    proxyIntent.setComponent(new ComponentName("com.lay.image_process", "com.lay.image_process.MainActivity2"));
                    proxyIntent.putExtra("old_intent", oldIntent);
                    args[targetIndex] = proxyIntent;
                }
                Log.e("TAG", "atmService=" + atmService + " method=" + method + ", args=" + args);
                return method.invoke(atmService, args);
            }
        });

        mInstanceField.set(atmSingleton, newProxyInstance);

    } catch (Exception e) {
        Log.e("TAG", "e--" + e.getMessage());
        e.printStackTrace();
    }
}
```

+ Hook Handler

```java
public static void hookHandler() {

        try {

            Class<?> activityThreadClazz = Class.forName("android.app.ActivityThread");
            //获取到ActivityThread对象
            Field sCurrentActivityThreadField = activityThreadClazz.getDeclaredField("sCurrentActivityThread");
            sCurrentActivityThreadField.setAccessible(true);
            Object sCurrentActivityThread = sCurrentActivityThreadField.get(null);

            //获取mH属性
            Field mHField = activityThreadClazz.getDeclaredField("mH");
            mHField.setAccessible(true);
            //获取mH Handler对象
            Object mH = mHField.get(sCurrentActivityThread);

            //反射Handler
            Class<?> handlerClazz = Class.forName("android.os.Handler");
            Field mCallbackField = handlerClazz.getDeclaredField("mCallback");
            mCallbackField.setAccessible(true);

            //创建callback对象
            Handler.Callback callback = new Handler.Callback() {
                @Override
                public boolean handleMessage(@NonNull Message msg) {

                    //处理Intent替换的逻辑
                    doIntentReplace(msg);
                    return false;
                }
            };
            //给系统Handler赋值
            mCallbackField.set(mH, callback);

        } catch (Exception exception) {

        }

    }

    private static void doIntentReplace(Message msg) {
        switch (msg.what) {
            case 159:
                //获取
                Class<?> transactionClazz = msg.obj.getClass();
                try {
                    Field mActivityCallbacksField = transactionClazz.getDeclaredField("mActivityCallbacks");
                    mActivityCallbacksField.setAccessible(true);
                    List callbacks = (List) mActivityCallbacksField.get(msg.obj);

                    for (int i = 0; i < callbacks.size(); i++) {
                        Object transactionItem = callbacks.get(i);
                        //判断是不是LaunchActivityItem
                        if (transactionItem.getClass().getName().equals("android.app.servertransaction.LaunchActivityItem")) {
                            //获取mIntent属性
                            Field mIntentField = transactionItem.getClass().getDeclaredField("mIntent");
                            mIntentField.setAccessible(true);
                            Log.e("TAG", "intent " + mIntentField.get(transactionItem));
                            Intent mIntent = (Intent) mIntentField.get(transactionItem);
                            Intent oldIntent = mIntent.getParcelableExtra("old_intent");
                            mIntentField.set(transactionItem, oldIntent);
//                            mIntent.setComponent(oldIntent.getComponent()); 这种方式同样有效
                        }
                    }


                } catch (Exception e) {
                    e.printStackTrace();
                }

                break;
        }
    }
```

+ 加载插件资源

```java
public class PluginActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_plugin);
    }

    @Override
    public Resources getResources() {
        PluginManager.getInstance().loadPlugin(getApplicationContext(), "/data/data/com.gs.plugin/plugin.apk");
        return PluginManager.getInstance().getResources();
    }
}
```

#### 参考

+ [Android热修复及插件化原理](https://juejin.cn/post/7037041959352926221)
+ [Android进阶宝典 -- 插件化1（加载插件中类）](https://juejin.cn/post/7142475355293499422)
+ [Android进阶宝典 -- 插件化2（Hook启动插件中四大组件）](https://juejin.cn/post/7144243095989649415)
+ [Android进阶宝典 -- 插件化3（加载插件资源）](https://juejin.cn/post/7144627103122751518)
