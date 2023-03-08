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

#### 参考

+ [Android Window 机制探索](https://juejin.cn/post/6844903512447385614#heading-5)
+ [为什么 Dialog 不能用 Application 的 Context](https://juejin.cn/post/6844904115105955853)