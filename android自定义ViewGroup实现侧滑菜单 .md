---
title: android自定义ViewGroup实现侧滑菜单 
date: 2024-02-23 00:00:00
tags:
categories:
- 安卓
---

**实现截图**

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/slide_viewgroup.png)

**实现内容**

完成左侧菜单滑出, 滑动距离小于菜单宽度一半自动关闭, 大于则自动打开

**布局文件**

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.gs.myapplication.SlideViewGroup
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:id="@+id/menu"
        android:layout_width="200dp"
        android:layout_height="match_parent"
        android:background="#F4DCDC">

        <TextView
            android:text="left menu"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

    <LinearLayout
        android:id="@+id/main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FBCABB">

        <TextView
            android:text="world"
            android:background="#9FD1E8"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</com.gs.myapplication.SlideViewGroup>
```

**自定义VIewGroup内容**

```kotlin
class SlideViewGroup : ViewGroup {
    constructor(context: Context) : super(context)
    constructor(context: Context, attributeSet: AttributeSet) : super(context, attributeSet)

    private var scroller: Scroller = Scroller(context)
    private lateinit var menuView: LinearLayout
    private lateinit var mainView: LinearLayout

    override fun onFinishInflate() {
        super.onFinishInflate()
        menuView = findViewById(R.id.menu)
        mainView = findViewById(R.id.main)
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        measureChildren(widthMeasureSpec, heightMeasureSpec)
        val widthSize = MeasureSpec.getSize(widthMeasureSpec)
        val heightSize = MeasureSpec.getSize(heightMeasureSpec)
        setMeasuredDimension(widthSize, heightSize)
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        menuView.layout(-menuView.measuredWidth, 0, 0, menuView.measuredHeight)
        mainView.layout(0, 0, mainView.measuredWidth, mainView.measuredHeight)
    }

    private var lastX: Int = 0

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                lastX = event.x.toInt()
            }
            MotionEvent.ACTION_MOVE -> {
                val curX = event.x.toInt()
                var moveX = curX - lastX
                if (moveX >= 0) { // 右滑
                    if (moveX - scrollX > menuView.measuredWidth) {
                        moveX = menuView.measuredWidth + scrollX
                    }
                } else { // 左滑
                    if (scrollX - moveX > 0) {
                        moveX = scrollX
                    }
                }
                scrollBy(-moveX, 0)
                lastX = curX
            }
            MotionEvent.ACTION_UP -> {
                if (scrollX > -menuView.measuredWidth / 2) { // closeMenu
                    scroller.startScroll(scrollX, 0, -scrollX, 0)
                    invalidate()
                } else { // openMenu
                    scroller.startScroll(scrollX, 0, -menuView.measuredWidth - scrollX, 0)
                    invalidate()
                }
            }
        }
        return true
    }

    override fun computeScroll() {
        super.computeScroll()
        if (scroller.computeScrollOffset()) { // 返回true,表示动画没结束
            scrollTo(scroller.currX, 0)
            invalidate()
        }
    }
}
```

**待完善功能**

+ 支持右侧菜单滑动
+ 支持层级菜单打开和关闭

**参考**

+ [android自定义ViewGroup(侧滑菜单)](https://www.cnblogs.com/jasongaoh/p/7834229.html)
+ [Android自定义组件系列【3】——自定义ViewGroup实现侧滑](https://www.cnblogs.com/lanzhi/p/6469039.html)