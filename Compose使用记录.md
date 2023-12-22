---
title: Compose使用记录
date: 2023-12-22 00:00:00
tags:
categories:
- 安卓
---

#### remember 函数

```kotlin
@Composable
fun CountTest() {
    var count by mutableStateOf(0)
    Column {
        Text(text = "$count")
        Button(onClick = { count++ }) {
            Text(text = "加1")
        }
    }
}
```

点击按钮时, 状态 count 的值加 1, 触发 Composable 函数的重组, 相当于 `var count by mutableStateOf(0)` 重新执行了一遍, Text 的值不会更新


```kotlin
@Composable
fun CountTestWithRemember() {
    var count by remember {
        mutableStateOf(0)
    }
    Column {
        Text(text = "$count")
        Button(onClick = { count++ }) {
            Text(text = "加1")
        }
    }
}
```

加上 remember 函数, 进入组合时存储 count 的值, 后续重组时直接读取, 退出组合时才移除 count 的值, Text 的值能够更新

```kotlin
@Composable
fun CountTestWithRememberKey() {
    var key by remember { mutableStateOf(System.currentTimeMillis()) }
    var count by remember(key) {
        Log.d("wlzhou", "存储 count 的值")
        mutableStateOf(0)
    }
    Column {
        Text(text = "$count")
        Button(onClick = { count++ }) {
            Text(text = "加1")
        }
        Spacer(modifier = Modifier.height(100.dp))
        Button(onClick = { key = System.currentTimeMillis() }) {
            Text(text = "change key")
        }
    }
}
```

remember 函数接收 key 作为参数, key 改变时 lambda 表达式重新执行

#### 附带效应

+ LaunchedEffect

在某个可组合项的作用域内运行挂起函数

```kotlin
@Composable
fun Test() {
    var count by remember {
        mutableStateOf(0)
    }
    LaunchedEffect(Unit) { //LaunchedEffect(key)
        repeat(Int.MAX_VALUE) {
            count++
            delay(1000)
        }
    }
    Column {
        Text(text = "$count")
    }
}
```

进入组合, 执行协程每秒加 1; 退出组合, 协程自动取消; key 变化协程先取消再执行

```kotlin

```

+ rememberCoroutineScope

获取组合感知作用域，以便在可组合项外启动协程

```kotlin
@Composable
fun Test() {
    var content by remember {
        mutableStateOf("hello")
    }
    val scope = rememberCoroutineScope()
    Column {
        Text(text = content)
        Button(onClick = {
            scope.launch {
                delay(5000)
                content = "world"
            }
        }) {
            Text(text = "change")
        }
    }
}
```

点击按钮启动一个协程, 5秒后更新 text 的值为 world

+ rememberUpdatedState

在效应中引用某个值，该效应在值改变时不应重启

```kotlin
@Composable
fun DialogTest() {
    var lastInteractionTime by remember { mutableStateOf(System.currentTimeMillis()) }
    AutoExitTimer(
        lastInteractionTimeProvider = { lastInteractionTime },
        onExit = { exit() },
    )
    Box(modifier = Modifier
        .fillMaxSize()
        .pointerInput(Unit) {
            detectTapGestures {
                lastInteractionTime = System.currentTimeMillis()
            }
        },
    )
}

@Composable
fun AutoExitTimer(
    lastInteractionTimeProvider: () -> Long,
    onRefresh: (() -> Unit) = {},
    onExit: (() -> Unit) = {},
) {
    val rememberLastInteractionTime by rememberUpdatedState(lastInteractionTimeProvider())
    LaunchedEffect(Unit) {
        repeat(Int.MAX_VALUE) {
            delay(1000)
            val curInteractionTime = System.currentTimeMillis()
            if (curInteractionTime - rememberLastInteractionTime >= 30000) {
                Log.d(ConstantUtil.TAG, "back to home page")
                onExit()
            } else {
                onRefresh()
            }
        }
    }
}
```

弹窗中使用 AutoExitTimer 实现 30 秒无操作自动退出, 协程中需要感知最后交互时间, 但不需要重启协程

+ DisposableEffect

需要清理的效应

```kotlin
@Composable
fun ConfConfigScreen() {
    val viewModel: ConfConfigViewModel = viewModel()
    val lifecycleOwner = LocalLifecycleOwner.current
    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { source, event ->
            when (event) {
                Lifecycle.Event.ON_START -> viewModel.onStart()
                Lifecycle.Event.ON_STOP -> viewModel.onStop()
                else -> {}
            }
        }
        lifecycleOwner.lifecycle.addObserver(observer)
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

在 Composable 监听 Activity 生命周期变化的时候使用, onStart 和 onStop 状态分别添加和移除监听器

+ SideEffect

将 Compose 状态发布为非 Compose 代码

```kotlin
@Composable
fun Test() {
    var count by remember {
        mutableStateOf(0)
    }
    SideEffect {
        Log.d("wlzhou", "count = $count")
    }
    Column {
        Text(text = "$count")
        Button(onClick = { count++ }) {
            Text(text = "加1")
        }
    }
}
```

Composable 函数每次重组都会执行 lambda 表达式代码

+ derivedStateOf

将一个或多个状态对象转换为其他状态

```kotlin
@Composable
fun Test() {
    var content by remember {
        mutableStateOf("")
    }
    val canClick by remember {
        derivedStateOf {
            content.isNotEmpty()
        }
    }
    SideEffect {
        Log.d("wlzhou", "Test recompose")
    }
    Column {
        TextField(
            value = content,
            onValueChange = {
                content = it
            }
        )
        Button(
            onClick = {},
            enabled = canClick,
        ) {
            Text(text = "按钮")
        }
    }
}
```

canClick 状态由 content 状态派生而来, Button 中只关心文本输入框中有没有内容, 不需要获取内容的值, 通过拆分可组合项可以减少重组次数

#### 重组作用域

Compose 编译器做了大量的工作让重组的范围尽可能的小, 它会在编译期间找出所有使用了 State 的代码块, 如果 State 发生了变化, 那么对应的代码块就会重组, 这个受State影响的代码块就是所谓的重组作用域。

```kotlin
@Composable
fun MainScreen() {
    SideEffect { Log.d("wlzhou", "MainScreen recompose") }
    var content by remember {
        mutableStateOf("hello")
    }
    Column {
        ContentA(value = content)
        ContentB()
        Button(onClick = { content = "world" }) {
            Text(text = "change content")
        }
    }
}

@Composable
fun ContentA(value: String) {
    SideEffect { Log.d("wlzhou", "--ContentA recompose") }
    Column(
        modifier = Modifier.width(300.dp).height(100.dp).background(Color.Red)
    ) {
        Text(text = "ContentA的作用域")
        WidgetA(value = value)
    }
}

@Composable
fun ContentB() {
    SideEffect { Log.d("wlzhou", "--ContentB recompose") }
    Column(
        modifier = Modifier.width(300.dp).height(100.dp).background(Color.Green)
    ) {
        Text(text = "ContentB的作用域")
    }
}

@Composable
fun ContentC() {
    SideEffect { Log.d("wlzhou", "--ContentC recompose") }
    Column(
        modifier = Modifier.width(300.dp).height(100.dp).background(Color.Green)
    ) {
        Text(text = "ContentC的作用域")
    }
}

@Composable
fun WidgetA(value: String) {
    SideEffect {
        Log.d("wlzhou", "----WidgetA recompose")
    }
    Text(text = "WidgetA的作用域,value = $value", modifier = Modifier.background(Color.Blue))
}
```

MainScreen 进入组合

```
2023-12-22 14:36:35.270 28857-28857 wlzhou                  com.gs.myapplication                 D  MainScreen recompose
2023-12-22 14:36:35.271 28857-28857 wlzhou                  com.gs.myapplication                 D  --ContentA recompose
2023-12-22 14:36:35.271 28857-28857 wlzhou                  com.gs.myapplication                 D  ----WidgetA recompose
2023-12-22 14:36:35.271 28857-28857 wlzhou                  com.gs.myapplication                 D  --ContentB recompose
2023-12-22 14:36:35.271 28857-28857 wlzhou                  com.gs.myapplication                 D  --ContentC recompose
```

点击按钮更新 content

```
2023-12-22 14:36:49.508 28857-28857 wlzhou                  com.gs.myapplication                 D  MainScreen recompose
2023-12-22 14:36:49.508 28857-28857 wlzhou                  com.gs.myapplication                 D  --ContentA recompose
2023-12-22 14:36:49.508 28857-28857 wlzhou                  com.gs.myapplication                 D  ----WidgetA recompose
```

MainScreen 中 State 变化时, 只有读取了 State 状态的组件才会发生重组

```kotlin
@Composable
fun MainScreen() {
    SideEffect { Log.d("wlzhou", "MainScreen recompose") }
    var content by remember {
        mutableStateOf("hello")
    }
    Column {
//        ContentA(value = content)
        ContentB()
        ContentC()
        Button(onClick = { content = "world" }) {
            Text(text = "change content")
        }
    }
}
```

content 虽然更新了, 但没有组件读取 content 的值, 不会发生重组

**发生重组的条件**

1. 组件外部的传参发生了变化
2. 组件内部的 State 发生了变化, 而且组件读取了这个状态

#### 重组的优化

+ 使用派生状态来降低重组次数

```kotlin
@Composable
fun Test() {
    SideEffect { Log.d("wlzhou", "Test recompose") }
    var score by remember {
        mutableStateOf(60)
    }
    Column {
        Text(text = if (score >= 60) "及格" else "不及格")
        Button(onClick = { score++ }) {
            Text(text = "+1")
        }
        Button(onClick = { score-- }) {
            Text(text = "-1")
        }
    }
}
```

分数计数器, 得分大于等于 60 显示及格, 小于 60 显示不及格, 每次加1和减1都会发生重组, 使用 derivedStateOf 改进, 只有分数及格或者不及格时才会发生重组

```kotlin
@Composable
fun MainScreen() {
    SideEffect { Log.d("wlzhou", "Test recompose") }
    var score by remember {
        mutableStateOf(60)
    }
    val isPass by remember {
        derivedStateOf {
            score >= 60
        }
    }
    Column {
        Text(text = if (isPass) "及格" else "不及格")
        Button(onClick = { score++ }) {
            Text(text = "+1")
        }
        Button(onClick = { score-- }) {
            Text(text = "-1")
        }
    }
}
```

+ 使用lambda间接传值/跳过阶段

**父组件向子组件传值**

```kotlin
@Composable
fun ParentScreen() {
    SideEffect {
        Log.d("wlzhou", "parent recompose")
    }
    var content by remember {
        mutableStateOf("hello")
    }
    Column {
        Button(onClick = { content = "world" }) {
            Text(text = "父组件向子组件传值")
        }
        ChildScreen(value = content)
    }
}

@Composable
fun ChildScreen(value: String) {
    SideEffect {
        Log.d("wlzhou", "--child recompose")
    }
    Text(text = "我是父组件传来的值: $value")
}
```

父组件中点击按钮更新 content 的值使子组件重组, 但父组件中读取了 content 的值, 同样发生重组, 使用 lambda 间接传值改进

```kotlin
@Composable
fun ParentScreen() {
    SideEffect {
        Log.d("wlzhou", "parent recompose")
    }
    var content by remember {
        mutableStateOf("hello")
    }
    Column {
        Button(onClick = { content = "world" }) {
            Text(text = "父组件向子组件传值")
        }
        ChildScreen(valueProvider = { content })
    }
}

@Composable
fun ChildScreen(valueProvider: () -> String) {
    SideEffect {
        Log.d("wlzhou", "--child recompose")
    }
    Text(text = "我是父组件传来的值: ${valueProvider()}")
}
```

**纯背景颜色改变**

电子门牌项目中存在空闲、即将开始、会议中三个状态, 状态改变背景颜色也发生变化

+ 方式1: 会议状态改变发生重组

```kotlin
@Composable
fun MainScreen(
    confStateProvider: () -> RemoteConfState,
) {
    Box(modifier = Modifier
        .fillMaxSize()
        .background(
            color = when (confStateProvider()) {
                is RemoteConfState.Idle -> Color(0xFF00a645)
                is RemoteConfState.ReadyFlag -> Color(0xFFfd9a38)
                is RemoteConfState.Ready -> Color(0xFFfd9a38)
                is RemoteConfState.Run -> Color(0xFFab021b)
                is RemoteConfState.Disable -> Color(0xFFaaaaaa)
            }
        )
    )
}
```

+ 方式2: 跳过组合和布局阶段,仅仅需要绘制

```kotlin
@Composable
fun MainScreen(
    confStateProvider: () -> RemoteConfState,
) {
    Box(modifier = Modifier
        .fillMaxSize()
        .drawBehind {
            drawRect(color = when (confStateProvider()) {
                is RemoteConfState.Idle -> Color(0xFF00a645)
                is RemoteConfState.ReadyFlag -> Color(0xFFfd9a38)
                is RemoteConfState.Ready -> Color(0xFFfd9a38)
                is RemoteConfState.Run -> Color(0xFFab021b)
                is RemoteConfState.Disable -> Color(0xFFaaaaaa)
            })
        }
    )
}
```

#### 参考

+ [Compose 中的附带效应](https://developer.android.google.cn/jetpack/compose/side-effects?hl=de#sideeffect-publish)
+ [Compose：从重组谈谈页面性能优化思路，狠狠优化一笔](https://juejin.cn/post/7263341982407868472?searchId=20231219225441BC237881A15EC01F69ED#heading-5)
+ [Compose：附带效应 SideEffect 和 Compose 中的协程](https://blog.csdn.net/qq_31339141/article/details/129649942)
+ [Compose学习笔记（五）：derivedStateOf和remember的使用](https://juejin.cn/post/7094029276592226312#heading-5)