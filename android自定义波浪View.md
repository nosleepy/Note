---
title: android自定义波浪View
date: 2024-03-30 00:00:00
tags:
categories:
- 安卓
---

#### 实现

WaveView.kt

```kotlin
class WaveView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): View(context, attrs, defStyleAttr)  {
    private val circlePath = Path()
    private val wavePath = Path()
    private val circlePaint by lazy {
        Paint().apply {
            isAntiAlias = true
            color = Color.GRAY
            style = Paint.Style.FILL
        }
    }
    private val wavePaint by lazy {
        Paint().apply {
            isAntiAlias = true
            color = Color.parseColor("#449FFF")
            style = Paint.Style.FILL_AND_STROKE
        }
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        setMeasuredDimension(WIDTH, HEIGHT)
    }

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        circlePath.reset()
        circlePath.addCircle(SPACE * 2, SPACE * 2, RADIUS, Path.Direction.CCW)
        wavePath.reset()
        wavePath.apply {
            moveTo(0f, SPACE * 2)
            quadTo(SPACE, SPACE, SPACE * 2, SPACE * 2)
            quadTo(SPACE * 3, SPACE * 3, SPACE * 4, SPACE * 2)
            quadTo(SPACE * 5, SPACE, SPACE * 6, SPACE * 2)
            quadTo(SPACE * 7, SPACE * 3, SPACE * 8, SPACE * 2)
            lineTo(WIDTH * 1.0f, HEIGHT * 1.0f)
            lineTo(0f, HEIGHT * 1.0f)
            close()
        }
    }

    private var offsetX = 0f
        set(value) {
            field = value
            invalidate()
        }

    private val valueAnimator by lazy {
        ValueAnimator.ofFloat(0f, -WIDTH / 2f).apply {
            duration = 1500
            interpolator = LinearInterpolator()
            repeatMode = ValueAnimator.RESTART
            repeatCount = ValueAnimator.INFINITE
        }
    }

    private val objectAnimator by lazy {
        ObjectAnimator.ofFloat(this, "offsetX", 0f, -WIDTH / 2f).apply {
            duration = 1500
            interpolator = LinearInterpolator()
            repeatMode = ObjectAnimator.RESTART
            repeatCount = ObjectAnimator.INFINITE
        }
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.drawPath(circlePath, circlePaint)
//        圆形区域裁剪
        canvas.clipPath(circlePath)
        canvas.withSave {
            canvas.translate(offsetX, 0f)
            canvas.drawPath(wavePath, wavePaint)
        }
    }

    override fun onAttachedToWindow() {
        super.onAttachedToWindow()
//        1.使用值动画
//        valueAnimator.addUpdateListener {
//            offsetX = it.animatedValue as Float
//            invalidate()
//        }
//        valueAnimator.start()
//        2.使用对象动画
        objectAnimator.start()
    }

    override fun onDetachedFromWindow() {
        super.onDetachedFromWindow()
//        1.值动画取消
//        valueAnimator.cancel()
//        valueAnimator.removeAllUpdateListeners()
//        2.对象动画取消
        objectAnimator.cancel()
    }

    companion object {
        private const val SPACE = 75f
        private const val WIDTH = 600
        private const val HEIGHT = 300
        private const val RADIUS = 150f
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
    tools:context=".MainActivity">

    <com.base.module.myapplication.widget.WaveView
        android:background="@color/bg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

#### 示例截图

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/wave_view_test.png)