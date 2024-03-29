---
title: android自定义中国象棋
date: 2024-03-29 00:00:00
tags:
categories:
- 安卓
---

#### 实现

ChessboardView.kt

```kotlin
enum class ChessState {
    INIT,
    PRESS,
}

enum class ChessColor {
    BLACK,
    RED,
}

enum class ChessTYPE {
    CHE,
    MA,
    XIANG,
    SHI,
    JIANG,
    PAO,
    ZU,
}

data class ChessBean(
    var x: Int,
    var y: Int,
    val type: ChessTYPE,
    val color: ChessColor,
    var state: ChessState = ChessState.INIT,
    var isLoss: Boolean = false
)

data class PositionBean(
    var x: Int,
    var y: Int,
)

class ChessboardView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0,
): View(context, attrs, defStyleAttr)  {
    private var lossX = 10
    private var lossY = 0
    private var curChess: ChessBean? = null
    private var curPosition: PositionBean? = null
    private var textRect = Rect()
    private val linePaint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val chessList = listOf(
        //黑棋
        ChessBean(0, 0, ChessTYPE.CHE, ChessColor.BLACK), ChessBean(1, 0, ChessTYPE.MA, ChessColor.BLACK), ChessBean(2, 0, ChessTYPE.XIANG, ChessColor.BLACK), ChessBean(3, 0, ChessTYPE.SHI, ChessColor.BLACK), ChessBean(4, 0, ChessTYPE.JIANG, ChessColor.BLACK), ChessBean(5, 0, ChessTYPE.SHI, ChessColor.BLACK), ChessBean(6, 0, ChessTYPE.XIANG, ChessColor.BLACK), ChessBean(7, 0, ChessTYPE.MA, ChessColor.BLACK), ChessBean(8, 0, ChessTYPE.CHE, ChessColor.BLACK),
        ChessBean(1, 2, ChessTYPE.PAO, ChessColor.BLACK), ChessBean(7, 2, ChessTYPE.PAO, ChessColor.BLACK),
        ChessBean(0, 3, ChessTYPE.ZU, ChessColor.BLACK), ChessBean(2, 3, ChessTYPE.ZU, ChessColor.BLACK), ChessBean(4, 3, ChessTYPE.ZU, ChessColor.BLACK), ChessBean(6, 3, ChessTYPE.ZU, ChessColor.BLACK), ChessBean(8, 3, ChessTYPE.ZU, ChessColor.BLACK),
        //红棋
        ChessBean(0, 6, ChessTYPE.ZU, ChessColor.RED), ChessBean(2, 6, ChessTYPE.ZU, ChessColor.RED), ChessBean(4, 6, ChessTYPE.ZU, ChessColor.RED), ChessBean(6, 6, ChessTYPE.ZU, ChessColor.RED), ChessBean(8, 6, ChessTYPE.ZU, ChessColor.RED),
        ChessBean(1, 7, ChessTYPE.PAO, ChessColor.RED), ChessBean(7, 7, ChessTYPE.PAO, ChessColor.RED),
        ChessBean(0, 9, ChessTYPE.CHE, ChessColor.RED), ChessBean(1, 9, ChessTYPE.MA, ChessColor.RED), ChessBean(2, 9, ChessTYPE.XIANG, ChessColor.RED), ChessBean(3, 9, ChessTYPE.SHI, ChessColor.RED), ChessBean(4, 9, ChessTYPE.JIANG, ChessColor.RED), ChessBean(5, 9, ChessTYPE.SHI, ChessColor.RED), ChessBean(6, 9, ChessTYPE.XIANG, ChessColor.RED), ChessBean(7, 9, ChessTYPE.MA, ChessColor.RED), ChessBean(8, 9, ChessTYPE.CHE, ChessColor.RED),
    )
    private val positionList = mutableListOf<PositionBean>()

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        linePaint.color = resources.getColor(R.color.black)
        linePaint.style = Paint.Style.STROKE
        canvas.withSave {
            canvas.translate(OFFSET_X, OFFSET_Y)
            // 绘制边框
            linePaint.strokeWidth = 2f
            linePaint.color = resources.getColor(R.color.chess_border)
            canvas.drawRect(0f, 0f, WIDTH, HEIGHT, linePaint)
            // 绘制棋盘线
            linePaint.strokeWidth = 2f
            for (i in 1..8) {
                canvas.drawLine(0f, SPACE * i, WIDTH, SPACE * i, linePaint)
            }
            for (i in 1..7) {
                canvas.drawLine(SPACE * i, 0f, SPACE * i, SPACE * 4, linePaint)
            }
            for (i in 1..7) {
                canvas.drawLine(SPACE * i, SPACE * 5, SPACE * i, SPACE * 9, linePaint)
            }
            canvas.drawLine(SPACE * 3, 0f, SPACE * 5, SPACE * 2, linePaint)
            canvas.drawLine(SPACE * 5, 0f, SPACE * 3, SPACE * 2, linePaint)
            canvas.drawLine(SPACE * 3, SPACE * 7, SPACE * 5, SPACE * 9, linePaint)
            canvas.drawLine(SPACE * 5, SPACE * 7, SPACE * 3, SPACE * 9, linePaint)
            // 绘制棋子
            linePaint.textSize = 30f
            chessList.forEach {
                val content = when (it.type) {
                    ChessTYPE.CHE -> "车"
                    ChessTYPE.MA -> "马"
                    ChessTYPE.XIANG -> if (it.color == ChessColor.BLACK) "象" else "相"
                    ChessTYPE.SHI -> if (it.color == ChessColor.BLACK) "仕" else "士"
                    ChessTYPE.JIANG -> if (it.color == ChessColor.BLACK) "将" else "帅"
                    ChessTYPE.PAO -> "炮"
                    ChessTYPE.ZU -> if (it.color == ChessColor.BLACK) "卒" else "兵"
                }
                linePaint.getTextBounds(content, 0, content.length, textRect)
                val textWidth = textRect.right - textRect.left
                val textHeight = textRect.bottom - textRect.top
                linePaint.style = Paint.Style.FILL
                linePaint.color = resources.getColor(if (it.color == ChessColor.BLACK) R.color.black_chess else R.color.white_chess)
                canvas.drawCircle(it.x * SPACE, it.y * SPACE, RADIUS, linePaint)
                //===press===
                linePaint.color = Color.WHITE
                linePaint.style = Paint.Style.STROKE
                if (it.state == ChessState.PRESS) {
                    //选中棋子状态
                    canvas.drawCircle(it.x * SPACE, it.y * SPACE, RADIUS + 4, linePaint)
                }
                linePaint.style = Paint.Style.FILL
                //===press===
                linePaint.color = Color.WHITE
                canvas.drawText(content, it.x * SPACE - textWidth / 2, it.y * SPACE + textHeight / 2, linePaint)
            }
            //绘制辅助位置
            positionList.forEach {
                linePaint.color = resources.getColor(R.color.flag_color)
                canvas.drawCircle(it.x * SPACE, it.y * SPACE, RADIUS / 4, linePaint)
            }
            //绘制楚河汉界
            linePaint.color = resources.getColor(R.color.chess_border)
            linePaint.textSize = 38f
            canvas.drawText("楚河", SPACE * 1.95f, SPACE * 4.7f, linePaint)
            canvas.drawText("汉界", SPACE * 4.95f, SPACE * 4.7f, linePaint)
        }
    }

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                curPosition = isContainPositionList(event.x - OFFSET_X, event.y - OFFSET_Y)
                curPosition?.let {
                    //被吃掉的棋子移动,红棋左,黑棋右
                    val lossChess = getChess(it.x, it.y)
                    lossChess?.apply {
                        isLoss = true
                        x = lossX
                        y = lossY
                        lossY++
                        if (lossY == 10) {
                            lossX++
                            lossY = 0
                        }
                    }
                    //棋子移动结束
                    curChess?.x = it.x
                    curChess?.y = it.y
                    curChess?.state = ChessState.INIT
                    curChess = null
                    positionList.clear()
                }
                if (curPosition == null) {
                    //清除之前棋子选中状态
                    curChess?.let {
                        it.state = ChessState.INIT
                    }
                    //清除标记提示
                    positionList.clear()
                    //更新选择的棋子
                    curChess = isContainChessList(event.x - OFFSET_X, event.y - OFFSET_Y)
                    curChess?.let {
                        //更新棋子状态
                        it.state = ChessState.PRESS
                        //清除提示位置
                        positionList.clear()
                        //计算提示位置
                        if (!it.isLoss) {
                            when (it.type) {
                                ChessTYPE.ZU -> calcZu(it.x, it.y, it.color)
                                ChessTYPE.CHE -> calcChe(it.x, it.y, it.color)
                                ChessTYPE.PAO -> calcPao(it)
                                else -> {}
                            }
                        }
                    }
                }
            }
            MotionEvent.ACTION_MOVE -> {}
            MotionEvent.ACTION_UP -> {}
        }
        invalidate()
        return true
    }

    private fun isContainChessList(x: Float, y: Float): ChessBean? {
        chessList.forEach { data ->
            if (PointF(x, y).containsCircle(PointF(data.x * SPACE, data.y * SPACE), RADIUS)) {
                return data
            }
        }
        return null
    }

    private fun isContainPositionList(x: Float, y: Float): PositionBean? {
        positionList.forEach { data ->
            if (PointF(x, y).containsCircle(PointF(data.x * SPACE, data.y * SPACE), RADIUS)) {
                return data
            }
        }
        return null
    }

    private fun getChess(x: Int, y: Int): ChessBean? {
        chessList.forEach { data ->
            if (x == data.x && y == data.y) {
                return data
            }
        }
        return null
    }

    private fun isUse(x: Int, y: Int): Boolean {
        chessList.forEach { data ->
            if (data.x == x && data.y == y) {
                return true
            }
        }
        return false
    }

    private fun isSameColor(color: ChessColor, x: Int, y: Int): Boolean {
        val target = chessList.filter {
            it.x == x && it.y == y
        }
        return target[0].color == color
    }

    private fun calcZu(cx: Int, cy: Int, color: ChessColor) {
        if (color == ChessColor.BLACK) {
            positionList.add(PositionBean(cx, cy + 1))
        } else {
            positionList.add(PositionBean(cx, cy - 1))
        }
    }

    private fun calcChe(cx: Int, cy: Int, color: ChessColor) {
        for (i in 1..8) {
            val nx = cx + i
            if (nx !in 0..8) {
                break
            }
            if (!isUse(nx, cy)) {
                positionList.add(PositionBean(nx, cy))
            } else {
                if (!isSameColor(color, nx, cy)) {
                    positionList.add(PositionBean(nx, cy))
                }
                break
            }
        }
        for (i in 1..8) {
            val nx = cx - i
            if (nx !in 0..8) {
                break
            }
            if (!isUse(nx, cy)) {
                positionList.add(PositionBean(nx, cy))
            } else {
                if (!isSameColor(color, nx, cy)) {
                    positionList.add(PositionBean(nx, cy))
                }
                break
            }
        }
        for (i in 1..9) {
            val ny = cy + i
            if (ny !in 0..9) {
                break
            }
            if (!isUse(cx, ny)) {
                positionList.add(PositionBean(cx, ny))
            } else {
                if (!isSameColor(color, cx, ny)) {
                    positionList.add(PositionBean(cx, ny))
                }
                break
            }
        }
        for (i in 1..9) {
            val ny = cy - i
            if (ny !in 0..9) {
                break
            }
            if (!isUse(cx, ny)) {
                positionList.add(PositionBean(cx, ny))
            } else {
                if (!isSameColor(color, cx, ny)) {
                    positionList.add(PositionBean(cx, ny))
                }
                break
            }
        }
    }

    private fun calcPao(chessBean: ChessBean) {
        val cx = chessBean.x
        val cy = chessBean.y
        var hasCross = false
        for (i in 1..8) {
            val nx = cx + i
            if (nx !in 0..8) {
                break
            }
            if (!isUse(nx, cy)) {
                if (!hasCross) {
                    positionList.add(PositionBean(nx, cy))
                }
            } else {
                if (!hasCross) {
                    hasCross = true
                } else {
                    if (!isSameColor(chessBean.color, nx, cy)) {
                        positionList.add(PositionBean(nx, cy))
                        break
                    }
                }
            }
        }
        hasCross = false
        for (i in 1..8) {
            val nx = cx - i
            if (nx !in 0..8) {
                break
            }
            if (!isUse(nx, cy)) {
                if (!hasCross) {
                    positionList.add(PositionBean(nx, cy))
                }
            } else {
                if (!hasCross) {
                    hasCross = true
                } else {
                    if (!isSameColor(chessBean.color, nx, cy)) {
                        positionList.add(PositionBean(nx, cy))
                        break
                    }
                }
            }
        }
        hasCross = false
        for (i in 1..9) {
            val ny = cy + i
            if (ny !in 0..9) {
                break
            }
            if (!isUse(cx, ny)) {
                if (!hasCross) {
                    positionList.add(PositionBean(cx, ny))
                }
            } else {
                if (!hasCross) {
                    hasCross = true
                } else {
                    if (!isSameColor(chessBean.color, cx, ny)) {
                        positionList.add(PositionBean(cx, ny))
                        break
                    }
                }
            }
        }
        hasCross = false
        for (i in 1..9) {
            val ny = cy - i
            if (ny !in 0..9) {
                break
            }
            if (!isUse(cx, ny)) {
                if (!hasCross) {
                    positionList.add(PositionBean(cx, ny))
                }
            } else {
                if (!hasCross) {
                    hasCross = true
                } else {
                    if (!isSameColor(chessBean.color, cx, ny)) {
                        positionList.add(PositionBean(cx, ny))
                        break
                    }
                }
            }
        }
    }

    companion object {
        private const val SPACE = 70f
        private const val WIDTH = SPACE * 8
        private const val HEIGHT = SPACE * 9
        private const val RADIUS = 24f
        private const val OFFSET_X = 360f
        private const val OFFSET_Y = 60f
    }
}

fun PointF.containsCircle(p: PointF, radius: Float): Boolean {
    val deltaX = this.x - p.x
    val deltaY = this.y - p.y
    return hypot(deltaX.toDouble(), deltaY.toDouble()) <= radius
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.base.module.myapplication.widget.ChessboardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/bg"
    tools:context=".MainActivity">

</com.base.module.myapplication.widget.ChessboardView>
```

#### 示例截图

+ 初始状态

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/chessboard_view_init.png)

+ 移动状态

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/chessboard_view_move.png)

+ 吃掉状态

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/chessboard_view_loss.png)