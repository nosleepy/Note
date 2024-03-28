---
title: android自定义流式布局
date: 2024-03-28 00:00:00
tags:
categories:
- 安卓
---

#### 实现

FlowViewGroup.kt

```kotlin
class FlowViewGroup @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): ViewGroup(context, attrs, defStyleAttr)  {
    private val childViewList = mutableListOf<MutableList<View>>()
    private var lineViewList = mutableListOf<View>()
    private var hasMeasure = false
    private var lineMaxHeight = 0
    private val lineHeightList = mutableListOf<Int>()
    private var maxWidth = 0
    private var maxHeight = 0

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        val (widthSize, widthMode) = widthMeasureSpec.let {
            MeasureSpec.getSize(it) to MeasureSpec.getMode(it)
        }
        val (heightSize, heightMode) = heightMeasureSpec.let {
            MeasureSpec.getSize(it) to MeasureSpec.getMode(it)
        }
        if (!hasMeasure) {
            hasMeasure = true
            var lineWidthUsed = 0
            for (i in 0 until childCount) {
                val child = getChildAt(i)
                measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0)
                val layoutParams = getChildAt(i).layoutParams as MarginLayoutParams
                if (lineWidthUsed + child.measuredWidth + layoutParams.leftMargin + layoutParams.rightMargin <= widthSize - paddingStart - paddingEnd) {
                    lineViewList.add(child)
                    lineWidthUsed += child.measuredWidth + layoutParams.leftMargin + layoutParams.rightMargin
                    lineMaxHeight = max(lineMaxHeight, child.measuredHeight + layoutParams.topMargin + layoutParams.bottomMargin)
                } else {
                    maxWidth = max(maxWidth, lineViewList.let {
                        var sum = 0
                        for (view in it) {
                            val params = view.layoutParams as MarginLayoutParams
                            sum += view.measuredWidth + params.leftMargin + params.rightMargin
                        }
                        sum
                    })
                    maxHeight += lineMaxHeight
                    childViewList.add(lineViewList)
                    lineHeightList.add(lineMaxHeight)
                    lineViewList = mutableListOf()
                    lineWidthUsed = child.measuredWidth + layoutParams.leftMargin + layoutParams.rightMargin
                    lineMaxHeight = max(0, child.measuredHeight + layoutParams.topMargin + layoutParams.bottomMargin)
                    lineViewList.add(child)
                }
            }
            if (lineViewList.isNotEmpty()) {
                maxWidth = max(maxWidth, lineViewList.let {
                    var sum = 0
                    for (view in it) {
                        val params = view.layoutParams as MarginLayoutParams
                        sum += view.measuredWidth + params.leftMargin + params.rightMargin
                    }
                    sum
                })
                maxHeight += lineMaxHeight
                childViewList.add(lineViewList)
                lineHeightList.add(lineMaxHeight)
            }
        }
        val measureWidth = when (widthMode) {
            MeasureSpec.EXACTLY -> widthSize
            MeasureSpec.AT_MOST -> maxWidth + paddingLeft + paddingRight
            else -> widthSize
        }
        val measureHeight = when (heightMode) {
            MeasureSpec.EXACTLY -> heightSize
            MeasureSpec.AT_MOST -> maxHeight + paddingTop + paddingBottom
            else -> widthSize
        }
        setMeasuredDimension(measureWidth, measureHeight)
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        var curLeft = paddingLeft
        var curTop = paddingTop
        childViewList.forEachIndexed { index, list ->
            list.forEach { view ->
                val layoutParams = view.layoutParams as MarginLayoutParams
                view.layout(curLeft + layoutParams.leftMargin, curTop + layoutParams.topMargin, curLeft + view.measuredWidth + layoutParams.leftMargin, curTop + view.measuredHeight + layoutParams.topMargin)
                curLeft += view.measuredWidth + layoutParams.leftMargin + layoutParams.rightMargin
            }
            curLeft = paddingLeft
            curTop += lineHeightList[index]
        }
    }

    override fun generateLayoutParams(attrs: AttributeSet?): LayoutParams {
        return MarginLayoutParams(context, attrs)
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.base.module.myapplication.widget.FlowViewGroup
        android:background="@color/bg"
        android:padding="40dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:text="111111111"
            android:textSize="30dp"
            android:background="@color/color1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <TextView
            android:text="222222222"
            android:textSize="30dp"
            android:background="@color/color6"
            android:layout_width="wrap_content"
            android:layout_height="60dp"/>

        <TextView
            android:text="3333333333"
            android:textSize="30dp"
            android:background="@color/color2"
            android:layout_width="wrap_content"
            android:layout_height="100dp"/>

        <TextView
            android:text="999999999999999999"
            android:textSize="30dp"
            android:background="@color/color3"
            android:layout_width="wrap_content"
            android:layout_height="80dp"/>

        <TextView
            android:text="1111111111111111"
            android:textSize="30dp"
            android:background="@color/color4"
            android:layout_width="wrap_content"
            android:layout_height="60dp"/>

        <TextView
            android:text="8888888888888888888888888888888888888888888"
            android:textSize="30dp"
            android:background="@color/color1"
            android:layout_width="wrap_content"
            android:layout_height="80dp"/>

        <TextView
            android:text="444444444444444444444444444444444444444444444444444444444444444"
            android:textSize="30dp"
            android:background="@color/color5"
            android:layout_width="wrap_content"
            android:layout_height="80dp"/>

        <TextView
            android:text="555555555555555555555555555"
            android:textSize="30dp"
            android:background="@color/color3"
            android:layout_width="wrap_content"
            android:layout_height="60dp"/>

        <TextView
            android:text="666666666666666"
            android:textSize="30dp"
            android:background="@color/color1"
            android:layout_width="wrap_content"
            android:layout_height="100dp"/>

        <TextView
            android:text="77777777777777"
            android:textSize="30dp"
            android:background="@color/color2"
            android:layout_width="wrap_content"
            android:layout_height="40dp"/>

    </com.base.module.myapplication.widget.FlowViewGroup>

</LinearLayout>
```

#### 示例截图

+ width 为 match_parent, height 为 match_parent

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/flow_viewgroup_mp_mp.png)

+ width 为 wrap_content, height 为 match_parent

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/flow_viewgroup_wc_mp.png)

+ width 为 match_parent, height 为 wrap_content

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/flow_viewgroup_mp_wc.png)

+ width 为 wrap_content, height 为 wrap_content

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/flow_viewgroup_wc_wc.png)

+ width 为 800dp, height 为 600dp

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/flow_viewgroup_800_600.png)