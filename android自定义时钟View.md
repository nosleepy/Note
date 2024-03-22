---
title: android自定义时钟View
date: 2024-03-22 00:00:00
tags:
categories:
- 安卓
---

#### 测试

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/clock_test.png)

#### 实现

+ ClockView.kt

```kotlin
class ClockView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): View(context, attrs, defStyleAttr) {
    private val mTextPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.FILL
        textSize = 20f
    }
    private val mPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.STROKE
    }
    private val mRadius = 280f
    private val centerX = 300f
    private val centerY = 300f
    private val mHandler = MyHandler(this)

    class MyHandler(clockView: ClockView): Handler() {
        private val reference = WeakReference(clockView)
        override fun handleMessage(msg: Message) {
            super.handleMessage(msg)
            when (msg.what) {
                REFRESH -> {
                    reference.get()?.invalidate()
                    sendEmptyMessageDelayed(REFRESH, 1000)
                }
            }
        }
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        setMeasuredDimension(600, 600)
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        drawCircle(canvas)
        drawScale(canvas)
        drawPointer(canvas)
        mHandler.sendEmptyMessage(1)
    }

    private fun drawCircle(canvas: Canvas) {
        mPaint.strokeWidth = 3f
        mPaint.style = Paint.Style.STROKE
        mPaint.color = Color.BLACK
        canvas.drawCircle(centerX, centerY, mRadius, mPaint)
    }

    private fun drawScale(canvas: Canvas) {
        for (index in 0..59) {
            if (index == 0 || index == 15 || index == 30 || index == 45) {
                mPaint.strokeWidth = 6f
                mPaint.color = Color.MAGENTA
                canvas.drawLine(centerX, centerY - mRadius + 8, centerX, centerY - mRadius + 30f, mPaint)
                val scaleTv = if (index == 0) 12 else index / 5
                canvas.drawText("$scaleTv", centerX - mTextPaint.measureText("$scaleTv") / 2, centerY - mRadius + 54f, mTextPaint)
            } else if (index == 5 || index == 10 || index == 20 || index == 25 || index == 35 || index == 40 || index == 50 || index == 55) {
                mPaint.strokeWidth = 4f
                mPaint.color = Color.BLACK
                canvas.drawLine(centerX, centerY - mRadius + 8, centerX, centerY - mRadius + 26f, mPaint)
                val scaleTv = index / 5
                canvas.drawText("$scaleTv", centerX - mTextPaint.measureText("$scaleTv") / 2, centerY - mRadius + 54f, mTextPaint)
            } else {
                mPaint.strokeWidth = 2f
                mPaint.color = Color.BLACK
                canvas.drawLine(centerX, centerY - mRadius + 8, centerX, centerY - mRadius + 22f, mPaint)
            }
            canvas.rotate(6f, centerX, centerY)
        }
    }

    private fun drawPointer(canvas: Canvas) {
        val calendar = Calendar.getInstance()
        val hour = calendar.get(Calendar.HOUR)
        val minute = calendar.get(Calendar.MINUTE)
        val second = calendar.get(Calendar.SECOND)
        mPaint.strokeCap = Paint.Cap.ROUND

        canvas.save()
        mPaint.strokeWidth = 8f
        mPaint.color = Color.RED
        val hourAngle = (hour * 60 + minute) / 60f * (360f / 12f)
        canvas.rotate(hourAngle, centerX, centerY)
        canvas.drawLine(centerX, centerY + 60, centerX, centerY - 100, mPaint)
        canvas.restore()

        canvas.save()
        mPaint.strokeWidth = 6f
        mPaint.color = Color.YELLOW
        val minuteAngle = (minute * 60 + second) / 60f * (360f / 60f)
        canvas.rotate(minuteAngle, centerX, centerY)
        canvas.drawLine(centerX, centerY + 80, centerX, centerY - 140, mPaint)
        canvas.restore()

        canvas.save()
        mPaint.strokeWidth = 4f
        mPaint.color = Color.GREEN
        val secondAngle = second * (360f / 60f)
        canvas.rotate(secondAngle, centerX, centerY)
        canvas.drawLine(centerX, centerY + 100, centerX, centerY - 180, mPaint)
        canvas.restore()

        canvas.save()
        mPaint.style = Paint.Style.FILL
        mPaint.color = Color.BLACK
        canvas.drawCircle(centerX, centerY, 6f, mPaint)
        canvas.restore()
    }
}

const val REFRESH = 1
```

+ activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context=".MainActivity">

    <com.base.module.myapplication.widget.ClockView
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        tools:ignore="MissingConstraints" />

</LinearLayout>
```

#### 参考

+ [Android自定义一个属于自己的时间钟表](https://blog.csdn.net/u014741977/article/details/53582090)