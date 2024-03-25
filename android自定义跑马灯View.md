---
title: android自定义跑马灯View
date: 2024-03-25 00:00:00
tags:
categories:
- 安卓
---

#### 测试

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/aperture_test.png)

#### 实现

+ ApertureView.kt

```kotlin
class ApertureView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): View(context, attrs, defStyleAttr) {
    private val paint = Paint()
    private val size by lazy { width / 20f }
    private val color1 by lazy {
        LinearGradient(width / 2f, height / 2f, 0f, 0f, intArrayOf(Color.TRANSPARENT, Color.RED), floatArrayOf(0f, 1f), Shader.TileMode.CLAMP)
    }
    private val color2 by lazy {
        LinearGradient(width / 2f, height / 2f, width * 1f, height * 1f, intArrayOf(Color.TRANSPARENT, Color.GREEN), floatArrayOf(0f, 1f), Shader.TileMode.CLAMP)
    }

    private val animator by lazy {
        val animator = ObjectAnimator.ofFloat(this, "currentSpeed", 0f, 360f)
        animator.repeatCount = ObjectAnimator.INFINITE
        animator.repeatMode = ObjectAnimator.RESTART
        animator.interpolator= null
        animator.duration = 2000
        animator
    }

    var currentSpeed = 0f
        set(value) {
            field = value
            invalidate()
        }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        setMeasuredDimension(300, 300)
    }

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        outlineProvider = object : ViewOutlineProvider() {
            override fun getOutline(view: View, outline: Outline) {
                outline.setRoundRect(0, 0, view.width, view.height, 20f)
            }
        }
        setClipToOutline(true)
        animator.start()
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        paint.style = Paint.Style.FILL
        canvas.drawColor(Color.YELLOW)
        canvas.withSave {
            canvas.rotate(currentSpeed, width / 2f, height / 2f)
            paint.color = Color.RED
            paint.shader = color1
            canvas.drawRect(-width / 2f, -height / 2f, width / 2f, height / 2f, paint)
            paint.shader = null
            paint.color = Color.GRAY
            paint.shader = color2
            canvas.drawRect(width / 2f, height /2f, width / 2f * 3, height / 2f * 3, paint)
            paint.shader = null
        }
        canvas.withSave {
            paint.color = Color.WHITE
            canvas.translate(size, size)
            canvas.drawRoundRect(0f, 0f, width - size * 2, height - size * 2, 20f, 20f, paint)
        }
    }
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

    <com.base.module.myapplication.widget.ApertureView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

#### 参考

+ [android 自定义view 跑马灯-光圈效果](https://juejin.cn/post/7171030095866363934?searchId=20240325103302AF47A7D5A2552D85650C)