---
title: 安卓添加Window窗口
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### 添加应用窗口

不需要权限

```kotlin
class MainActivity : Activity() {
    private lateinit var mWindowManager: WindowManager
    private lateinit var layoutParams: WindowManager.LayoutParams
    private lateinit var btn: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mWindowManager = getSystemService(WINDOW_SERVICE) as WindowManager // 使用activity的context
        layoutParams = WindowManager.LayoutParams().apply {
            width = 200
            height = 120
            gravity = Gravity.CENTER
            flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL // 传递单击事件给下层
        }
        btn = Button(this).apply {
            text = "Btn"
            setOnClickListener { mWindowManager.removeView(btn) } // 移除窗口
        }
        mWindowManager.addView(btn, layoutParams) // 添加窗口
    }
}
```

WindowManager 的 LayoutParams 的默认类型是 TYPE_APPLICATION。

#### 添加子窗口

子窗口依赖于父窗口的token, 所以需要等父窗口添加完成才能添加

```java
getWindow().getDecorView().post(new Runnable() {
    @Override
    public void run() {
        IBinder token = getWindow().getDecorView().getWindowToken();
        WindowManager windowManager = (WindowManager) getSystemService(Context.WINDOW_SERVICE);
        WindowManager.LayoutParams params = new WindowManager.LayoutParams();
        params.height = 100;
        params.width = 1024;
        params.format = PixelFormat.TRANSPARENT;
        params.gravity = Gravity.TOP;
        params.flags = WindowManager.LayoutParams.FLAG_FULLSCREEN | WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL ;
        params.type = WindowManager.LayoutParams.TYPE_APPLICATION_PANEL;
        params.token = token;
        TextView view = new TextView(MainActivity.this);
        view.setText("HelloWorld");
        view.setTextColor(Color.RED);
        view.setBackgroundColor(Color.GREEN);
        windowManager.addView(view, params);
    }
});
```

#### 添加系统窗口

添加权限，显示在其他应用上层

```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
```

简单测试代码

```kotlin
class MainActivity : Activity() {
    private lateinit var mWindowManager: WindowManager
    private lateinit var layoutParams: WindowManager.LayoutParams
    private lateinit var btn: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mWindowManager = applicationContext.getSystemService(WINDOW_SERVICE) as WindowManager // 使用application的context
        layoutParams = WindowManager.LayoutParams().apply {
            width = 200
            height = 120
            gravity = Gravity.CENTER
            flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL // 传递单击事件给下层
            type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY // 标识为系统窗口
        }
        btn = Button(this).apply {
            text = "Btn"
            setOnClickListener { windowManager.removeView(btn) } // 移除窗口
        }
    }

    override fun onResume() {
        super.onResume()
        if (!checkFloatPermission(this)) {
            val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION) // 跳转授权界面
            startActivityForResult(intent, 1)
        } else {
            windowManager!!.addView(btn, layoutParams) // 添加窗口
        }
    }

    private fun checkFloatPermission(context: Context): Boolean {
        val appOpsMgr = context.getSystemService(APP_OPS_SERVICE) as AppOpsManager
        val mode = appOpsMgr.checkOpNoThrow("android:system_alert_window", Process.myUid(), context.packageName)
        return mode == AppOpsManager.MODE_ALLOWED || mode == AppOpsManager.MODE_IGNORED
    }
}
```

#### 注意

Activity 和 Dialog 属于应用窗口，Toast 和输入法属于系统窗口，PopupWindow 属于子窗口。

#### 常见问题

+ 想让一个Window屏蔽掉系统手势和左右返回怎么做?

```java
WindowManager.LayoutParams params = getWindow().getAttributes();
params.flags = WindowManager.LayoutParams.FLAG_FULLSCREEN; // 设置全屏flag
params.systemUiVisibility = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
    | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
    | View.SYSTEM_UI_FLAG_FULLSCREEN;
getWindow().setAttributes(params);
setContentView(R.layout.activity_main);
```

+ 如何不显示StatusBar区域的内容?

申请悬浮窗权限或者拥有系统权限,添加系统类型窗口,覆盖在StatusBar上面

#### 参考

+ [Android Window 机制探索](https://juejin.cn/post/6844903512447385614#heading-5)
+ [为什么 Dialog 不能用 Application 的 Context](https://juejin.cn/post/6844904115105955853)
