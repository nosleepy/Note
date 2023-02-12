---
title: Java数组复制的三种方法
date: 2020-05-14 16:13:18
tags:
categories:
- Java基础
---

## 将数组元素逐个复制到目标数组中

**使用for循环**

```java
public class ArrayCopy01 {
    public static void main(String[] args) {
        //源数组
        int[] source = {10, 20, 30, 40};
        //目标数组
        int[] target = new int[source.length];
        for (int i = 0; i < source.length; i++) {
            target[i] = source[i];
        }
        System.out.println(Arrays.toString(source));
        System.out.println(Arrays.toString(target));
    }
}
```

**复制结果**

```
[10, 20, 30, 40]
[10, 20, 30, 40]
```

## 使用System类的arraycopy()方法

**arraycopy()方法**

```java
public static native void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)

src #源数组
srcPos #源数组的起始下标
dest #目标数组
destPos #目标数组的起始下标
length #复制的数组元素的个数
```

```java
public class ArrayCopy02 {
    public static void main(String[] args) {
        int[] a = {10, 20, 30, 40};
        int[] b = new int[5];
        int[] c = new int[3];
        try {
            System.arraycopy(a, 0, b, 0, 4);
            System.arraycopy(a, 0, c, 0, 4); //目标数组不足以容纳源数组元素，会抛出异常
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Arrays.toString(a));
        System.out.println(Arrays.toString(b));
        System.out.println(Arrays.toString(c));
    }
}
```

**复制结果**

```
java.lang.ArrayIndexOutOfBoundsException
	at java.lang.System.arraycopy(Native Method)
	at com.zhx2020.javabase.copy.arraycopy.ArrayCopy02.main(ArrayCopy02.java:16)
[10, 20, 30, 40]
[10, 20, 30, 40, 0]
[0, 0, 0]
```

## 使用Arrays类的copyOf()方法和copyOfRange()方法

**copyOf()方法**

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength)); //调用System.arraycopy()方法
    return copy;
}

original #源数组
newLength # 新数组的长度

如果 `newLength` 小于源数组的长度，则将源数组的前面若干个元素复制到目标数组。
如果 `newLength` 大于源数组的长度，则将源数组的所有元素复制到目标数组。
```

**copyOfRange()方法**

```java
public static int[] copyOfRange(int[] original, int from, int to) {
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    int[] copy = new int[newLength];
    System.arraycopy(original, from, copy, 0,
                     Math.min(original.length - from, newLength));
    return copy;
}

original #源数组
fron #起始下标
to #结束下标，不包括
```

```java
public class ArrayCopy03 {
    public static void main(String[] args) {
        int[] a = {10, 20, 30, 40};
        int[] b = Arrays.copyOf(a, 3);
        int[] c = Arrays.copyOfRange(a, 1, 4);
        System.out.println(Arrays.toString(a));
        System.out.println(Arrays.toString(b));
        System.out.println(Arrays.toString(c));
    }
}
```

**复制结果**

```
[10, 20, 30, 40]
[10, 20, 30]
[20, 30, 40]
```

## 参考

+ [Java 数组元素复制的三种方法](https://www.cnblogs.com/my-program-life/p/11020422.html)






