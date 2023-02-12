---
title: Java常用方法
date: 2020-06-05 10:31:35
tags:
categories:
- Java基础
---

## String

字符串前面补0

```java
String str = String.format("%08d", 123);
System.out.println(str); // 00000123
```

字符串后面补0

```java
String str = String.format("%-8d", 123).replace(" ", "0");
System.out.println(str); // 12300000
```

保留指定位数小数

```java
int x = 1328;
float num = 13.283457f;
System.out.printf("%.4f", num); // 13.2835
System.out.printf("%d", x); // 1328
```

格式化输出

```java
int a = 1328;
float b = 13.283457f;
String str = String.format("%d %.4f", a, b);
System.out.println(str); // 1328 13.2835
```

字符串替换

```java
String str="hellollo";
System.out.println(str.replace('h', 'k')); // kellollo
System.out.println(str.replace("ll", "ww")); // hewwowwo
System.out.println(str.replaceFirst("ll", "ww")); // hewwollo
System.out.println(str.replaceAll("ll", "ww")); // hewwowwo
```

字符串截取

```java
String str ="Hello";
System.out.println(str.substring(1)); // ello
System.out.println(str.substring(2, 4)); // ll
System.out.println(str.subSequence(2, 4)); // ll
```

## Integer

构造方法

```java
public Integer(int value) {
    this.value = value;
}
```

```java
public Integer(String s) throws NumberFormatException {
    this.value = parseInt(s, 10);
}
```

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

字符串转整数

```java
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```

```java
public static int parseInt(String s, int radix) throws NumberFormatException {
   // 将给定进制的字符串转换为10进制整数 
}
```

整数转给定进制字符串

```java
public static String toString(int i, int radix) {
    // i表示要转换的整数，radix表示要转换的进制
}
```

## BigInteger

构造方法

```java
public BigInteger(String val) {
    this(val, 10);
}
```

```java
public BigInteger(String val, int radix) {
	// 将给定进制的字符串转换为10进制大数
}
```

```java
public static BigInteger valueOf(long val) {
    // 传入一个long类型的值
}
```

大数转换给定进制字符串

```java
public String toString(int radix) {
    // radix代表要转换的进制
}
```

## Arrays

数组排序

```java
Arrays.sort();
```

数组转成list

```java
String[] arr = {"a", "b", "c"};
List<String> list = Arrays.asList(arr);
```

## Collections

集合排序

```java
Collections.sort();
```

集合反转

```java
Collections.reverse();
```

## System

保留2位小数

```java
System.out.printf("%.2f", 78.1276); // 78.13
```

## List

list转成数组

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
String[] arr = new String[list.size()];
list.toArray(arr);
```