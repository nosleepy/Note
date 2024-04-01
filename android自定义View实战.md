---
title: android自定义View实战
date: 2024-04-01 00:00:00
tags:
categories:
- 安卓
---

#### 弹出式圆环菜单

PopupViewGroup.kt

```kotlin
class PopupViewGroup @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): ViewGroup(context, attrs, defStyleAttr)  {
    private var isExpand = false
    private val mRoundColor = resources.getColor(R.color.color2)
    private val mCenterColor = resources.getColor(R.color.color6)
    private val mRoundPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply { color = mRoundColor }
    private val mCenterPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply { color = mRoundColor }
    private var curProgress = 0f
    private var addRotate = 0f
        set(value) {
            field = value
            invalidate()
        }
    private fun mRotateAnimator() =
        ObjectAnimator.ofFloat(
            this,
            "addRotate",
            if (isExpand) 45f else 0f,
            if (isExpand) 0f else 45f,
        ).apply {
            interpolator = OvershootInterpolator()
            duration = 300
        }
    private val mExpandCollapseAnimator by lazy {
        ValueAnimator.ofFloat(0f, 1f).apply {
            interpolator = OvershootInterpolator()
            duration = 400
            addUpdateListener {
                curProgress = it.animatedValue as Float
                if (!isExpand) {
                    curProgress = 1 - curProgress
                }
                invalidate()
            }
        }
    }
    private fun mAlphaAnimator() =
        ValueAnimator.ofFloat(
            if (isExpand) 1f else 0f,
            if (isExpand) 0f else 1f,
        ).apply {
            duration = 400
            addUpdateListener {
                mRoundPaint.alpha = ((it.animatedValue as Float) * 255).toInt()
                invalidate()
            }
        }
    private fun mColorAnimator() =
        ValueAnimator.ofObject(
            ArgbEvaluator(),
            if (isExpand) mCenterColor else mRoundColor,
            if (isExpand) mRoundColor else mCenterColor,
        ).apply {
            duration = 400
            addUpdateListener {
                mCenterPaint.color = it.animatedValue as Int
                invalidate()
            }
        }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        measureChildren(widthMeasureSpec, heightMeasureSpec)
        setMeasuredDimension(400, 400)
    }

    override fun dispatchDraw(canvas: Canvas) {
        canvas.drawColor(Color.WHITE)
        //绘制外圆
        if (curProgress > 0) {
            canvas.drawCircle(CENTER_X, CENTER_Y, SMALL_RADIUS + (BIG_RADIUS - SMALL_RADIUS) * curProgress, mRoundPaint)
        }
        //绘制辅助圆
//        canvas.drawPath(
//            Path().apply { addArc(HELP_RECT_F, 0f, 360f) },
//            Paint(Paint.ANTI_ALIAS_FLAG).apply {
//                color = Color.BLACK
//                style = Paint.Style.STROKE
//                val intervals = floatArrayOf(10f, 10f)
//                pathEffect = DashPathEffect(intervals, 0f)
//            }
//        )
        //绘制中心圆
        canvas.drawCircle(CENTER_X, CENTER_Y, SMALL_RADIUS, mCenterPaint)
        //绘制加号
        canvas.withSave {
            canvas.translate(CENTER_X, CENTER_Y)
            canvas.rotate(addRotate)
            val bitmap = BitmapFactory.decodeResource(resources, R.drawable.add, BitmapFactory.Options())
            canvas.drawBitmap(bitmap, -bitmap.width / 2f, -bitmap.height / 2f, mCenterPaint)
        }
        super.dispatchDraw(canvas)
    }

    private var hasDown = false

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                if (hasDown) {
                    return true
                } else {
                    hasDown = true
                    Thread {
                        Thread.sleep(500)
                        hasDown = false
                    }.start()
                }
                if (isExpand) {
                    if (PointF(event.x, event.y).containsCircle(PointF(CENTER_X, CENTER_Y), SMALL_RADIUS)) {
                        collapse()
                    } else {
                        return false
                    }
                } else {
                    if (PointF(event.x, event.y).containsCircle(PointF(CENTER_X, CENTER_Y), SMALL_RADIUS)) {
                        expand()
                    } else {
                        return false
                    }
                }
                mExpandCollapseAnimator.start()
                mColorAnimator().start()
                mRotateAnimator().start()
                mAlphaAnimator().start()
                isExpand = !isExpand
            }
        }
        invalidate()
        return true
    }

    override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
        when (ev?.action) {
            MotionEvent.ACTION_DOWN -> {
                if (!isExpand) {
                    return true
                }
            }
        }
        return super.onInterceptTouchEvent(ev)
    }

    private fun collapse() {
        var delay = 30L
        for (i in 0 until childCount) {
            getChildAt(i).animate().apply {
                startDelay = delay
                duration = 300
                alphaBy(1f)
                scaleXBy(1f)
                scaleYBy(1f)
                alpha(0f)
                scaleX(0f)
                scaleY(0f)
                start()
            }
            delay += 30L
        }
    }

    private fun expand() {
        var delay = 30L
        for (i in 0 until childCount) {
            getChildAt(i).animate().apply {
                startDelay = delay
                duration = 300
                alphaBy(0f)
                scaleXBy(0f)
                scaleYBy(0f)
                alpha(1f)
                scaleX(1f)
                scaleY(1f)
                start()
            }
            delay += 30L
        }
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        val path = Path().apply { addArc(HELP_RECT_F, 0f, 360f,) }
        val pathMeasure = PathMeasure(path, false)
        val length = pathMeasure.length
        val itemLength = length / childCount
        for (i in 0 until childCount) {
            val points = FloatArray(2)
            pathMeasure.getPosTan(i * itemLength, points, null)
            val x = points[0]
            val y = points[1]
            val child = getChildAt(i).apply {
                alpha = 0f
            }
            child.layout(
                (x - child.measuredWidth / 2).toInt(),
                (y - child.measuredHeight / 2).toInt(),
                (x + child.measuredWidth / 2).toInt(),
                (y + child.measuredHeight / 2).toInt(),
            )
        }
    }

    companion object {
        private const val CENTER_X = 200f
        private const val CENTER_Y = 200f
        private const val BIG_RADIUS = 150f
        private const val SMALL_RADIUS = 50f
        private const val HELP_RADIUS = (BIG_RADIUS - SMALL_RADIUS) / 2f + SMALL_RADIUS
        private val HELP_RECT_F = RectF(CENTER_X - HELP_RADIUS, CENTER_Y - HELP_RADIUS, CENTER_X + HELP_RADIUS, CENTER_Y + HELP_RADIUS)
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context=".MainActivity">

    <com.base.module.myapplication.widget.PopupViewGroup
        android:background="@color/color1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <ImageView
            android:src="@drawable/label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <ImageView
            android:src="@drawable/delete"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <ImageView
            android:src="@drawable/phone"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <ImageView
            android:src="@drawable/settings"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <ImageView
            android:src="@drawable/alarm"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <ImageView
            android:id="@+id/help"
            android:src="@drawable/help"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </com.base.module.myapplication.widget.PopupViewGroup>

</LinearLayout>
```

示例截图

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/popup_viewgroup_init.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/popup_viewgroup_show.png)

#### 参考

+ [『Android自定义View实战』实现一个小清新的弹出式圆环菜单](https://mp.weixin.qq.com/s/k6WF1SG_d9WQ4rrlM1bMxA)