---
title: Fragment生命周期
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

**Fragment生命周期图**

![](https://img2020.cnblogs.com/blog/595094/202004/595094-20200429150020682-753845082.png)

**Fragment#add**

往容器内添加fragment,添加重复的fragment会报错。

**Fragment#replace**

移除容器内的所有fragment,然后添加fargment。

**Fragment#show**

显示在容器里的fargment,生命周期不会变化。

**Fragment#hide**

隐藏在容器里的fargment,生命周期不会变化。

**Fragment#addToBackStack**

操作后按返回键可以回到当前fragment视图。

**Activity和Fragment生命周期变化**

```
Activity       -onCreate
Fragment  -onAttach
Fragment  -onCreate
Fragment  -onCreateView
Fragment  -onActivityCreated
Activity       -onStart
Fragment  - onStart
Activity      -onResume
Fragment  -onResume
Fragment  -onPause
Activity       -onPause
Fragment  -onStops
Activity      -onStop
Fragment  -onDestroyView
Fragment  -onDestroy
Fragment  -onDetach
Activity       -onDestroy
```
