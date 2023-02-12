---
title: Java大数运算
date: 2020-04-29 11:31:29
tags:
- 大数
categories:
- Java基础
---

## 概述

在 Java 中提供了大数字的操作类，即 `java.math.BigInteger` 类与 `java.math.BigDecimal` 类。这两个类用于高精度计算，`BigInteger` 类是针对大整数的处理类，而 `BigDecimal` 类则是针对大浮点数的处理类。

## BigInteger

构造方法

```java
public static BigInteger valueOf(long val) {
	...
}
//val是一个long类型的值
```

```java
public BigInteger(String val, int radix) {
	...
}
//将十进制字符串转换为指定进制的大数值
```

```java
public BigInteger(String val) {
    this(val, 10);
}
//其中，val是十进制字符串
```

常用方法

1. `public BigInteger add(BigInteger val)`：做加法运算
2. `public BigInteger subtract(BigInteger val) `：做减法运算
3. `public BigInteger multiply(BigInteger val)` ： 做乘法运算
4. `public BigInteger divide(BigInteger val)` ： 做除法运算
5. `public BigInteger mod(BigInteger m)`：做求余运算
6. `public boolean equals(Object x)` ：做数字比较操作（true / false）
7. `public int compareTo(BigInteger val) `:做数字比较操作（1 / -1 / 0）
8. `public BigInteger min(BigInteger val) `： 返回较小的数值
9. `public BigInteger max(BigInteger val) `： 返回较大的数值

示例

```java
public class BigIntegerTest {

    public static void main(String[] args) {
        BigInteger bigInteger = new BigInteger("10");
        System.out.println("加法操作：" + bigInteger.add(new BigInteger("3")));
        System.out.println("减法操作：" + bigInteger.subtract(new BigInteger("3")));
        System.out.println("乘法操作：" + bigInteger.multiply(new BigInteger("3")));
        System.out.println("除法操作：" + bigInteger.divide(new BigInteger("3")));
        System.out.println("取余操作：" + bigInteger.mod(new BigInteger("3")));
    }

}
```

运行结果

```
加法操作：13
减法操作：7
乘法操作：30
除法操作：3
取余操作：1
```

## BigDecimal

构造方法

```java
public BigDecimal(double val) {
	...
}
//val是一个双精度型的变量
```

```java
public BigDecimal(String val) {
	...
}
//val是一个字符串形式的浮点类型值
```

常用方法

1. `public BigDecimal add(BigDecimal augend)` ：做加法操作
2. `public BigDecimal subtract(BigDecimal subtrahend)` ：做减法操作
3. `public BigDecimal multiply(BigDecimal multiplicand)` ：做乘法操作
4. `public BigDecimal divide(BigDecimal divisor, int sacle, int roundingMode)` ：做除法操作，方法中 3 个参数分别代表除数、商的小数点后的位数、近似处理模式

示例

```java
public class BigDecimalTest {

    public static void main(String[] args) {
        BigDecimal bigDecimal = new BigDecimal(1.2);
        System.out.println("加法操作：" + bigDecimal.add(new BigDecimal(0.2)));
        System.out.println("减法操作：" + bigDecimal.subtract(new BigDecimal(0.2)));
        System.out.println("乘法操作：" + bigDecimal.multiply(new BigDecimal(0.2)));
        System.out.println("除法操作：" + bigDecimal.divide(new BigDecimal(0.33), 10, BigDecimal.ROUND_HALF_UP));
    }

}
```
运行结果

```
加法操作：1.399999999999999966693309261245303787291049957275390625
减法操作：0.999999999999999944488848768742172978818416595458984375
乘法操作：0.2400000000000000044408920985006256686564609092309028676696466982586064542459780568606220185756683349609375
除法操作：3.6363636364
```

## 参考

+ [初识Java（Java数字处理类-大数字运算)](https://zhuanlan.zhihu.com/p/63853401)
+ [java大数详解](https://blog.csdn.net/dongchengrong/article/details/78848399)
+ [BigDecimal舍入模式](https://www.cnblogs.com/geekfx/p/12423061.html)