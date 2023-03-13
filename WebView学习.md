---
title: WebView学习
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### WebView创建

+ xml配置

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <WebView
        android:id="@+id/wv_webview"
        android:layout_width="match_parent"
        android:layout_height="500dp" />

</LinearLayout>
```

+ new方式

```java
WebView webView = new WebView(this);
```

#### 权限配置

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <!-- 添加网络权限 -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
         <!-- 允许明文传输 -->
        android:usesCleartextTraffic="true">
    </application>

</manifest>
```

#### 基本使用

```java
webView.settings.javaScriptEnabled = true //允许与Javascript交互
webView.loadUrl("http://www.baidu.com") //访问网页
//系统默认会通过手机浏览器打开网页，为了能够直接通过WebView显示网页，则必须设置
webView.webViewClient = object : WebViewClient() {
    override fun shouldOverrideUrlLoading(view: WebView, url: String): Boolean {
    view.loadUrl(url) //使用WebView加载显示url
    	return true
    }

    override fun onPageStarted(view: WebView, url: String, favicon: Bitmap) {
    	//设定加载开始的操作
    }

    override fun onPageFinished(view: WebView, url: String) {
    	//设定加载结束的操作
    }

    override fun onLoadResource(view: WebView, url: String) {
    	//设定加载资源的操作
    }

    override fun onReceivedError(view: WebView, errorCode: Int, description: String, failingUrl: String) {
    	//加载页面的服务器出现错误
    }

    override fun onReceivedSslError(view: WebView, handler: SslErrorHandler, error: SslError) {
    	//处理https请求出现错误
    }
}
webView.webChromeClient = object : WebChromeClient() {
    override fun onProgressChanged(view: WebView, newProgress: Int) { //获取网页加载进度
    	//newProgress
    }

    override fun onReceivedTitle(view: WebView, title: String) { //获取网页标题
    	//title
    }

    override fun onJsAlert(view: WebView, url: String, message: String, result: JsResult): Boolean { //响应js的alert函数
    	return true
    }
}
```

#### 和JavaScript交互

+ 新建 test.html 页面

project/app/src/main 目录下增加 assets 目录,编写 test.html 页面

+ Java 调用 JS

html 页面添加 javaCallJs 方法

```html
<script type="text/javascript">
    function javaCallJs(message){
    	alert(message);
    }
   function add(a, b) {
        return a + b;
	}
</script>
```

Java 代码通过 `javascript:XXX` 的格式调用

```kotlin
//方法1
//webView.loadUrl("javascript:javaCallJs('Message From Java')")
//方法2
webView.evaluateJavascript("javascript:add(1, 99)") { value ->
	Toast.makeText(this@MainActivity, "add(1, 99) = $value", Toast.LENGTH_SHORT).show()
}
```

+ Js 调用 Java

html 页面加入按钮用于访问 Java 方法

```html
<button type="button" onClick="window.main.jsCallJava('Message From Js')" >Js Call Java</button>
```

Java 代码添加 `@JavascriptInterface` 注解的方法

```kotlin
@JavascriptInterface
fun jsCallJava(message: String) {
	Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
}
```

WebView 注入接口方法

```kotlin
mWebView.addJavascriptInterface(MainActivity.this, "main");
```

+ 完整代码

html 部分

```html
<html>
<head>
    <meta http-equiv="Content-Type"  content="text/html;charset=UTF-8">
    <script type="text/javascript">
        function javaCallJs(message){
            alert(message);
        }
        function add(a, b) {
            return a + b;
        }
    </script>
</head>
<body>
<button type="button" onClick="window.main.jsCallJava('Message From Js')" >Js Call Java</button>
</body>
</html>
```

Java 部分

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var webView: WebView

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        webView = findViewById<WebView>(R.id.wv).apply {
            settings.javaScriptEnabled = true
            loadUrl("file:///android_asset/html/test.html")
            addJavascriptInterface(this@MainActivity, "main")
            webViewClient = object : WebViewClient() {
                override fun shouldOverrideUrlLoading(view: WebView, url: String): Boolean {
                    view.loadUrl(url)
                    return true
                }
            }
            webChromeClient = object : WebChromeClient() {
                override fun onJsAlert(view: WebView, url: String, message: String, result: JsResult): Boolean {
                    AlertDialog.Builder(this@MainActivity)
                        .setTitle("")
                        .setMessage(message)
                        .setPositiveButton(android.R.string.ok) { dialog, which -> result.confirm() }
                        .setCancelable(false)
                        .create().show()
                    return true
                }
            }
        }
    }

    fun callJs(view: View) {
        //方法1
        //webView.loadUrl("javascript:javaCallJs('Message From Java')")
        //方法2
        webView.evaluateJavascript("javascript:add(1, 99)") { value ->
            Toast.makeText(this@MainActivity, "add(1, 99) = $value", Toast.LENGTH_SHORT).show()
        }
    }

    @JavascriptInterface
    fun jsCallJava(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }
}
```

#### 内存泄露问题

+ 不要使用 xml 方式创建，而是使用代码把 WebView 给 new 出来
+ 不要让 WebView 持有对 Activity/Fragment 的 Context 引用（核心）
+ 销毁时，停止 WebView 的加载，并从父控件中将其移除

```java
// 让 WebView 使用 ApplicationContext
val webview = WebView(this.applicationContext)

override fun onDestroy() {
    // webview?.loadDataWithBaseURL(null, "", "text/html", "utf-8", null)
    // webview?.clearView()
    webview?.loadUrl("about:blank")
    webview?.parent?.let {
        (it as ViewGroup).removeView(webview)
    }
    webview?.stopLoading()
    webview?.settings?.javaScriptEnabled = false
    webview?.clearHistory()
    webview?.clearCache(true)
    webview?.removeAllViewsInLayout()
    webview?.removeAllViews()
    webview?.webViewClient = null
    webview?.webChromeClient = null
    webview?.destroy()
    webview = null
    super.onDestroy()
}
```

#### 注意

+ Js 调用 Java 方法运行在 JavaBridge 线程
+ Java 调用 Js 方法必须在 main 主线程运行

#### 参考

+ [解决WebView内存泄漏【最干货】](https://blog.csdn.net/CSDN_LQR/article/details/115370719)