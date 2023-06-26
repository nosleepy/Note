---
title: Java泛型
date: 2021-06-26 00:00:00
tags:
categories:
- Java基础
---

#### 简介

Java 泛型（generics）是 JDK 5 中引入的一个新特性，其本质是参数化类型，解决不确定具体对象类型的问题。其所操作的数据类型被指定为一个参数（type parameter）这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

+ 泛型类

```java
public class Pair<K, V> {
    private K first;
    private V second;
    
    public Pair(K first, V second) {
        this.first = first;
        this.second = second;
    }
    
    public K getFirst() {
        return first;
    }
    
    public void setFirst(K first) {
        this.first = first;
    }
    
    public V getSecond() {
        return second;
    }
    
    public void setSecond(V second) {
        this.second = second;
    }
}
```

使用

```java
Pair<Integer, String> pair = new Pair<>(1024, "wlzhou");
System.out.println("first = " + pair.getFirst() + ", second = " + pair.getSecond());
// first = 1024, second = wlzhou
```

+ 泛型接口

```java
public interface Generator<T> {
    T next();
}
```

指定具体类型为Integer

```java
public class NumberGenerator implements Generator<Integer> {
    @Override
    public Integer next() {
        return 1024;
    }
}
```

指定具体类型为String

```java
public class StringGenerator implements Generator<String> {
    @Override
    public String next() {
        return "wlzhou";
    }
}
```

使用

```java
Generator<Integer> numberGenerator = new NumberGenerator();
Generator<String> stringGenerator = new StringGenerator();
System.out.println("number = " + numberGenerator.next() + ", string = " + stringGenerator.next());
// number = 1024, string = wlzhou
```

+ 泛型方法

```java
public <T> void func(T t) {
	System.out.println(t.getClass().getName());
}
```

使用

```java
public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        main.func("字符串");
        main.func(666);
    }
}
// java.lang.String
// java.lang.Integer
```

#### 泛型上下限

实体类

```java
class Fruit {}
class Apple extends Fruit{}
class Pear extends Fruit{}
class Orange extends Fruit{}
```

+ 上限

```java
public static void upperLimit(List<? extends Fruit> list) {
    list.add(new Apple()); // fail
    list.add(new Pear()); // fail
    list.add(new Orange()); // fail
    list.add(new Fruit()); // fail
    for (Fruit fruit : list) {
    	System.out.println(fruit);
    }
}
```

为什么不能添加

1. list是Fruit类型的,此时你去添加Fruit的子类都没问题
2. list是Apple类型的,此时你只能添加Apple,同理Pear,Orange也是

为什么可以取出

因为我们从list中拿出来的必定是Fruit类型的,毕竟Apple等都去继承Fruit了,可以自动向上转型

+ 下限

```java
public static void lowerLimit(List<? super Fruit> list) {
    list.add(new Apple());
    list.add(new Pear());
    list.add(new Orange());
    list.add(new Fruit());
    for (Object o : list) {
    	System.out.println(o);
    }
}
```

为什么可以添加

这里定义了下限是Fruit,也就是说这个list里面的类型都是Fruit的父类，所以我们只能添加Fruit和他的子类

为什么不可以取出

因为取的时候没法确实是Fruit的哪个父类,最后都只能获取我们的根类Object

#### 参考

+ [java泛型上下限](https://www.cnblogs.com/hujunzheng/p/5351697.html)
+ [泛型的上下限要怎么理解](https://blog.csdn.net/weixin_40251892/article/details/109063161)