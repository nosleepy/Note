---
title: Java反射详解
date: 2020-05-16 16:03:59
tags:
categories:
- Java基础
---

## 反射介绍

Java反射机制是在运行状态中，对于任何一个类，都能够知道这个类的所有属性和方法；对于任何一个对象，都能够调用它的任意方法和属性；这种动态获取类的信息以及动态调用对象属性和方法的功能称为Java语言的反射机制。

想要使用反射机制，就必须要先获取到该类的字节码文件对象（`.class`），通过字节码文件对象，就能够通过该类中的方法获取到我们想要的所有信息(方法，属性，类名，父类名，实现的所有接口等等)，每一个类对应着一个字节码文件也就对应着一个 `Class` 类型的对象，也就是字节码文件对象。

## User类

```java
public class User {
    public int id; //公有成员变量
    private String name;
    private String pass;

    public User() {
    }

    public User(String name, String pass) {
        this.name = name;
        this.pass = pass;
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

    private int getId() { //私有成员方法
        return id;
    } 

    private void setId(int id) { //私有成员方法
        this.id = id;
    }

    public void test(String info) { //测试方法
        System.out.println("info = " + info);
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", pass='" + pass + '\'' +
                '}';
    }
}
```

## 获取Class对象

1、根据类名

```java
Class clazz = User.class;
```

2、根据对象

```java
Class clazz = new User().getClass();
```

3、根据全限定类名

```java
Class clazz = Class.forName("com.zhx2020.javabase.reflect.User");
```

## 创建实例

1、无参构造器创建对象

```java
Class clazz = User.class;
Constructor constructor = clazz.getConstructor();
User user = (User) constructor.newInstance();
```

2、有参构造器创建对象

```java
Class clazz = User.class;
Constructor constructor = clazz.getConstructor(String.class, String.class);
User user = (User) constructor.newInstance("lh", "123456");
```

3、Class对象创建对象

```java
Class clazz = User.class;
User user = (User) clazz.newInstance();
```

## 获取成员变量

1、`getFields()`：获取公有成员变量（public），包括父类

```java
Class clazz = User.class;
Field[] fields = clazz.getFields();
for (Field field : fields) {
    System.out.println(field);
}

//运行结果
public int com.zhx2020.javabase.reflect.User.id
```

2、`getDeclaredFields()`：获取所有成员变量（private、public、protected、default），不包括父类

```java
Class clazz = User.class;
Field[] fields = clazz.getDeclaredFields();
for (Field field : fields) {
    System.out.println(field);
}

//运行结果
public int com.zhx2020.javabase.reflect.User.id
private java.lang.String com.zhx2020.javabase.reflect.User.name
private java.lang.String com.zhx2020.javabase.reflect.User.pass
```

## 获取方法

1、`getMethods()`：获取共有方法（public），包括父类

```java
Class clazz = User.class;
Method[] methods = clazz.getMethods();
for (Method method : methods) {
    System.out.println(method);
}

//运行结果
public java.lang.String com.zhx2020.javabase.reflect.User.toString()
public java.lang.String com.zhx2020.javabase.reflect.User.getName()
public void com.zhx2020.javabase.reflect.User.setName(java.lang.String)
public java.lang.String com.zhx2020.javabase.reflect.User.getPass()
public void com.zhx2020.javabase.reflect.User.setPass(java.lang.String)
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
```

2、`getDeclaredMethods()`：获取所有方法（private、public、protected、default），不包括父类

```java
Class clazz = User.class;
Method[] methods = clazz.getDeclaredMethods();
for (Method method : methods) {
    System.out.println(method);
}

//运行结果
public java.lang.String com.zhx2020.javabase.reflect.User.toString()
public java.lang.String com.zhx2020.javabase.reflect.User.getName()
private int com.zhx2020.javabase.reflect.User.getId()
public void com.zhx2020.javabase.reflect.User.setName(java.lang.String)
public java.lang.String com.zhx2020.javabase.reflect.User.getPass()
private void com.zhx2020.javabase.reflect.User.setId(int)
public void com.zhx2020.javabase.reflect.User.setPass(java.lang.String)
```

## 调用成员变量

1、`setAccessible(boolean flag)` // true表示可以访问私有变量

2、`get(Object obj)` //获取值

3、`set(Object obj, Object value)` //设置值

示例代码

```java
Class clazz = User.class;
Constructor constructor = clazz.getConstructor(String.class, String.class);
User user = (User) constructor.newInstance("lh", "123456");
Field name = clazz.getDeclaredField("name");
name.setAccessible(true); //访问私有变量
System.out.println("before name = " + name.get(user)); //获取私有变量的值
name.set(user, "xm"); //设置私有变量的值
System.out.println("after name = " + name.get(user)); //获取私有变量的值
```

运行结果

```
before name = lh
after name = xm
```

## 调用方法

通过 `invoke(Object obj, Object... args)` 调用方法

示例代码

```java
Class clazz = User.class;
Constructor constructor = clazz.getConstructor(String.class, String.class);
User user = (User) constructor.newInstance("lh", "123456");
Method test = clazz.getDeclaredMethod("test", String.class);
System.out.println(test);
test.invoke(user, "student");
```

运行结果

```
public void com.zhx2020.javabase.reflect.User.test(java.lang.String)
info = student
```

## 参考

+ [Java反射使用](https://blog.csdn.net/luck_zz/article/details/80027438)
+ [java反射机制详解](https://www.cnblogs.com/myRichard/p/11742194.html)
+ [Class类中getDeclaredFields() 与getFields()的区别](https://blog.csdn.net/shenhaiwen/article/details/75305176)



