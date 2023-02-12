---
title: 自定义View学习
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### 为什么要自定义View

+ Android系统内置的View无法实现我们的需求，我们需要针对我们的业务需求定制我们想要的View。
+ 一般需要重写两个方法 onMeasure() 和 onDraw()，onMeasure负责测量，onDraw负责绘制。至少写2个构造函数：

```java
public class MyView extends View {
    public MyView(Context context) { // 用于new对象
        this(context, null);
    }

    public MyView(Context context, AttributeSet attrs) { // 用于布局文件
        super(context, attrs);
    }
}
```

#### onMeasure方法

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
```
widthMeasureSpec 和 heightMeasureSpec 是测量规格，包含测量模式和测量大小，通过安卓内置类 MeasureSpec 可以提取出来。

```java
int widthMode = MeasureSpec.getMode(widthMeasureSpec);
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
```

测量规格意义

|测量模式|表示意思|测试大小|
|:--:|:--:|----|
|UNSPECIFIED|父容器没有对当前View有任何限制，当前View可以任意取尺寸|无限大|
|EXACTLY|当前的尺寸就是当前View应该取的尺寸|有具体数值|
|AT_MOST|当前尺寸是当前View能取的最大尺寸|默认最大宽/高|

#### 自定义View

通过自定义 MyView 继承 View，实现圆的绘制。

```java
public class MyView extends View {
    private final Paint paint = new Paint();

    public MyView(Context context) {
        this(context, null);
    }

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
        paint.setColor(Color.GREEN);
    }

    private int getMySize(int defaultSize, int measureSpec) {
        int mySize = defaultSize;
        int mode = MeasureSpec.getMode(measureSpec);
        int size = MeasureSpec.getSize(measureSpec);
        switch (mode) {
            case MeasureSpec.UNSPECIFIED:
                mySize = defaultSize;
                break;
            case MeasureSpec.AT_MOST: // wrap_content
                mySize = size;
                break;
            case MeasureSpec.EXACTLY: // match_parent 或 100dp
                mySize = size;
                break;
        }
        return mySize;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = getMySize(100, widthMeasureSpec);
        int height = getMySize(100, heightMeasureSpec);
        // 使View的宽高相等
        if (width < height) {
            height = width;
        } else {
            width = height;
        }
        setMeasuredDimension(width, height);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //绘制圆圈
        int r = getMeasuredWidth() / 2;
        int centerX = getLeft() + r;
        int centerY = getTop() + r;
        canvas.drawCircle(centerX, centerY, r, paint);
    }
}
```

布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.grandstream.myapplication.MyView
        android:background="#FACECE"
        android:layout_width="100dp"
        android:layout_height="150dp"/>

</LinearLayout>
```

#### 自定义ViewGroup

需要重写 onMeasure() 和 onLayout() 两个方法。onMeasure()：实现测量子 View 大小以及设定 ViewGroup 的大小；onLayout()：确定子 View 的摆放位置。

```java
public class MyViewGroup extends ViewGroup {
    public MyViewGroup(Context context) {
        this(context, null);
    }

    public MyViewGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    private int getMaxChildWidth() {
        int childCount = getChildCount();
        int maxWidth = 0;
        for (int i = 0; i < childCount; i++) {
            View childView = getChildAt(i);
            if (childView.getMeasuredWidth() > maxWidth) {
                maxWidth = childView.getMeasuredWidth();
            }
        }
        return maxWidth;
    }

    private int getTotalHeight() {
        int childCount = getChildCount();
        int height = 0;
        for (int i = 0; i < childCount; i++) {
            View childView = getChildAt(i);
            height += childView.getMeasuredHeight();
        }
        return height;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //将所有的子View进行测量，这会触发每个子View的onMeasure函数
        //注意要与measureChild区分，measureChild是对单个view进行测量
        measureChildren(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        int childCount = getChildCount();
        if (childCount == 0) {//如果没有子View,当前ViewGroup没有存在的意义，不用占用空间
            setMeasuredDimension(0, 0);
        } else {
            //如果宽高都是包裹内容
            if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
                //我们将高度设置为所有子View的高度相加，宽度设为子View中最大的宽度
                int height = getTotalHeight();
                int width = getMaxChildWidth();
                setMeasuredDimension(width, height);
            } else if (heightMode == MeasureSpec.AT_MOST) {//如果只有高度是包裹内容
                //宽度设置为ViewGroup自己的测量宽度，高度设置为所有子View的高度总和
                setMeasuredDimension(widthSize, getTotalHeight());
            } else if (widthMode == MeasureSpec.AT_MOST) {//如果只有宽度是包裹内容
                //宽度设置为子View中宽度最大的值，高度设置为ViewGroup自己的测量值
                setMeasuredDimension(getMaxChildWidth(), heightSize);
            }
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int count = getChildCount();
        //记录当前的高度位置
        int curHeight = t;
        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            int height = child.getMeasuredHeight();
            int width = child.getMeasuredWidth();
            child.layout(l, curHeight, l + width, curHeight + height);
            curHeight += height;
        }
    }
}
```

布局测试文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.grandstream.myapplication.MyViewGroup xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFBFD7"
    android:id="@+id/my"
    tools:context=".MainActivity">

    <TextView
        android:text="111"
        android:background="#FD6E6E"
        android:layout_width="match_parent"
        android:layout_height="200dp"/>

    <TextView
        android:text="222"
        android:background="#A0E1A3"
        android:layout_width="match_parent"
        android:layout_height="300dp"/>

    <TextView
        android:text="333"
        android:background="#FD6E6E"
        android:layout_width="match_parent"
        android:layout_height="200dp"/>

    <TextView
        android:text="444"
        android:background="#C2E0F8"
        android:layout_width="match_parent"
        android:layout_height="400dp"/>

</com.grandstream.myapplication.MyViewGroup>
```

实现了类似 LinearLayout 竖直排列的效果。
