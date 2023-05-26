---
title: Kotlin跨平台开发实践
date: 2023-05-26 00:00:00
tags:
categories:
- 安卓
---

#### KMM环境搭建

##### KMM是什么

+ KMM即Kotlin Multiplatform Mobile ,是由Jetbrains提供的跨平台移动开发SDK，借助 Kotlin的跨平台能力，可以使用一个工程为多个平台编译。
+ 使用 KMM，具备灵活性的同时也保留了原生编程的优势。为 Android/iOS 应用程序的业务逻辑代码使用单一的代码库，仅在需要的时候编写平台特定代码，例如实现原生的 UI，使用平台特定 API 等等。
+ KMM 可以和你的工程无缝集成。共享代码，使用 Kotlin 编写，使用 Kotlin/JVM 编译成 JVM 字节码，使用 Kotlin/Native 编译成二进制，所以你可以和使用其他一般类库一样使用 KMM 业务逻辑模块。

![](https://kotlinlang.org/docs/images/basic-project-structure.png)

##### KMM环境搭建

AndroidStudio安装KMM插件

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/install_kmm_plugin.png)

新建Kotlin Multiplatform App类型的项目

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/new_kmm_project.png)

创建的KMM项目包含androidApp、iosApp、shared三个模块

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/kmm_project.png)

+ shared 模块：存放 Android/iOS 通用业务逻辑代码的 Kotlin 模块，会被编译为 Android library 和 iOS framework。使用 Gradle 进行构建。
+ androidApp 模块：Android 应用的 Kotlin 模块。使用 Gradle 构建。
+ iosApp 模块：构建 iOS 应用的 Xcode 工程。

项目的 build.gradle.kts 文件

```groovy
plugins {
    id("com.android.application").version("7.2.0").apply(false)
    id("com.android.library").version("7.2.0").apply(false)
    kotlin("android").version("1.8.10").apply(false)
    kotlin("multiplatform").version("1.8.10").apply(false)
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

shared 模块

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/kmm_project_shared_module.png)

build.gradle.kts 文件

```groovy
plugins {
    kotlin("multiplatform")
    id("com.android.library")
}

kotlin {
    android {
        compilations.all {
            kotlinOptions {
                jvmTarget = "1.8"
            }
        }
    }
    
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach {
        it.binaries.framework {
            baseName = "shared"
        }
    }

    sourceSets {
        val commonMain by getting
        val androidMain by getting
        val iosX64Main by getting
        val iosArm64Main by getting
        val iosSimulatorArm64Main by getting
        val iosMain by creating {
            dependsOn(commonMain)
            iosX64Main.dependsOn(this)
            iosArm64Main.dependsOn(this)
            iosSimulatorArm64Main.dependsOn(this)
        }
    }
}

android {
    namespace = "com.example.kmmapp"
    compileSdk = 33
    defaultConfig {
        minSdk = 21
    }
}
```

包含了 Android 和 iOS 的公用代码。但是，为了在 Android/iOS 上实现同样的逻辑，有时候你不得不写两份版本特定代码，例如蓝牙，Wifi 等等。为了处理这种情况，Kotlin 提供了 expect/actual 机制。shared 模块的源代码按三个源集进行分类：

+ commonMain：存储为所有平台工作的代码，包括 expect 声明。
+ androidMain：存储 Android 的特定代码，包括 actual 实现。
+ iosMain：存储 iOS 的特定代码，包括 actual 实现。

expect/actual 关键字使用

对于跨平台应用来说，版本特定代码是很常见的。例如实现一个功能：获取当前应用运行的设备名称(android或者ios)和版本号。

+ commonMain

```kotlin
interface Platform {
    val name: String
}

expect fun getPlatform(): Platform
```

+ androidMain

```kotlin
class AndroidPlatform : Platform {
    override val name: String = "Android ${android.os.Build.VERSION.SDK_INT}"
}

actual fun getPlatform(): Platform = AndroidPlatform()
```

+ iosMain

```kotlin
import platform.UIKit.UIDevice

class IOSPlatform: Platform {
    override val name: String = UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
}

actual fun getPlatform(): Platform = IOSPlatform()
```

commonMain 中公开获取设备名称和版本号的方法

```kotlin
class Greeting {
    private val platform: Platform = getPlatform()

    fun greet(): String {
        return "Hello, ${platform.name}!"
    }
}
```

##### 跨平台运行测试

androidApp 和 iosApp 模块统一调用 common 模块的 Greeting#greet 方法显示设备名称和版本号

+ Android平台

MainActivity.kt 文件

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApplicationTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colors.background
                ) {
                    GreetingView(Greeting().greet())
                }
            }
        }
    }
}

@Composable
fun GreetingView(text: String) {
    Text(text = text)
}
```

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/kmm_android.png)

+ Ios平台

ContentView.swift 文件

```swift
struct ContentView: View {
	let greet = Greeting().greet()

	var body: some View {
		Text(greet)
	}
}
```

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/kmm_ios.png)

#### Jetpack Compose 跨平台开发

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose-multiplatform.png)

+ Jetpack Compose 是 Google 针对 Android 推出的新一代声明式 UI 工具包，完全基于 Kotlin 打造，天然具备了跨平台的使用基础。
+ JetBrains 以 Jetpack Compose（compose-android）为基础，相继发布了 compose-desktop 和 compose-web ，使 Compose 可以运行在更多不同平台。
+ Compose Multiplatform（compose-jb）本质上是将 compose-desktop，compose-web 以及 compose-android 三者进行了整合，开发者可以在单个工程中使用同一套 Artifacts 开发出运行在 Android，Desktop（Windows, macOS, LInux）以及 Web 等多端的应用程序，工程中可以实现大部分代码的共享以此达到跨平台开发的目的。

##### 单平台 Compose-Desktop 应用创建

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/new_compose_desktop_app.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_desktop_project.png)

build.gradle.kts 文件

```groovy
import org.jetbrains.compose.desktop.application.dsl.TargetFormat

plugins {
    kotlin("multiplatform")
    id("org.jetbrains.compose")
}

group = "com.example"
version = "1.0-SNAPSHOT"

repositories {
    google()
    mavenCentral()
    maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
}

kotlin {
    jvm {
        jvmToolchain(11)
        withJava()
    }
    sourceSets {
        val jvmMain by getting {
            dependencies {
                implementation(compose.desktop.currentOs)
            }
        }
    }
}

compose.desktop {
    application {
        mainClass = "MainKt"
        nativeDistributions {
            targetFormats(TargetFormat.Dmg, TargetFormat.Msi, TargetFormat.Deb)
            packageName = "desktopDemo"
            packageVersion = "1.0.0"
        }
    }
}
```

入口 Main.kt 文件，Desktop 应用 Compose 组件和 Android 应用一致

```kotlin
@Composable
@Preview
fun App() {
    var text by remember { mutableStateOf("Hello, World!") }

    MaterialTheme {
        Button(onClick = {
            text = "Hello, Desktop!"
        }) {
            Text(text)
        }
    }
}

fun main() = application {
    Window(onCloseRequest = ::exitApplication) {
        App()
    }
}
```

执行 ./gradlew run 命令


![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_desktop_demo.png)

点击按钮后 Hello,World 变成 Hello,Desktop

##### 单平台 Compose-Web 应用创建

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/new_compose_web_app.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_web_project.png)

build.gradle.kts 文件

```groovy
plugins {
    kotlin("multiplatform")
    id("org.jetbrains.compose")
}

group = "com.example"
version = "1.0-SNAPSHOT"

repositories {
    google()
    mavenCentral()
    maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
}

kotlin {
    js(IR) {
        browser {
            testTask {
                testLogging.showStandardStreams = true
                useKarma {
                    useChromeHeadless()
                    useFirefox()
                }
            }
        }
        binaries.executable()
    }
    sourceSets {
        val jsMain by getting {
            dependencies {
                implementation(compose.web.core)
                implementation(compose.runtime)
            }
        }
    }
}
```

Web 应用还需要创建资源文件 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sample</title>
</head>
<body>
<div id="root"></div>
<script src="webDemo.js"></script>
</body>
</html>
```

入口 Main.kt 文件，渲染 id 为 "root" 的 Div 元素内容，webDemo.js 是 Kotlin/JS 编译器生成的脚本文件。

```kotlin
fun main() {
    var count: Int by mutableStateOf(0)

    renderComposable(rootElementId = "root") {
        Div({ style { padding(25.px) } }) {
            Button(attrs = {
                onClick { count -= 1 }
            }) {
                Text("-")
            }

            Span({ style { padding(15.px) } }) {
                Text("$count")
            }

            Button(attrs = {
                onClick { count += 1 }
            }) {
                Text("+")
            }
        }
    }
}
```

执行 ./gradlew jsBrowserRun 命令，示例程序是一个加减计数器

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_web_demo.png)

使用 F12 弹出控制台，可以发现 compose-web 相关的组件被解析成 dom 元素。

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/compose_web_check.png)

##### 多平台 Compose 应用

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/new_multiple_project.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/multiple_project.png)

android 和 desktop 模块共享 common 模块的 UI 和 业务逻辑，做到完全复用，IDEA 默认只会创建 android 加 desktop 的多平台 compose 应用，需要手动添加 web 和 ios 平台。

#### ContactDemo实践

使用到的 Kotlin 跨平台库

+ [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines)
+ [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)
+ [Ktor](https://github.com/ktorio/ktor)
+ [SQLDelight](https://cashapp.github.io/sqldelight/2.0.0-alpha05/)
+ [Napier](https://github.com/AAkira/Napier/)
+ [PreCompose](https://github.com/Tlaster/PreCompose)

项目结构

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/multiplatform_app_project.png)

+ backend：使用Ktor框架搭建服务端，响应Http请求。
+ shared：公共模块，平台差异通过expect/actual解决。
+ androidApp：android端相关代码。
+ jsApp：web端相关代码。
+ desktopApp：desktop端相关代码。
+ iosApp：ios端相关代码，测试需要mac环境。

多平台运行截图

+ android

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/android_test1.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/android_test2.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/android_test3.png)

+ desktop

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/desktop_test1.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/desktop_test2.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/desktop_test3.png)

+ web

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/web_test1.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/web_test2.png)

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/web_test3.png)

#### 参考

+ [multiplatform-mobile-getting-started](https://kotlinlang.org/docs/multiplatform-mobile-getting-started.html#0)
+ [10个问题带你看懂 Compose 跨平台](https://juejin.cn/post/7039698227397918756)
+ [Kotlin Multiplatform libraries](http://libs.kmp.icerock.dev/)
+ [快速入门KMM和Compose Multiplatform](https://zhuanlan.zhihu.com/p/602992799)