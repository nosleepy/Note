---
title: 自定义ScrollLinearLayout
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

安卓中的 LinearLayout 默认只能显示屏幕高度的内容，不支持滑动，通过自定义 View 来实现，需要继承 LinearLayout。

```java
public class ScrollLinearLayout extends LinearLayout {
    public ScrollLinearLayout(Context context) {
        this(context, null);
    }

    public ScrollLinearLayout(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    private int mLastY;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getRawY();
        int screenHeight = getScreenHeight(getContext());
        int curHeight = getScrollY() + screenHeight;
        int allHeight = getAllHeight();
        switch (event.getAction()) {
            case MotionEvent.ACTION_MOVE:
                int dy = mLastY - y;
                if (dy >= 0 && allHeight >= screenHeight) { //上滑
                    if (curHeight <= allHeight) {
                        int offsetY = allHeight - curHeight;
                        if (offsetY < 100) {
                            scrollBy(0, offsetY); //保持底部对齐
                        } else {
                            scrollBy(0, dy); //继续滑动
                        }
                    }
                } else { //下滑
                    if (getScrollY() >= 0) { //不能向上滑了
                        if (getScrollY() <= 100) {
                            scrollTo(0, 0); //保持对齐
                        } else {
                            scrollBy(0, dy); //继续滑动
                        }
                    }
                }
                break;
        }
        mLastY = y;
        return true;
    }

    public int getAllHeight() {
        int sum = 0;
        for (int i = 0; i < getChildCount(); i++) {
            sum += getChildAt(i).getMeasuredHeight();
        }
        return sum;
    }

    public int getScreenHeight(Context context) {
        WindowManager wm = (WindowManager) getContext().getSystemService(Context.WINDOW_SERVICE);
        return wm.getDefaultDisplay().getHeight();
    }
}
```

重写 onTouchEvent 方法实现滚动效果，增加逻辑判断后不能滚出子 View 总高度。

布局文件，ScrollLinearLayout 内容超过屏幕高度。

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.example.myapplication.ScrollLinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="500dp"
        android:background="#FFE7E7" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="500dp"
        android:background="#EDFFD8" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="500dp"
        android:background="#D9DDF6" />

</com.example.myapplication.ScrollLinearLayout>
```
