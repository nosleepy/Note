---
title: android实现菱形按钮布局
date: 2023-10-13 00:00:00
tags:
- 自定义view
categories:
- 安卓
---

使用 RelativeLayout 相对布局 + 自定义 View 实现

菱形按钮布局效果截图

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/diamond_layout.png)

#### 自定义 DiamondView

**自定义属性**

res/values 目录下新建 attrs.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="DiamondView">
        <attr name="diamondColor" format="color"/>
        <attr name="diamondPressColor" format="color"/>
        <attr name="diamondTitle" format="string"/>
        <attr name="diamondIcon" format="reference"/>
    </declare-styleable>

</resources>
```

**自定义 View**

```kotlin
class DiamondView @JvmOverloads constructor(context: Context?, attrs: AttributeSet? = null) :
    View(context, attrs, 0) {
    private val diamondPaint: Paint
    private val whitePaint: Paint
    private val diamondPath: Path
    private val diamondColor: Int
    private val diamondPressColor: Int
    private val diamondTitle: String?
    private val diamondIcon: Int
    private val iconBitmap: Bitmap

    init {
        val typedArray = getContext().obtainStyledAttributes(attrs, R.styleable.DiamondView)
        diamondColor = typedArray.getColor(R.styleable.DiamondView_diamondColor, 0)
        diamondPressColor = typedArray.getColor(R.styleable.DiamondView_diamondPressColor, 0)
        diamondTitle = typedArray.getString(R.styleable.DiamondView_diamondTitle)
        diamondIcon = typedArray.getResourceId(R.styleable.DiamondView_diamondIcon, 0)
        typedArray.recycle()
        iconBitmap = BitmapFactory.decodeResource(resources, diamondIcon)
        diamondPaint = Paint()
        diamondPaint.isAntiAlias = true
        diamondPaint.color = diamondColor
        diamondPaint.style = Paint.Style.FILL
        diamondPath = Path()
        whitePaint = Paint()
        whitePaint.isAntiAlias = true
        whitePaint.color = Color.WHITE
        whitePaint.textSize = 30f
        whitePaint.textAlign = Paint.Align.CENTER
    }

    @Override
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val size = width
        val offset = 6
        diamondPath.reset()
        diamondPath.moveTo((size / 2 - offset).toFloat(), offset.toFloat())
        diamondPath.cubicTo(
            (size / 2 - offset).toFloat(),
            offset.toFloat(),
            (size / 2).toFloat(),
            0f,
            (size / 2 + offset).toFloat(),
            offset.toFloat()
        )
        diamondPath.lineTo((size - offset).toFloat(), (size / 2 - offset).toFloat())
        diamondPath.cubicTo(
            (size - offset).toFloat(),
            (size / 2 - offset).toFloat(),
            size.toFloat(),
            (size / 2).toFloat(),
            (size - offset).toFloat(),
            (size / 2 + offset).toFloat()
        )
        diamondPath.lineTo((size / 2 + offset).toFloat(), (size - offset).toFloat())
        diamondPath.cubicTo(
            (size / 2 + offset).toFloat(),
            (size - offset).toFloat(),
            (size / 2).toFloat(),
            size.toFloat(),
            (size / 2 - offset).toFloat(),
            (size - offset).toFloat()
        )
        diamondPath.lineTo(offset.toFloat(), (size / 2 + offset).toFloat())
        diamondPath.cubicTo(
            offset.toFloat(),
            (size / 2 + offset).toFloat(),
            0f,
            (size / 2).toFloat(),
            offset.toFloat(),
            (size / 2 - offset).toFloat()
        )
        diamondPath.close()
        canvas.drawPath(diamondPath, diamondPaint)
        canvas.drawBitmap(iconBitmap, ((size - 90) / 2).toFloat(), 73f, whitePaint)
        canvas.drawText(diamondTitle!!, (size / 2).toFloat(), 200f, whitePaint)
    }

    @Override
    override fun onTouchEvent(event: MotionEvent): Boolean {
        val x = event.x.toInt()
        val y = event.y.toInt()
        val size = width
        val cx = x - size / 2
        val cy = size - y - size / 2
        return if (abs(cx * size / 2) + abs(cy * size / 2) <= size * size / 4) {
            when (event.action) {
                MotionEvent.ACTION_DOWN -> diamondPaint.color = diamondPressColor
                MotionEvent.ACTION_UP -> diamondPaint.color = diamondColor
            }
            invalidate()
            super.onTouchEvent(event)
        } else {
            diamondPaint.color = diamondColor
            invalidate()
            false
        }
    }
}
```

#### 使用 RelativeLayout 相对布局

```xml
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:layout_width="582dp"
        android:layout_height="582dp"
        android:layout_centerInParent="true">

        <com.gs.myapplication.DiamondView
            android:id="@+id/left"
            android:layout_width="276dp"
            android:layout_height="276dp"
            android:layout_alignParentLeft="true"
            android:layout_centerVertical="true"
            app:diamondTitle="拨号"
            app:diamondIcon="@drawable/home_icon_dial"
            app:diamondColor="#8dc13f"
            app:diamondPressColor="#998dc13f"/>

        <com.gs.myapplication.DiamondView
            android:id="@+id/top"
            android:layout_width="276dp"
            android:layout_height="276dp"
            android:layout_alignParentTop="true"
            android:layout_centerHorizontal="true"
            app:diamondTitle="视频会议"
            app:diamondIcon="@drawable/home_icon_video_conf"
            app:diamondColor="#32b2ff"
            app:diamondPressColor="#9932b2ff"/>

        <com.gs.myapplication.DiamondView
            android:id="@+id/right"
            android:layout_width="276dp"
            android:layout_height="276dp"
            android:layout_alignParentRight="true"
            android:layout_centerVertical="true"
            app:diamondTitle="通讯录"
            app:diamondIcon="@drawable/home_icon_enterprise_contacts"
            app:diamondColor="#fc993b"
            app:diamondPressColor="#99fc993b"/>

        <com.gs.myapplication.DiamondView
            android:id="@+id/bottom"
            android:layout_width="276dp"
            android:layout_height="276dp"
            android:layout_alignParentBottom="true"
            android:layout_centerHorizontal="true"
            app:diamondTitle="语音会议"
            app:diamondIcon="@drawable/home_icon_audio_conf"
            app:diamondColor="#c95cf7"
            app:diamondPressColor="#99c95cf7"/>

    </RelativeLayout>

</RelativeLayout>
```

DiamondView 中的 onDraw 方法使用 Path 连接正方形四个边的中点,实现了菱形效果,利用 RelativeLayout 相对布局很方便完成菱形布局。
