---
title: 安卓View的滑动
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### View的坐标系

屏幕上的默认坐标系。

![](http://gcsblog.oss-cn-shanghai.aliyuncs.com/blog/2019-04-29-071020.jpg?gcssloop)

View的坐标系统是相对于父控件而言的。

```
getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```

![](http://gcsblog.oss-cn-shanghai.aliyuncs.com/blog/2019-04-29-071021.jpg?gcssloop)

#### MotionEvent 中 get 和 getRaw 的区别

```
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();
```

![](http://gcsblog.oss-cn-shanghai.aliyuncs.com/blog/2019-04-29-71022.jpg?gcssloop)

#### View 的滑动方式

假如我们要把一个 ImageView 向右移动 300px。

首先自定义ImageView。

```java
public class MyImageView extends ImageView {
    private Scroller scroller;

    public MyImageView(Context context) {
        this(context, null);
    }

    public MyImageView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        scroller = new Scroller(context);
    }

    private int lastX;
    private int lastY;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getRawX();
        int y = (int) event.getRawY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                int dx = lastX - x; //向右移动<0
                int dy = lastY - y; // 向下移动<0
                /*
                //方法1
                scrollBy(dx, dy); // 移动ImageView内容
                ((View) getParent()).scrollBy(dx, dy); // 移动View的Parent，间接移动View本身
                //方法2
                layout(getLeft() - dx, getTop() - dy, getRight() - dx, getBottom() - dy);
                //方法3
                offsetLeftAndRight(-dx);
                offsetTopAndBottom(-dy);
                //方法4
                ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft() - dx;
                layoutParams.topMargin = getTop() - dy;
                setLayoutParams(layoutParams);
                */
                break;
        }
        lastX = x;
        lastY = y;
        return true;
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

  <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:background="#FFD7D7">

        <com.grandstream.myapplication.MyImageView
            android:id="@+id/iv"
            android:src="@drawable/img"
            android:layout_width="50dp"
            android:layout_height="50dp"/>

    </LinearLayout>

</LinearLayout>
```

1. scrollTo和scrollBy

View方法调用移动的是View的内容,ViewGroup方法调用移动的是View本身。

```java
View.scrollTo(int x, int y) // 基于起始位置的绝对滑动
View.scrollBy(int x, int y) // 基于当前位置的相对滑动

ViewGroup.scrollTo(int x, int y)
ViewGroup.scrollBy(int x, int y)

// x > 0 左移，x < 0 右移
// y > 0 上移，y < 0 下移
```

2. 使用layout

```java
View.layout(int l, int t, int r, int b) // 相对于父View走上右下的距离
```

3. offsetLeftAndRight 和 offsetTopAndBottom

```java
View.offsetLeftAndRight(int offset) // offset > 0 右移,offset < 0 左移
View.offsetTopAndBottom(int offset) // offset > 0 下移,offset < 0 上移
```

4. LayouParams

```java
ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
layoutParams.leftMargin = getLeft() - dx;
layoutParams.topMargin = getTop() - dy;
setLayoutParams(layoutParams);
```

5. Scroller

可以指定持续时间，实现弹性滑动。

```java
public class MyImageView extends ImageView {
    private Scroller scroller;

    public MyImageView(Context context) {
        this(context, null);
    }

    public MyImageView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        scroller = new Scroller(context);
    }

    public void slide(int x, int y) { //公开slide方法弹性移动view
        scroller.startScroll(((View)getParent()).getScrollX(), ((View)getParent()).getScrollY(), x, y, 4000);
        invalidate();
    }

    @Override
    public void computeScroll() {
        if (scroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(scroller.getCurrX(), scroller.getCurrY());
            invalidate();
        }
    }
}
```

6. 动画实现

+ 补间动画

只是改变View的内容，移动之后不能进行点击触摸事件。

```java
//从左上角(0,0)移动到右下角(200, 200)的位置
TranslateAnimation animation = new TranslateAnimation(0, 200, 0, 200);
animation.setDuration(4000);
animation.setFillAfter(true);
 iv.startAnimation(animation);
```

+ 属性动画

真正改变View的位置。

```java
//向右移动150px
ObjectAnimator.ofFloat(iv, "translationX", 0, 150).setDuration(4000).start();
```

#### 滑动冲突解决

自定义 PullLayout 继承 ViewGroup

```java
public class PullLayout extends ViewGroup {
    private View header;
    private View footer;
    private TextView tip1;
    private ProgressBar bar1;

    public PullLayout(Context context) {
        this(context, null);
    }

    public PullLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        header = LayoutInflater.from(context).inflate(R.layout.header, null);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.MATCH_PARENT
        );
        tip1 = header.findViewById(R.id.tip1);
        bar1 = header.findViewById(R.id.bar1);
        header.setLayoutParams(params);
        addView(header);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        measureChildren(widthMeasureSpec, heightMeasureSpec);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int count = getChildCount();
        int curHeight = 0;
        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            int width = child.getMeasuredWidth();
            int height = child.getMeasuredHeight();
            if (child == header) {
                child.layout(0, -height, width, 0);
            } else {
                child.layout(0, curHeight, width, curHeight + height);
                curHeight += height;
            }
        }
    }

    private Handler handler = new Handler();

    private int mLastY;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                int dy = mLastY - y;
                scrollBy(0, dy);
                if (Math.abs(getScrollY()) >= 100) {
                    tip1.setText("松开刷新");
                }
                break;
            case MotionEvent.ACTION_UP:
                if (Math.abs(getScrollY()) >= 100) {
                    scrollTo(0, -50);
                    tip1.setVisibility(View.GONE);
                    bar1.setVisibility(View.VISIBLE);
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            try {
                                Thread.sleep(1000);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                            handler.post(new Runnable() {
                                @Override
                                public void run() {
                                    scrollTo(0, 0);
                                    tip1.setVisibility(View.VISIBLE);
                                    bar1.setVisibility(View.GONE);
                                    tip1.setText("向下拉动刷新");
                                }
                            });
                            if (callback != null) {
                                callback.refresh();
                            }
                        }
                    }).start();
                } else {
                    scrollTo(0, 0);
                    tip1.setVisibility(View.VISIBLE);
                    bar1.setVisibility(View.GONE);
                    tip1.setText("向下拉动刷新");
                }
                break;
        }
        mLastY = y;
        return true;
    }

    private Callback callback;

    public void setCallback(Callback callback) {
        this.callback = callback;
    }

    public interface Callback {
        void refresh();
    }
}
```

布局结构为 PullLayout 内嵌一个 ListView

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.grandstream.myapplication.PullLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/pull"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:text="Hello World"
        android:background="#F8D9D9"
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="50dp"/>

    <ListView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</com.grandstream.myapplication.PullLayout>
```

希望 ListView 展示的第一项可见时，从 ListView 内部下拉可以触发刷新事件。

1. 外部拦截

PullLayout 重写 onInterceptTouchEvent 拦截触摸事件

```java
    private int mY;

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercept = false;
        int y = (int) ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                if (y > mY) {
                    View child = getChildAt(1);
                    AdapterView adapterChild = (AdapterView) child;
                    if (adapterChild.getFirstVisiblePosition() == 0 || adapterChild.getChildAt(0).getTop() == 0) {
                        intercept = true;
                    }
                }
                break;
            case MotionEvent.ACTION_UP: {
                break;
            }
        }
        mY = y;
        mLastY = y;
        return intercept;
    }
```

2. 内部拦截

PullLayout 不拦截按下事件

```java
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        mLastY = (int) ev.getY();
        int action = ev.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
    }
```

自定义 MyListView 继承 ListView，重写 dispatchTouchEvent 方法处理事件分发，自定义逻辑让父 View 是否拦截事件。

```java
public class MyListView extends ListView {
    public MyListView(Context context) {
        super(context);
    }

    public MyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    private int mLastX;
    private int mLastY;

    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                getParent().requestDisallowInterceptTouchEvent(true); //请求父View不拦截
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                int dx = mLastX - x;
                int dy = mLastY - y;
                if (dy < 0) { //下滑
                    if (getFirstVisiblePosition() == 0 || getChildAt(0).getTop() == 0) {
                        getParent().requestDisallowInterceptTouchEvent(false); //需要父View拦截
                    }
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                break;
            }
        }
        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }
}
```
