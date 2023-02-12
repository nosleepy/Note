---
title: JavaWeb面试题
date: 2020-07-09 15:20:18
tags:
categories:
- Java面试
---

**Servlet是什么**

Servlet就是一个Java接口。([Java - Servlet究竟是什么](https://zhuanlan.zhihu.com/p/34518314))

**Maven是什么**

项目构建，管理，jar包下载。([maven是干嘛的？](https://www.zhihu.com/question/20104186/answer/73797359))

**cookie和session的区别**

+ cookie数据存放在客户端，session数据放在服务器上。
+ cookie不是很安全，别人可以分析存放在本地的Cookie并进行Cookie欺骗，考虑到安全应当使用session。
+ session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用Cookie。
+ 单个Cookie保存的数据长度不能超过4K，很多浏览器都限制一个网址最多保存20个cookie。
+ cookie只能存储String类型的对象，session能够存储任意Java对象。

**请求转发与请求重定向的区别**

+ 本质区别：转发是服务器行为，重定向是客户端行为。
+ 重定向特点：两次请求，浏览器地址发生变化，可以访问自己web之外的资源，传输数据会丢失。 
+ 转发特点：一次请求，浏览器地址不变，访问的是自己本身的web资源，传输的数据不会丢失。

**GET和POST的区别**

1. GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，POST提交的数据放在主体中。
2. GET提交的数据大小有限制，POST提交的数据大小没有限制。
3. GET提交数据会带来安全问题，POST提交数据不会带来安全问题。

**说下原生jdbc操作数据库流程**

1. 注册驱动程序：Class.forName("com.mysql.jdbc.Driver")。
2. 使用驱动管理类来获取数据连接对象：conn = DriverManager.getConnection(…)。
3. 获取数据库操作对象：Statement stmt = conn.createStatement()。
4. 定义操作的SQL语句。
5. 执行SQL：stmt.executeQuery(sql)。
6. 处理结果集：ResultSet，如果SQL前有参数值就设置参数值setXXX()。
7. 关闭对象，回收数据库资源（关闭结果集–>关闭数据库操作对象–>关闭连接）。

**为什么要使用PreparedStatement而不是Statement**

+ 代码的可读性和可维护性。Statement需要不断地拼接，而PreparedStatement不会。
+ PreparedStatement尽最大可能提高性能。DB有缓存机制，相同的预编译语句再次被调用不会再次需要编译。
+ 极大地提高了安全性。Statement容易被SQL注入，而PreparedStatement传入的内容不会和sql语句发生任何匹配关系。

**数据库连接池的机制是什么**

数据库连接池在初始化的时候创建一定数量的数据库连接放到连接池中，这些数量由最小连接数决定，当需要建立数据库连接时，从连接池中取出一个，使用完之后再放回去。通过最大连接数来限制数据库连接。当请求的连接数超过最大连接数时，这些请求将被加入到等待队列中。

**http的长连接和短连接**

+ 短连接：客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。
+ 长连接：当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。

**http/1.1与http/1.0的区别**

**http常见的状态码有哪些**

```
200 OK                     //客户端请求成功
301                        //永久重定向，比如使用域名跳转
302                        //临时重定向，比如未登陆的用户访问用户中心重定向到登录页面
400 Bad Request            //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized           //请求未经授权，这个状态码必须和WWW-Authenticate报头域一起使用
403 Forbidden              //服务器收到请求，但是拒绝提供服务
404 Not Found              //请求资源不存在，eg：输入了错误的URL
500 Internal Server Error  //服务器发生不可预期的错误
503 Server Unavailable     //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```

**在单点登陆中，如果cookie被禁用了怎么办**

单点登录的原理是后端生成一个sessionID，然后设置到cookie，后面的所有请求浏览器都会带上cookie，然后服务端从cookie里获取sessionID，再查看到用户信息。所以，保持登录的关键不是cookie，而是通过cookie保存和传输的sessionID，其本质是能获取用户信息的数据。除了cookie，还通常使用HTTP请求头来传输。但是这个请求头浏览器不会像cookie一样自动携带，需要手工处理。

**什么是jsp**

jsp本质上就是一个Servlet，它是Servlet的一种特殊形式，每个jsp页面都是一个Servlet实例。

**什么是Servlet**

Servlet是由Java提供的用于开发web服务器应用程序的一个组件，运行在服务端，由Servlet容器管理，用于生成动态内容。

**jsp和Servlet的区别**

+ jsp是html页面中内嵌的Java代码，侧重页面显示。
+ Servlet是html代码和Java代码分离，侧重逻辑控制。
+ mvc设计思想中jsp位于视图层，servlet位于控制层。

**jsp四大域对象**

page context，request，session，application。

**jsp九大内置对象**

request，response，session，out，page context，page，exception，application，config。

**什么是xml**

xml是一种可扩展性标记语言，支持自定义标签使用DTD和XML Schema标准化XML结构。

**xml优点**

用于配置文件，格式统一，符合标准，用于在互不兼容的系统间交互数据，共享数据方便。

**xml缺点**

xml文件格式复杂，数据传输占流量，服务端和客户端解析xml文件占用大量资源且不易维护。

**Linux中如何查看日志**

动态打印日志信息：tail -f 日志文件。

**Linux怎么关闭进程**

通常用ps查看进程PID，用kill命令终止进程。

**分布式session怎么实现的**

+ Tomcat搭建集群做Session全局复制（集群内每个tomcat的session完全同步），会影响集群的性能呢，不建议。
+ 根据请求的IP进行Hash映射到对应的机器上（这就相当于请求的IP一直会访问同一个服务器），如果服务器宕机了，会丢失了一大部分Session的数据，不建议。
+ 把Session数据放在Redis中（使用Redis模拟Session），建议。

**正向代理和反向代理**

+ 正向代理：客户端不能直接访问目标服务器，需要通过代理服务器间接访问目标服务器。
+ 反向代理：客户端知道代理服务器的地址，代理服务器具体访问哪一台目标服务器不知道。

**Session的实现机制**

浏览器输入用户名和密码提交到服务器端，服务器端验证通过后创建一个session对象，将用户信息存放在session中，并生成一个随机数id，保存在一个叫做JSESSIONID的cookie中，发送给浏览器，浏览器在下次访问服务器的时候就会在请求头携带这个cookie，服务器端会检查有没有这个SESSIONID对应的session对象，并取出session对象中的信息判断用户是否登录。

**Cookie和Session的区别**

+ 都是保存用户信息，不同在于Session存储在服务器端，Cookie是存储在客户端。
+ Session中可以保存任意对象，Cookie只能保存字符串。
+ Session随会话结束而关闭，Cookie可以长期保存在客户端硬盘上，也可以临时保存在浏览器内存中。
+ Session用来保存重要信息，Cookie用来保存不重要的用户信息。
