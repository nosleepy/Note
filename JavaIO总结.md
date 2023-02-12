---
title: JavaIO总结
date: 2020-08-04 19:30:11
tags:
categories:
- Java基础
---

## BIO

### 字节流

**FileOutputStream**

```java
// 字节输出流
public class Main {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        // 写一个字节
        fos.write(100); // d

        // 写字节数组
        byte[] bytes = {65, 66, 67, 68}; // ABCD
        fos.write(bytes);

        // 写字节数组的一部分
        fos.write(bytes, 1, 1); // B

        // 写字节数组的简便方式
        fos.write("hello".getBytes()); // hello

        //关闭资源
        fos.close();
    }
}
```

**FileInputStream**

```java
// 字节输入流
public class Main {
    public static void main(String[] args) throws Exception {
        FileInputStream fis = new FileInputStream("file.txt");
        // 读取一个字节
        /*int len = 0;
        while ((len = fis.read()) != -1) {
            System.out.print((char)len);
        }*/

        // 读取到字节数组
        byte[] bytes = new byte[1024];
        int len = 0;
        while ((len = fis.read(bytes)) != -1) {
            System.out.print(new String(bytes, 0, len));
        }

        // 关闭资源
        fis.close();
    }
}
```

### 字符流

**FileWriter**

```java
// 字符输出流
public class Main {
    public static void main(String[] args) throws Exception {
        FileWriter fw = new FileWriter("file.txt");
        // 写一个字符
        fw.write(100); // d
        fw.flush();

        // 写一个字符数组
        char[] c = {'a', 'b', 'c', 'd'};
        fw.write(c);
        fw.flush();

        // 写字符数组一部分
        fw.write(c, 2, 2); // cd
        fw.flush();

        // 写入字符串
        fw.write("hello");
        fw.flush();

        // 关闭资源
        fw.close();
    }
}
```

**FileReader**

```java
// 字符输入流
public class Main {
    public static void main(String[] args) throws Exception {
        FileReader fr = new FileReader("file.txt");

        // 读取单个字符
        /*int len = 0;
        while ((len = fr.read()) != -1) {
            System.out.print((char)len);
        }*/

        // 读取到字符数组
        char[] c = new char[1024];
        int len = 0;
        while ((len = fr.read(c)) != -1) {
            System.out.print(new String(c, 0, len));
        }

        // 关闭资源
        fr.close();
    }
}
```

### 转换流

**OutputStreamWriter**

```java
// 将字节输出流转换为字符输出流
public OutputStreamWriter(OutputStream out, Charset cs) {
    super(out);
    if (cs == null)
        throw new NullPointerException("charset");
    se = StreamEncoder.forOutputStreamWriter(out, this, cs);
}
```

**InputStreamReader**

```java
// 将字节输入流转换为字符输入流
public InputStreamReader(InputStream in) {
    super(in);
    try {
        sd = StreamDecoder.forInputStreamReader(in, this, (String)null);
    } catch (UnsupportedEncodingException e) {
        throw new Error(e);
    }
}
```

### 字节缓冲流

**BufferedOutputStream**

```java
// 字节输出缓冲流
public class Main {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        BufferedOutputStream bos = new BufferedOutputStream(fos);

        bos.write(55); // 7

        byte[] bytes = "hello".getBytes();
        bos.write(bytes); // hello
        bos.write(bytes, 3, 2); //lo

        bos.close();
        fos.close();
    }
}
```

**BufferedInputStream**

```java
// 字节输入缓冲流
public class Main {
    public static void main(String[] args) throws Exception {
        FileInputStream fis = new FileInputStream("file.txt");
        BufferedInputStream bis = new BufferedInputStream(fis);
        
        byte[] bytes = new byte[1024];
        int len = 0;
        while ((len = bis.read(bytes)) != -1) {
            System.out.print(new String(bytes, 0, len));
        }
        
        bis.close();
        fis.close();
    }
}

```

### 字符缓冲流

**BufferedWriter**

```java
// 字符输出缓冲流

public class Main {
    public static void main(String[] args) throws Exception {
        FileWriter fw = new FileWriter("file.txt");
        BufferedWriter bw = new BufferedWriter(fw);

        bw.write(100); // d
        bw.flush();

        bw.write("你好".toCharArray());
        bw.newLine(); // 写换行
        bw.flush();

        bw.write("大家好");
        bw.flush();

        bw.close();
        fw.close();
    }
}
```

**BufferedReader**

```java
// 字符输入缓冲流
public class Main {
    public static void main(String[] args) throws Exception {
        FileReader fr = new FileReader("file.txt");
        BufferedReader br = new BufferedReader(fr);

        /*char[] c = new char[1024];
        int len = 0;
        while ((len = br.read(c)) != -1) {
            System.out.print(new String(c, 0, len));
        }*/

        String line = null;
        while ((line = br.readLine()) != null) { // 读取一个文本行
            System.out.println(line);
        }

        br.close();
        fr.close();
    }
}
```

### 对象流

**User**

```java
// 用户类
public class User implements Serializable { // 实现序列化接口
    int id;
    String name;

    public User() {
    }

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

**ObjectOutputStream**

```java
// 对象输出流
public class Main {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        User user = new User(1, "echo");
        oos.writeObject(user);
        oos.close();
        fos.close();
    }
}
```

**ObjectInputStream**

```java
// 对象输入流
public class Main {
    public static void main(String[] args) throws Exception {
        FileInputStream fis = new FileInputStream("file.txt");
        ObjectInputStream ois = new ObjectInputStream(fis);
        User user = (User) ois.readObject();
        System.out.println(user);
        ois.close();
        fis.close();
    }
}
```

### 打印流

**PrintStream**

```java
// 字节打印流
public class Main {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        PrintStream ps = new PrintStream(fos);
        ps.print("a");
        ps.println("b");
        ps.print("c");
        ps.close();
        fos.close();
    }
}
```

**PrintWriter**

```java
// 字符打印流
public class Main {
    public static void main(String[] args) throws Exception {
        FileWriter fw = new FileWriter("file.txt");
        PrintWriter pw = new PrintWriter(fw);
        pw.print("a");
        pw.println("b");
        pw.print("c");
        pw.close();
        fw.close();
    }
}
```

### 标准输入输出流

```java
// System.in 默认指键盘输入
// System.out 向显示器输出
public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String s = null;
        while ((s = br.readLine()) != null) {
            if ("886".equals(s)) {
                break;
            }
            System.out.println(s);
        }
        br.close();
    }
}
```

## 网络编程

### UDP通信

**UDP发送端**

```java
public class UDPSend {
    public static void main(String[] args) throws Exception {
        // 创建数据包对象,封装要发送的数据,接收端IP,端口
        byte[] data = "你好UDP".getBytes();
        // 创建InetAddress对象,封装自己的IP地址
        InetAddress inet = InetAddress.getByName("127.0.0.1");
        DatagramPacket dp = new DatagramPacket(data, data.length, inet, 8080);
        // 创建DatagramSocket对象,数据包的发送和接收对象
        DatagramSocket ds = new DatagramSocket();
        // 调用ds对象的方法send,发送数据包
        ds.send(dp);
        // 关闭资源
        ds.close();
    }
}
```

**UDP接收端**

```java
public class UDPReceive {
    public static void main(String[] args) throws Exception {
        // 创建数据包传输对象DatagramSocket,绑定端口号
        DatagramSocket ds = new DatagramSocket(8080);
        // 创建字节数组
        byte[] data = new byte[1024];
        // 创建数据包对象,传递字节数组
        DatagramPacket dp = new DatagramPacket(data, data.length);
        // 调用ds对象的方法receive传递数据包
        ds.receive(dp);
        // 获取发送端的IP地址对象
        String ip = dp.getAddress().getHostAddress();
        //获取发送的端口号
        int port = dp.getPort();
        //获取接收到的字节个数
        int length = dp.getLength();
        System.out.println(ip + " " + port + ":" + new String(data, 0, length));
        ds.close();
    }
}
```

### TCP通信

**TCP服务器**

```java
public class TCPServer {
    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(8080);
        // 调用服务器套接字对象中的方法accept(),获取客户端套接字对象
        Socket socket = serverSocket.accept();
        // 通过客户端套接字对象Socket,获取字节输入流,读取客户端发送来的消息
        InputStream in = socket.getInputStream();
        byte[] data = new byte[1024];
        int len = in.read(data);
        System.out.println(new String(data, 0, len));
        // 服务器向客户端回数据,字节输出流,通过客户端套接字对象获取字节输出流
        OutputStream out = socket.getOutputStream();
        out.write("收到,谢谢!!!".getBytes());
        // 关闭资源
        out.close();
        in.close();
        socket.close();
        serverSocket.close();
    }
}
```

**TCP客户端**

```java
public class TCPClient {
    public static void main(String[] args) throws Exception {
        // 创建Socket对象,连接服务器
        Socket socket = new Socket("127.0.0.1", 8080);
        // 通过客户端的套接字对象Socket方法,获取字节输出流将数据写向服务器
        OutputStream out = socket.getOutputStream();
        out.write("服务器ok!".getBytes());
        //读取服务器发回的数据,使用socket套接字对象中字节输入流
        InputStream in = socket.getInputStream();
        byte[] data = new byte[1024];
        int len = in.read(data);
        System.out.println(new String(data, 0, len));
        // 关闭资源
        in.close();
        out.close();
        socket.close();
    }
}
```

## NIO

在单线程的情况下，BIO是无法处理并发的。

服务器端如果不活跃的线程比较多，应该考虑单线程。

**NIO**：new io，noblocking io。

### 三大组件

Buffer、Channel、Selector。

## Netty

### 传统IO与NIO比较

**传统IO特点**

阻塞点：server.accept()、in.read(bytes)
单线程情况下只能有一个客户端
用线程池可以有多个客户端连接，但是非常消耗性能

**NIO的特点**

ServerSocketChannel
SocketChannel
Selector
SelectionKey

### netty介绍

**netty版本**

+ netty3.x
+ netty4.x
+ netty5.x

**netty可以运用在哪些领域**

+ 分布式进程通信：例如hadoop、dubbo、akka等具有分布式功能的框架，底层RPC通信都是基于netty实现的，这些框架版本通常还在用netty3.x。
+ 游戏客户端开发：最新的游戏服务器有部分公司可能已经开始采用netty4.x或netty5.x。