---
title: android自定义锁屏View
date: 2024-03-21 00:00:00
tags:
categories:
- 安卓
---

#### 测试

<video src="https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/unlock_test.mp4"></video>

#### 实现

+ UnLockView.kt

```kotlin
sealed class UnLockState {
    data object SUCCESS: UnLockState()
    data object FAIL: UnLockState()
}

data class UnLockBean(
    val x: Float,
    val y: Float,
    val index: Int,
    var color: Int,
)

interface UnLockCallback {
    fun onResult(input: String)
}

class UnLockView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): View(context, attrs, defStyleAttr) {
    private val bigCirclePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        strokeWidth = 4f
        style = Paint.Style.STROKE
        alpha = (255 * 0.6).toInt()
    }

    private val smallCirclePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        strokeWidth = 4f
        style = Paint.Style.FILL
        alpha = 255
    }

    private val pathPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        strokeWidth = 4f
        style = Paint.Style.STROKE
        alpha = 255
        color = PRESS_COLOR
    }

    var callback: UnLockCallback? = null

    private val bigRadius by lazy { width / (NUMBER * 2) * 0.7f }
    private val smallRadius by lazy { bigRadius * 0.2f }
    private val unLockPoints = arrayListOf<ArrayList<UnLockBean>>()
    private val recordList = arrayListOf<UnLockBean>()
    private val path = Path()
    private var line = Pair(PointF(), PointF())
    private var isDown = false
    private var canTouch = true

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        val diameter = width / NUMBER
        val ratio = NUMBER * 2f
        var index = 1
        for (i in 0 until NUMBER) {
            val list = arrayListOf<UnLockBean>()
            for (j in 0 until NUMBER) {
                list.add(UnLockBean(
                    x = width / ratio + diameter * j,
                    y = height / ratio + diameter * i,
                    index = index++,
                    color = ORIGIN_COLOR,
                ))
            }
            unLockPoints.add(list)
        }
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        val widthSpec = MeasureSpec.getMode(widthMeasureSpec)
        val widthSize = MeasureSpec.getSize(widthMeasureSpec)
        val default = getDefaultSize(context)
        val actualWidth = when(widthSpec) {
            MeasureSpec.EXACTLY -> widthSize
            MeasureSpec.AT_MOST -> default
            MeasureSpec.UNSPECIFIED -> default
            else -> default
        }
        setMeasuredDimension(actualWidth, actualWidth)
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        unLockPoints.forEach {
            it.forEach { data ->
                bigCirclePaint.color = data.color
                smallCirclePaint.color = data.color
                canvas.drawCircle(data.x, data.y, bigRadius, bigCirclePaint)
                canvas.drawCircle(data.x, data.y, smallRadius, smallCirclePaint)
            }
        }
        canvas.drawPath(path, pathPaint)
        if (line.first.x != 0f && line.second.x != 0f) {
            canvas.drawLine(line.first.x, line.first.y, line.second.x, line.second.y, pathPaint)
        }
    }

    private fun isContains(x: Float, y: Float): UnLockBean? {
        unLockPoints.forEach {
            it.forEach { data ->
                if (PointF(x, y).containsCircle(PointF(data.x, data.y), bigRadius)) {
                    return data
                }
            }
        }
        return null
    }

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                if (!canTouch) {
                    return super.onTouchEvent(event)
                }
                val bean = isContains(event.x, event.y)
                isDown = true
                bean?.let {
                    it.color = PRESS_COLOR
                    recordList.add(it)
                    path.moveTo(it.x, it.y)
                    line.first.x = it.x
                    line.first.y = it.y
                }
            }
            MotionEvent.ACTION_MOVE -> {
                if (!isDown) {
                    return super.onTouchEvent(event)
                }
                val bean = isContains(event.x, event.y)
                bean?.let {
                    it.color = PRESS_COLOR
                    if (!recordList.contains(it)) {
                        recordList.add(it)
                        path.lineTo(it.x, it.y)
                        line.first.x = it.x
                        line.first.y = it.y
                    }
                }
                line.second.x = event.x
                line.second.y = event.y
            }
            MotionEvent.ACTION_CANCEL,
            MotionEvent.ACTION_UP -> {
                if (!isDown) {
                    return super.onTouchEvent(event)
                }
                isDown = false
                canTouch = false
                callback?.onResult(recordList.map { it.index }.toString())
            }
        }
        invalidate()
        return true
    }

    fun setUnLockState(state: UnLockState) {
        when (state) {
            UnLockState.SUCCESS -> {
                recordList.forEach { it.color = SUCCESS_COLOR }
                pathPaint.color = SUCCESS_COLOR
            }
            UnLockState.FAIL -> {
                recordList.forEach { it.color = FAIL_COLOR }
                pathPaint.color = FAIL_COLOR
            }
        }
        invalidate()
        postDelayed({
            reset()
        }, 1500)
    }

    private fun reset() {
        canTouch = true
        pathPaint.color = PRESS_COLOR
        path.reset()
        recordList.forEach { it.color = ORIGIN_COLOR }
        recordList.clear()
        line = Pair(PointF(), PointF())
        invalidate()
    }

    companion object {
        private const val NUMBER = 3
        private var ORIGIN_COLOR = Color.GRAY
        private var PRESS_COLOR = Color.YELLOW
        private var SUCCESS_COLOR = Color.GREEN
        private var FAIL_COLOR = Color.RED
    }
}

fun PointF.containsRect(p: PointF, radius: Float): Boolean {
    val isX = this.x <= p.x + radius && this.x >= p.x - radius
    val isY = this.y <= p.y + radius && this.y >= p.y - radius
    return isX && isY
}

fun PointF.containsCircle(p: PointF, radius: Float): Boolean {
    val deltaX = this.x - p.x
    val deltaY = this.y - p.y
    return hypot(deltaX.toDouble(), deltaY.toDouble()) <= radius
}

fun getScreenHeight(context: Context): Int {
    return context.resources.displayMetrics.heightPixels
}

fun getScreenWidth(context: Context): Int {
    return context.resources.displayMetrics.widthPixels
}

fun getDefaultSize(context: Context): Int {
    val width = getScreenWidth(context)
    val height = getScreenHeight(context)
    return if (width >= height) height else width
}
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

    <com.base.module.myapplication.widget.UnLockView
        android:id="@+id/unLock"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        tools:ignore="MissingConstraints" />

</LinearLayout>
```

#### 参考

+ [android自定义View: 九宫格解锁](https://juejin.cn/post/7143137578080796686?searchId=2024032017405048D5CD4872386806E401)