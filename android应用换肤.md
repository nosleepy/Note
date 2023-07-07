---
title: android应用换肤
date: 2023-07-07 00:00:00
tags:
categories:
- 安卓
---

#### 制作皮肤包

1. 新建 Module 名为 skin_plugin, 选择 No Activity 类型

2. res 目录创建和应用 app 同名的资源

+ app 和 module 都在 res/drawable 目录放置 background.png 图片(白天主题和夜晚主题)
+ res/values/color.xml 创建 colorPrimary 颜色标签

```
app: <color name="colorPrimary">#4CAF50</color> // 绿色
module: <color name="colorPrimary">#FF5722</color> // 橙色
```

3 打包成 apk, 拷贝到 /storage/emulated/0/Download/ 目录

#### 加载插件资源

创建 SkinManager 加载插件 apk, 获取皮肤包中的资源

```java
public class SkinManager {
    private static SkinManager instance = null;
    private static Context context;
    private static Resources newRes;
    private static String skinPackageName;

    private SkinManager() {}

    public static void init(Context ctx) {
        context = ctx;
        instance = new SkinManager();
        // 皮肤包的路径
        String skinApkPath = "/storage/emulated/0/Download/skin.apk";
        // 获取皮肤包的 packageInfo
        PackageInfo pInfo = context.getPackageManager().getPackageArchiveInfo(skinApkPath, PackageManager.GET_ACTIVITIES);
        skinPackageName = pInfo.packageName;
        pInfo.applicationInfo.sourceDir = skinApkPath;
        pInfo.applicationInfo.publicSourceDir = skinApkPath;
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
        newRes = new Resources(skinRes.getAssets(), res.getDisplayMetrics(), res.getConfiguration());
    }

    public static SkinManager getInstance() {
        return instance;
    }

    public int getSkinColorPrimary() {
        int skinResId = getSkinResId(R.color.colorPrimary);
        return newRes.getColor(skinResId); // 通过资源 id 在皮肤包的 Resources 中寻找 Color
    }

    public Drawable getSkinBackground() {
        int skinResId = getSkinResId(R.drawable.background);
        return newRes.getDrawable(skinResId); // 通过资源 id 在皮肤包的 Resources 中寻找 Drawable
    }

    private int getSkinResId(int resId) {
        Resources res = context.getResources();
        String resName = res.getResourceEntryName(resId); // colorPrimary
        String resType = res.getResourceTypeName(resId); // color
        return newRes.getIdentifier(resName, resType, skinPackageName); // 从皮肤包中寻找同名资源的 id
    }
}
```

#### 自定义属性

res/values 目录新建 attrs.xml 文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="Skinable">
        <attr name="isSupport" format="boolean"/>
    </declare-styleable>
</resources>
```

自定义 SkinFactory 实现 LayoutInflater.Factory2 接口

```java
class SkinView {
    View view;
    Map<String, String> attrsMap;

    public SkinView(View view, Map<String, String> attrsMap) {
        this.view = view;
        this.attrsMap = attrsMap;
    }
}

public class SkinFactory implements LayoutInflater.Factory2 {
    private AppCompatDelegate delegate;
    private List<SkinView> skinList = new ArrayList<>();
    private static final String[] prefixList = {"android.widget", "android.view", "android.webkit"};

    public SkinFactory(AppCompatDelegate delegate) {
        this.delegate = delegate;
    }

    @Nullable
    @Override
    public View onCreateView(@Nullable View parent, @NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        // 创建View
        View view = delegate.createView(parent, name, context, attrs);
        if (view == null) {
            // 没有创建成功，需要通过反射来创建
            for (String s : prefixList) {
                String newName = s + "." + name;
                view = onCreateView(newName, context, attrs);
                if (view != null) {
                    break;
                }
            }
        }
        // 收集可以换肤的组件
        if (view != null) {
            collectSkinComponent(attrs, context, view);
        }
        return view;
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        View view = null;
        try {
            // 获取到控件的class对象
            Class<?> clazz = context.getClassLoader().loadClass(name);
            // 获取到构造方法
            Constructor<?> constructor = clazz.getConstructor(Context.class, AttributeSet.class);
            view = (View) constructor.newInstance(context, attrs);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return view;
    }

    // 收集能够进行换肤的控件
    public void collectSkinComponent(AttributeSet attrs, Context context, View view) {
        //获取属性
        TypedArray skinAbleAttr = context.obtainStyledAttributes(attrs, R.styleable.Skinable, 0, 0);
        boolean isSupportSkin = skinAbleAttr.getBoolean(R.styleable.Skinable_isSupport, false);
        if (isSupportSkin) {
            Map<String, String> attrsMap = new HashMap<>();
            //收集起来
            for (int i = 0; i < attrs.getAttributeCount(); i++) {
                String name = attrs.getAttributeName(i);
                String value = attrs.getAttributeValue(i);
                if (name.equals("background") || name.equals("textColor")) {
                    attrsMap.put(name, value);
                }
            }
            SkinView skinView = new SkinView(view, attrsMap);
            skinList.add(skinView);
        }
        skinAbleAttr.recycle();
    }

    // 一键换肤
    public void changedSkin() {
        for (SkinView skinView : skinList) {
            for (Map.Entry<String, String> entry : skinView.attrsMap.entrySet()) {
                String name = entry.getKey();
                if (name.equals("background")) {
                    skinView.view.setBackgroundDrawable(SkinManager.getInstance().getSkinBackground());
                } else if (name.equals("textColor")) {
                    TextView textView = (TextView) skinView.view;
                    textView.setTextColor(SkinManager.getInstance().getSkinColorPrimary());
                }
            }
        }
    }
}
```

Activity 中替换掉 LayoutInflater 类的 Factory2 属性

```java
public class MainActivity extends AppCompatActivity {
    private SkinFactory skinFactory = new SkinFactory(getDelegate());

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        SkinManager.init(getApplicationContext());
        LayoutInflater.from(this).setFactory2(skinFactory);
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void changeSkin(View view) {
        skinFactory.changedSkin();
    }
}
```

添加权限

```xml
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>
```

申请权限

```java
boolean isHasStoragePermission = Environment.isExternalStorageManager();
if (!isHasStoragePermission) {
    Intent intent = new Intent(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
    startActivity(intent);
}
```

布局文件中使用自定义属性标注需要换肤的控件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/background"
    android:orientation="vertical"
    android:id="@+id/cs_root"
    skin:isSupport="true"
    xmlns:skin="http://schemas.android.com/apk/res-auto">

    <TextView
        android:text="hello world!"
        skin:isSupport="true"
        android:textSize="20sp"
        android:textColor="@color/colorPrimary"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <Button
        android:onClick="changeSkin"
        android:text="换肤"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

#### 创建接口

创建 SkinSupportable 接口, 自定义需要换肤的 View 实现该接口

```java
public interface SkinSupportable {
    void applySkin(); // 换肤方法
}

public class SkinTextView extends TextView implements SkinSupportable {
    public SkinTextView(Context context) {
        super(context);
    }

    public SkinTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void applySkin() {
        setTextColor(SkinManager.getInstance().getSkinColorPrimary());
    }
}

public class SkinLinearLayout extends LinearLayout implements SkinSupportable {
    public SkinLinearLayout(Context context) {
        super(context);
    }

    public SkinLinearLayout(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void applySkin() {
        setBackgroundDrawable(SkinManager.getInstance().getSkinBackground());
    }
}
```

自定义 SkinFactory 收集需要换肤的控件

```java
public class SkinFactory implements LayoutInflater.Factory2 {
    private List<SkinSupportable> skinViews = new ArrayList<>();
    private AppCompatDelegate delegate;
    private static final String[] prefixList = {"android.widget", "android.view", "android.webkit"};

    public SkinFactory(AppCompatDelegate delegate) {
        this.delegate = delegate;
    }

    @Nullable
    @Override
    public View onCreateView(@Nullable View parent, @NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        if (name.equals("TextView")) { // 如果是 Button 就替换为我们自己的 SkinButton
            SkinTextView view = new SkinTextView(context, attrs);
            skinViews.add(view); // 记录
            return view;
        } else if (name.equals("LinearLayout")) {
            SkinLinearLayout view = new SkinLinearLayout(context, attrs);
            skinViews.add(view);
            return view;
        }
        View view = delegate.createView(parent, name, context, attrs);
        if (view == null) {
            // 没有创建成功，需要通过反射来创建
            for (String s : prefixList) {
                String newName = s + "." + name;
                view = onCreateView(newName, context, attrs);
                if (view != null) {
                    break;
                }
            }
        }
        return view;
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        View view = null;
        try {
            // 获取到控件的class对象
            Class<?> clazz = context.getClassLoader().loadClass(name);
            // 获取到构造方法
            Constructor<?> constructor = clazz.getConstructor(Context.class, AttributeSet.class);
            view = (View) constructor.newInstance(context, attrs);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return view;
    }

    public void changedSkin() {
        for (SkinSupportable skinView : skinViews) {
            skinView.applySkin();
        }
    }
}
```

Activity 中设置 LayoutInflater 的 factory2 属性

```java
public class MainActivity extends AppCompatActivity {
    private SkinFactory skinFactory = new SkinFactory(getDelegate());

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        boolean isHasStoragePermission = Environment.isExternalStorageManager();
        if (!isHasStoragePermission) {
            Intent intent = new Intent(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
            startActivity(intent);
        }
        SkinManager.init(getApplicationContext());
        LayoutInflater.from(this).setFactory2(skinFactory);
        // 要在调用父类 onCreate 方法前对 Factory2 进行替换
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void changeSkin(View view) {
        skinFactory.changedSkin();
    }
}
```

布局文件配置

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@drawable/background"
    tools:context=".MainActivity">

    <TextView
        android:text="hello world!"
        android:textSize="20sp"
        android:textColor="@color/colorPrimary"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <Button
        android:onClick="changeSkin"
        android:text="换肤"
        android:background="@color/colorPrimary"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

#### 实现效果

+ 换肤前：白天主题绿色字体

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/app_light.png)

+ 换肤后：夜间主题橙色字体

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/app_night.png)

#### 参考

+ [Android进阶宝典 -- 使用Hook技术拦截系统实例化View过程实现App换肤功能](https://juejin.cn/post/7193260391256817724)
+ [Android 干货分享：插件化换肤原理（1）—— 布局加载过程、View创建流程、Resources 浅析](https://juejin.cn/post/7153807668988084237)
+ [Android 干货分享：插件化换肤原理（2）—— 实现思路、主流框架分析](https://juejin.cn/post/7154205345462616077)