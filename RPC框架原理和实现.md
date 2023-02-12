---
title: RPC框架原理和实现
date: 2020-05-22 22:43:58
tags:
categories:
- 框架
---

## 实现原理

**RPC（Remote Procedure Call）远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议**。

不管是什么样的RPC框架，总体思路都是服务提供方暴露服务，消费方通过服务方提供的接口使用动态代理获取代理对象，然后调用代理对象的方法，代理对象在内部进行远程调用，获得计算结果。简要示意图如下图所示：

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/RPC示意图.png" width="560px"/>

在这个过程中，有两处关键点。第一是获取代理对象，第二是代理对象与服务提供方建立连接。对于获取代理对象的方式，需要了解 Java 的动态代理，可参考 Java 的三种代理模式。总的来说，Java 的动态代理（此处只指 JDK 中生成代理对象的 API，不包括 cglib ）把建立远程连接的细节封装起来，使服务消费方可以在仅已知服务提供方的接口的情况下，可以像调用本地对象的方法一样去调用远程服务。

## 简单实例

### 通用类

**传输实体类**

```java
public class NetModel implements Serializable {
    private static final long serialVersionUID = -1822672222620235061L; // 序列化UID
    private String className; // 类名
    private String methodName; // 方法名
    private Object[] args; // 参数

    public NetModel() {
    }

    public NetModel(String className, String methodName, Object[] args) {
        this.className = className;
        this.methodName = methodName;
        this.args = args;
    }

    public static long getSerialVersionUID() {
        return serialVersionUID;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public String getMethodName() {
        return methodName;
    }

    public void setMethodName(String methodName) {
        this.methodName = methodName;
    }

    public Object[] getArgs() {
        return args;
    }

    public void setArgs(Object[] args) {
        this.args = args;
    }

    @Override
    public String toString() {
        return "NetModel{" +
                "className='" + className + '\'' +
                ", methodName='" + methodName + '\'' +
                ", args=" + Arrays.toString(args) +
                '}';
    }
}
```

**User类**

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1101835956823929089L;
    private Integer id;
    private String name;
    private String pass;

    public User() {
    }

    public User(Integer id, String name, String pass) {
        this.id = id;
        this.name = name;
        this.pass = pass;
    }

    public static long getSerialVersionUID() {
        return serialVersionUID;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPass() {
        return pass;
    }

    public void setPass(String pass) {
        this.pass = pass;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pass='" + pass + '\'' +
                '}';
    }
}
```

**UserService接口**

```java
public interface UserService {
    User findUserById(Integer id);

    String addUser(User user);
}
```

### 服务提供方

**UserServiceImpl实现类**

```java
public class UserServiceImpl implements UserService {
    public User findUserById(Integer id) {
        return new User(1, "lx", "123456");
    }

    public String addUser(User user) {
        return "success";
    }
}
```

**RPCServer服务端**

```java
public class RPCServer {
    public static void openServer() throws Exception {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("提供方服务开启，等待消费方调用...");
        while (true) {
            Socket socket = serverSocket.accept();
            System.out.println("消费方开始调用服务：" + socket.getLocalAddress());
            ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
            NetModel netModel = (NetModel) in.readObject();
            String className = netModel.getClassName();
            String methodName = netModel.getMethodName();
            Object[] args = netModel.getArgs();
            Class<?>[] types = new Class[args.length];
            for (int i = 0; i < types.length; i++) {
                types[i] = args[i].getClass();
            }
            Class<?> clazz = Class.forName(className + "Impl");
            Method method = clazz.getMethod(methodName, types);;
            Object object = method.invoke(clazz.newInstance(), args);
            ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
            out.writeObject(object);
            out.flush();
            out.close();
            in.close();
            socket.close();
        }
    }

    public static void main(String[] args) {
        try {
            openServer();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 服务消费方

**ProxyFactory代理类工厂**

```java
public class ProxyFactory {
    private static InvocationHandler handler = new InvocationHandler() {
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            NetModel netModel = new NetModel(proxy.getClass().getInterfaces()[0].getName(), method.getName(), args);
            return RPCClient.send(netModel);
        }
    };

    public static <T> T getInstance(Class<T> clazz) {
        return (T) Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{clazz}, handler);
    }
}
```

**RPCClient客户端**

```java
public class RPCClient {
    public static Object send(NetModel netModel) throws Exception {
        Socket socket = new Socket("127.0.0.1", 8080);
        ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
        out.writeObject(netModel);
        out.flush();
        ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
        Object object = in.readObject();
        out.close();
        in.close();
        socket.close();
        return object;
    }

    public static void main(String[] args) {
        UserService userService = ProxyFactory.getInstance(UserService.class);
        User user = userService.findUserById(1);
        System.out.println("查找结果：" + user);
        String result = userService.addUser(new User(2, "test", "123456"));
        System.out.println("添加结果：" + result);
    }
}
```

### 运行结果

**提供方开启服务**

```
提供方服务开启，等待消费方调用...
```

**消费方调用服务**

```
查找结果：User{id=1, name='lx', pass='123456'}
添加结果：success
```
**提供方显示消费方调用信息**

```
提供方服务开启，等待消费方调用...
消费方开始调用服务：/127.0.0.1
消费方开始调用服务：/127.0.0.1
```

## 参考

+ [RPC原理简析——三分钟看完](https://www.jianshu.com/p/d3c7a5bbca09)



