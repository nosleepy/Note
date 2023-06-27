---
title: Kotlin相关
date: 2023-06-27 00:00:00
tags:
categories:
- 安卓
---


#### [高阶函数](https://juejin.cn/post/7208129482095280189)

如果一个函数接收另一个函数作为参数,或者返回值的类型是另一个函数,那么该函数就称为高阶函数

示例

```kotlin
fun getDataFromNet(onSuccess: (data: String) -> Unit) {
    val requestResult = "我是从网络请求拿到的数据"

    onSuccess(requestResult)
}

fun main() {
    getDataFromNet(
        onSuccess = { data: String ->
            println("获取到数据：$data")
        }
    )
}
```

运行结果

```
获取到数据：我是从网络请求拿到的数据
```

#### 参考

+ [Kotlin 中的高阶函数及其应用](https://juejin.cn/post/7208129482095280189)
