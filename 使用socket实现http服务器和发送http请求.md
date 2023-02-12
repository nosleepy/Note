---
title: 使用socket实现http服务器和发送http请求
date: 2020-07-08 10:14:35
tags:
categories:
- Java基础
---

## 实现http服务器

```java
// 服务端
public class TestHttpServer {
    public static void main(String[] args) throws Exception{
        ServerSocket server = new ServerSocket(8080);
        System.out.println("服务端开启，等待客户端连接...");
        while (true) {
            Socket socket = server.accept();
            System.out.println("客户端连接成功！");
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String line = br.readLine();
            while (line != null && !"".equals(line)) {
                System.out.println(line);
                line = br.readLine();
            }
            String body = "Not Found";
            StringBuilder response = new StringBuilder();
            response.append("HTTP/1.1 404 Not Found\r\n") // 状态行
                    .append("Content-Length: ").append(body.getBytes().length).append("\r\n") // 消息报头
                    .append("Content-Type: text/plain; charset-utf-8\r\n") // 消息报头
                    .append("\r\n") // 空行
                    .append(body).append("\r\n"); // 响应正文
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            bw.write(response.toString());
            bw.flush();
            // 注意这里并没有将socket关闭
        }
    }
}
```

## 发送http请求

```java
// 客户端
public class TestHttpClient {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("localhost", 8080);
        OutputStream out = socket.getOutputStream();
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(out));
        bw.write("GET / HTTP/1.1\r\n"); // 请求行
        bw.write("Host: 127.0.0.1\r\n"); // 请求头
        bw.write("\r\n"); // 空行
        bw.flush();
        BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        String line = null;
        while ((line = br.readLine()) != null) { // 由于服务器端没有关闭socket，会一直阻塞在这里
            System.out.println(line);
        }
        br.close();
        bw.close();
        socket.close();
        System.out.println("close"); // 发现代码没有走到这里
    }
}
```

## 测试运行

**运行服务端**

```
// Server
服务端开启，等待客户端连接...
```

**运行客户端**

```
// Server
服务端开启，等待客户端连接...
客户端连接成功！
GET / HTTP/1.1
Host: 127.0.0.1
```

```
// Client
HTTP/1.1 404 Not Found
Content-Length: 9
Content-Type: text/plain; charset-utf-8

Not Found
```

**通过浏览器访问服务端**

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/实现http服务器.png)

## 参考

+ [使用socket实现http服务器和发送http请求](https://www.jianshu.com/p/fbdefa955e87)
+ [Java - Servlet究竟是什么](https://zhuanlan.zhihu.com/p/34518314)