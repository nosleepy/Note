---
title: Java深拷贝和浅拷贝
date: 2020-05-14 15:01:25
tags:
categories:
- Java基础
---

## 概念

**浅拷贝**

拷贝出当前对象的一个副本，这个新对象和当前对象处于不同的堆内存中，两个对象的基本数据类型的值完全一样，但是引用数据类型还是指向同一个对象的。

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/浅拷贝.png" width="400px"/>

**深拷贝**

深拷贝比浅拷贝更进一步，拷贝出当前对象的一个副本，这个新对象和当前对象处于不同的堆内存中，两个对象的基本数据类型的值完全一样，引用数据类型指向的对象也拷贝出了一份一模一样的副本。

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/深拷贝.png" width="400px"/>

**深拷贝和浅拷贝对比**

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/深拷贝和浅拷贝对比.jpg" width="480px"/>

## 如何使用

1、需要拷贝的对象必须实现 `Cloneable` 接口

2、重写*java.lang.Object*类中的 `clone()` 方法

3、在 `clone()` 方法中调用 `super.clone()`

## 浅拷贝示例

```java
//地址类

public class Address {
    private String country;
    private String province;

    public Address() {
    }

    public Address(String country, String province) {
        this.country = country;
        this.province = province;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    @Override
    public String toString() {
        return "Address{" +
                "country='" + country + '\'' +
                ", province='" + province + '\'' +
                '}';
    }
}
```

```java
//学生类

public class Student implements Cloneable {
    private int age;
    private String name;
    private Address address;

    public Student() {
    }

    public Student(int age, String name, Address address) {
        this.age = age;
        this.name = name;
        this.address = address;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", address=" + address +
                '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

```java
//测试类

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Student s1 = new Student(20, "a", new Address("中国", "张家界"));
        Student s2 = (Student) s1.clone();
        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s1 == s2);
        System.out.println(s1.getAddress() == s2.getAddress());
    }
}
```

```
//运行结果

Student{age=20, name='a', address=Address{country='中国', province='张家界'}}
Student{age=20, name='a', address=Address{country='中国', province='张家界'}}
false
true
```

## 深拷贝示例

```java
//地址类

public class Address implements Cloneable {
    private String country;
    private String province;

    public Address() {
    }

    public Address(String country, String province) {
        this.country = country;
        this.province = province;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    @Override
    public String toString() {
        return "Address{" +
                "country='" + country + '\'' +
                ", province='" + province + '\'' +
                '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

```java
//学生类

public class Student implements Cloneable {
    private int age;
    private String name;
    private Address address;

    public Student() {
    }

    public Student(int age, String name, Address address) {
        this.age = age;
        this.name = name;
        this.address = address;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", address=" + address +
                '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Student student = (Student) super.clone();
        student.address = (Address) address.clone();
        return student;
    }
}
```

```java
//测试类

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Student s1 = new Student(20, "a", new Address("中国", "张家界"));
        Student s2 = (Student) s1.clone();
        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s1 == s2);
        System.out.println(s1.getAddress() == s2.getAddress());
    }
}
```

```
//运行结果

Student{age=20, name='a', address=Address{country='中国', province='张家界'}}
Student{age=20, name='a', address=Address{country='中国', province='张家界'}}
false
false
```

## 参考

+ [Java深拷贝与浅拷贝](https://zhuanlan.zhihu.com/p/96408010)
+ [JAVA深复制(深克隆)与浅复制(浅克隆)](https://www.cnblogs.com/qlwang/p/7889802.html)
+ [细说 Java 的深拷贝和浅拷贝](https://www.jianshu.com/p/1e1c09bd0fa8)






