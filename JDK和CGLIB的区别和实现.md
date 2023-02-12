---
title: JDK和CGLIB的区别和实现
date: 2020-05-15 22:37:16
tags:
categories:
- 框架
---

## 原理区别

**JDK动态代理**是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用*InvokeHandler*来处理。

**CGLIB动态代理**是利用 `asm` 开源包，把代理对象类的 `class` 文件加载进来，通过修改其字节码生成子类来处理。

**JDK动态代理和CGLIB动态代理注意点**

1、如果目标对象实现了接口，默认情况下会采用 `JDK` 的动态代理实现 `AOP`

2、如果目标对象实现了接口，可以强制使用 `CGLIB` 实现 `AOP`

3、如果目标对象没有实现了接口，必须采用 `CGLIB` 库，`spring` 会自动在 `JDK` 动态代理和 `CGLIB` 之间转换

**JDK动态代理和CGLIB字节码生成的区别？**

1、`JDK` 动态代理只能对实现了接口的类生成代理，而不能针对类

2、`CGLIB` 是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以该类或方法最好不要声明成 `final`

**如何强制使用CGLIB实现AOP**

1、添加 `CGLIB` 库，`SPRING_HOME/cglib/*.jar`
2、在 `spring` 配置文件中加入 `<aop:aspectj-autoproxy proxy-target-class="true"/>`

## JDK动态代理实现

用户管理接口

```java
public interface UserManager {
    void addUser(String name, String pass);
    
    void delUser(String name);
}
```

用户管理接口实现类

```java
public class UserManagerImpl implements UserManager {
    @Override
    public void addUser(String name, String pass) {
        System.out.println("add an user!");
        System.out.println("name = " + name + ",pass = " + pass);
    }

    @Override
    public void delUser(String name) {
        System.out.println("del an user!");
        System.out.println("name = " + name);
    }
}
```

JDK动态代理

```java
//JDK动态代理实现InvocationHandler接口
public class JdkProxy implements InvocationHandler {
    private Object target; //需要代理的目标对象

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("jdk proxy start!");
        Object result = method.invoke(target, args);
        System.out.println("jdk proxy end!");
        return result;
    }

    //定义获取代理对象方法
    public Object getJDKProxy(Object targetObject) {
        //为目标对象target赋值
        this.target = targetObject;
        //JDK动态代理只能针对实现了接口的类进行代理
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(), targetObject.getClass().getInterfaces(), this);
    }

    public static void main(String[] args) {
        JdkProxy jdkProxy = new JdkProxy(); //实例化JDKProxy对象
        UserManager userManager = (UserManager) jdkProxy.getJDKProxy(new UserManagerImpl()); //获取代理对象
        userManager.addUser("lh", "123456"); //执行新增方法
    }
}
```

运行结果

```
jdk proxy start!
add an user!
name = lh, pass = 123456
jdk proxy end!
```

## CGLIB动态代理实现

用户管理类

```java
public class UserManager {
    public void addUser(String name, String pass) {
        System.out.println("add an user!");
        System.out.println("name = " + name + ", pass = " + pass);
    }

    public void delUser(String name) {
        System.out.println("del an user!");
        System.out.println("name = " + name);
    }
}
```

Cglib动态代理

```java
//Cglib动态代理，实现MethodInterceptor接口
public class CglibProxy implements MethodInterceptor {
    private Object target; //需要代理的目标对象

    //重写拦截方法
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib proxy start!");
        Object invoke = method.invoke(target, objects); //方法执行，参数：target 目标对象 arr参数数组
        System.out.println("cglib proxy end!");
        return invoke;
    }

    //定义获取代理对象方法
    public Object getCglibProxy(Object objectTarget) {
        //为目标对象target赋值
        this.target = objectTarget;
        Enhancer enhancer = new Enhancer();
        //设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
        enhancer.setSuperclass(objectTarget.getClass());
        enhancer.setCallback(this); //设置回调
        Object result = enhancer.create(); //创建并返回代理对象
        return result;
    }

    public static void main(String[] args) {
        CglibProxy cglibProxy = new CglibProxy(); //实例化CglibProxy对象
        UserManager userManager = (UserManager) cglibProxy.getCglibProxy(new UserManager()); //获取代理对象
        userManager.delUser("lh");
    }
}
```

Cglib动态代理运行结果

```
cglib proxy start!
del an user!
name = lh
cglib proxy end!
```

## 参考

+ [Spring的两种动态代理：Jdk和Cglib 的区别和实现](https://www.cnblogs.com/leifei/p/8263448.html)


