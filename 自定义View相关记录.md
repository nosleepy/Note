---
title: 自定义View相关记录
date: 2024-03-22 00:00:00
tags:
categories:
- 安卓
---

#### Canvas相关

+ drawText 方法绘制的文本位置不对

以一个600 * 600的画布为例, 先绘制出边框和中心点, 然后将画布位置移动到(300,300), 继续绘制出文本"100%"

```kotlin
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        mPaint.style = Paint.Style.STROKE
        canvas.drawRect(0f, 0f, 600f, 600f, mPaint)
        mPaint.style = Paint.Style.FILL
        canvas.drawCircle(300f, 300f, 5f, mPaint)
        mPaint.style = Paint.Style.STROKE
        mPaint.textSize = 50f

        canvas.save()
        canvas.translate(300f, 300f)
        canvas.drawText("100%", 0f, 0f, mPaint)
        canvas.restore()
    }
```

从截图可以看出文本实际绘制的位置在(x,y)的右上角, 也就是(0,0)右上角

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/text_draw_before.png)

计算文本的宽高

```kotlin
val rect = Rect()
mPaint.getTextBounds("%100", 0, "%100".length, rect)
val textWidth = rect.right - rect.left
val textHeight = rect.bottom - rect.top
Log.d("wlzhou", "rect = $rect, w = $textWidth, h = $textHeight")

// rect = Rect(2, -37 - 119, 1), w = 117, h = 38
```

想要文本居中绘制,x位置减去文本宽度一半,y位置加上文本高度一半

```kotlin
canvas.drawText("100%", 0f - textWidth / 2, 0f + textHeight / 2, mPaint)
```

最终的效果如下

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/text_draw_after.png)
