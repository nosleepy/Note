---
title: Java类的生命周期
date: 2020-04-21 10:45:39
tags:
categories:
- Java虚拟机
---

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture@master/img/Java类的生命周期.png" width="680px"/>

## 加载

我们编写一个java的源文件，经过编译后生成一个后缀名为.class的文件，这就是字节码文件，java虚拟机就识别这种文件，java的生命周期就是class文件从加载到消亡的过程。

关于加载，其实，就是将源文件的class文件找到类的信息将其加载到方法区中，
然后在堆区中实例化一个java.lang.Class对象，作为方法区中这个类的信息的入口。

但是这一功能是在JVM之外实现的，主要的原因是方便让应用程序自己决定如何获取这个类，
在不同的虚拟机实现的方式不一定相同，hotspot虚拟机是采用需要时在加载的方式，
也有其他是先预先加载的。

## 连接

一般会跟加载阶段和初始化阶段交叉进行，过程由三部分组成：验证、准备和解析三步

1. 验证：确定该类是否符合java语言的规范，有没有属性和行为的重复，继承是否合理，总之，就是保证jvm能够执行
2. 准备：主要做的就是为由static修饰的成员变量分配内存，并设置默认的初始值，默认初始值如下：
   + 八种基本数据类型默认的初始值是0
   + 引用类型默认的初始值是null
   + 有static final修饰的会直接赋值，例如：static final int x=10；则默认就是10.
3. 解析：这一阶段的任务就是把常量池中的符号引用转换为直接引用，说白了就是jvm会将所有的类或接口名、字段名、方法名转换为具体的内存地址。

## 初始化

这个阶段就是将静态变量（类变量）赋值的过程，即只有static修饰的才能被初始化，执行的顺序就是：

父类静态域或着静态代码块，然后是子类静态域或者子类静态代码块

## 使用

在类的使用过程中依然存在三步：对象实例化、垃圾收集、对象终结

1. 对象实例化：就是执行类中构造函数的内容，如果该类存在父类JVM会通过显示或者隐示的方式先执行父类的构造函数，在堆内存中为父类的实例变量开辟空间，并赋予默认的初始值，然后在根据构造函数的代码内容将真正的值赋予实例变量本身，然后，引用变量获取对象的首地址，通过操作对象来调用实例变量和方法
2. 垃圾收集：当对象不再被引用的时候，就会被虚拟机标上特别的垃圾记号，在堆中等待GC回收
3. 对象的终结：对象被GC回收后，对象就不再存在，对象的生命也就走到了尽头

## 类卸载

即类的生命周期走到了最后一步，程序中不再有该类的引用，该类也就会被JVM执行垃圾回收，从此生命结束…

## 示例

+ **没有父类**

```java
public class Parent {

    static int a;

    int b;

    static {
        a = 1;
        System.out.println("执行父类静态代码块");
    }

    {
        b = 2;
        System.out.println("执行父类构造代码块");
    }

    Parent() {
        System.out.println("执行父类无参构造方法");
    }

    Parent(int b) {
        this.b = b;
        System.out.println("执行父类有参构造方法");
    }

    public static void main(String[] args) {
        Parent parent = new Parent(10);
        System.out.println("a = " + Parent.a);
        System.out.println("b = " + parent.b);
    }

}

```

结果

```
执行父类静态代码块
执行父类构造代码块
执行父类有参构造方法
a = 1
b = 10
```

+ 有父类

```java
public class Child extends Parent {

    static int a;

    int b;

    static {
        a = 1;
        System.out.println("执行子类静态代码块");
    }

    {
        b = 2;
        System.out.println("执行子类构造代码块");
    }

    Child() {
        System.out.println("执行子类无参构造方法");
    }

    Child(int b) {
        this.b = b;
        System.out.println("执行子类有参构造方法");
    }

    public static void main(String[] args) {
        Child child = new Child(10);
        System.out.println("a = " + Child.a);
        System.out.println("b = " + child.b);
    }

}
```

结果

```
执行父类静态代码块
执行子类静态代码块
执行父类构造代码块
执行父类无参构造方法
执行子类构造代码块
执行子类有参构造方法
a = 1
b = 10
```

## 总结

如果类还没有被加载：
1. 先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。
2. 执行子类的静态代码块和静态变量初始化。
3. 执行父类的实例变量初始化
4. 执行父类的构造函数
5. 执行子类的实例变量初始化
6. 执行子类的构造函数
7. 静态方法与非静态方法只有被调用的时候才会被加载

如果类已经被加载：
1. 则静态代码块和静态变量就不用重复执行，再创建类对象时，只执行与实例相关的变量初始化和构造方法。

## 参考

+ [java类中的对象加载顺序](https://zhuanlan.zhihu.com/p/33256690)
+ [java类的生命周期](https://www.cnblogs.com/ipetergo/p/6441310.html)