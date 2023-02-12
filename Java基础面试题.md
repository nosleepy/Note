---
title: Java基础面试题
date: 2020-07-06 21:59:59
tags:
categories:
- Java面试
---

**什么是红黑树**

+ 每个节点要么是红色，要么是黑色。
+ 根节点必须是黑色
+ 红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）。
+ 对于每个节点，从该点至 null（树尾端）的任何路径，都含有相同个数的黑色节点。

**Java反射机制**

在运行状态中，对于任何一个类，都能够知道这个类的所有属性和方法；对于任何一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象属性和方法的功能称为Java语言的反射机制。

**什么是B树**

B树是红黑树的一个变种，是一颗自平衡的多叉树。结点不再存储一个数据项，而是有多个数据项，数据项中包含key和value。

**什么是B+树**

B+树在B树之上又做了一层优化，非叶子节点也是存储多个数据项，但是数据项中只包含了key。

**Java为什么要有包装类**

1. 包装类里面有一些很有用的方法和属性，如HashCode，ParseInt。
2. 基本类型不能赋null值，某些场合需要。
3. 有些地方不能直接用基本类型，比如集合。

**什么时候用包装类，什么时候用基本类型？**

1. 在pojo类中定义的属性用包装类。
2. 在rpc方法中定义参数和返回值的类型用包装类。
3. 定义局部变量用基本类型。

**Java数组与集合区别**

+ 数组：大小固定，只能存储相同数据类型的数据。
+ 集合：大小可动态扩展，可以存储各种类型的数据。

**JDK，JRE，JVM有什么区别？**

+ JDK：Java Development Kit，Java开发工具包，提供了Java的开发环境和运行环境。包含了编译Java源文件的编译器Javac，还有调试和分析的工具。
+ JRE：Java Runtime Environment，Java运行环境，包含Java虚拟机及一些基础类库。
+ JVM：Java Virtual Machine，Java虚拟机，提供执行字节码文件的能力。

**Java中的native关键字**

native方法称为本地方法。在java源程序中以关键字"native"声明，不提供函数体。其实现使用C/C++语言在另外的文件中编写，编写的规则遵循Java本地接口的规范(简称JNI)。简而言就是Java中声明的可调用的使用C/C++实现的方法。

**你不得不知道的哈希表（[知乎](https://zhuanlan.zhihu.com/p/77533501)）**

**Object类里面的方法**

+ getClass()：返回此 Object 的运行类
+ hashCode()：用于获取对象的哈希值
+ equals()：用于确认两个对象是否相同
+ clone()：创建并返回此对象的一个副本
+ toString()：返回该对象的字符串表示
+ finalize()：当对象不再被任何对象引用时，GC会调用该对象的finalize()方法

**运行时异常和检查时异常区别**

Java提供了两类主要的异常：runtime exception和checked exception。checked 异常也就是我们经常遇到的IO异常，以及SQL异常都是这种异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch。所以，面对这种异常不管我们是否愿意，只能自己去写一大堆catch块去处理可能的异常。 

但是另外一种异常：runtime exception，也称运行时异常，我们可以不处理。当出现这样的异常时，总是由虚拟机接管。比如：我们从来没有人去处理过NullPointerException异常，它就是运行时异常，并且这种异常还是最常见的异常之一。 

出现运行时异常后，系统会把异常一直往上层抛，一直遇到处理代码。如果没有处理块，到最上层，如果是多线程就由Thread.run()抛出，如果是单线程就被main()抛出。抛出之后，如果是线程，这个线程也就退出了。如果是主程序抛出的异常，那么这整个程序也就退出了。运行时异常是Exception的子类，也有一般异常的特点，是可以被Catch块处理的。只不过往往我们不对他处理罢了。也就是说，你如果不对运行时异常进行处理，那么出现运行时异常之后，要么是线程中止，要么是主程序终止。 

如果不想终止，则必须扑捉所有的运行时异常，决不让这个处理线程退出。队列里面出现异常数据了，正常的处理应该是把异常数据舍弃，然后记录日志。不应该由于异常数据而影响下面对正常数据的处理。在这个场景这样处理可能是一个比较好的应用，但并不代表在所有的场景你都应该如此。如果在其它场景，遇到了一些错误，如果退出程序比较好，这时你就可以不太理会运行时异常，或者是通过对异常的处理显式的控制程序退出。

**面向对象都有哪些特性**

封装、继承、多态、抽象。

**浅拷贝和深拷贝的区别**

+ 浅拷贝拷贝出当前对象的一个副本，这个新对象和当前对象处于不同的堆内存中，两个对象的基本数据类型的值完全一样，但是引用数据类型还是指向同一个对象。
+ 深拷贝拷贝出当前对象的一个副本，这个新对象和当前对象处于不同的堆内存中，两个对象的基本数据类型的值完全一样，引用数据类型指向的对象也拷贝出了一份一模一样的副本。

**Java 如何重写对象的 equals 方法和 hashCode 方法**

[https://www.cnblogs.com/yuxiaole/p/9570850.html](https://www.cnblogs.com/yuxiaole/p/9570850.html)

**String类相关**

+ String对象的intern()方法会得到字符串对象在常量池中对应的字符串的引用，如果常量池中没有对应的字符串，则该字符串将被添加到常量池中，然后返回常量池中字符串的引用。
+ 字符串的+操作其本质是创建了StringBuilder对象进行append操作，然后将拼接后的StringBuilder对象用toString方法处理成String对象。

**Java中有几种类型的流**

输入流、输出流、字节流、字符流。

**字节流如何转为字符流**

+ 字节输入流转字符输入流通过InputStreamReader实现，该类的构造函数可以传入InputStream对象。
+ 字节输出流转字符输出流通过OutputStreamWriter实现，该类的构造函数可以传入OutputStream对象。

**字节流和字符流的区别**

字节流可以处理所有类型数据，字符流只能处理字符数据。

**如何实现对象克隆**

+ 实现Cloneable接口并重写Object类中的clone()方法。
+ 实现Serializable接口，通过对象的序列化和反序列化实现克隆。

**写一个ArrayList的动态代理类**

```java
List<String> list = new ArrayList<>();
List<String> proxyInstance = (List<String>) Proxy.newProxyInstance(list.getClass().getClassLoader(), list.getClass().getInterfaces(), new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(list, args);
    }
});
```

**动态代理和静态代理的区别**

+ 静态代理通常只代理一个类，动态代理是代理一个接口下的多个实现类。
+ 静态代理事先知道要代理的是什么，而动态代理不知道要代理什么，只有在运行时才知道。

**JDK动态代理和CGLIB动态代理的区别**

+ JDk动态代理是实现InvocationHandler接口的invoke方法，代理的是接口，也就是业务类必须要实现接口，通过Proxy里的newProxyInstance得到代理对象。
+ CGLIB动态代理代理的是类，不需要业务类实现接口，通过派生的子类来实现代理，通过在运行时，动态修改字节码达到修改类的目的。

**Java数组和链表的区别**

+ 数组一旦初始化，长度就不能改变；链表长度可以改变。
+ 数组在内存中是连续存储的，链表不是连续存储的。
+ 数组查找快，插入删除慢；链表查找慢，插入删除快。

**Java8的Stream中的中间操作方法**

+ 过滤：filter()
+ 截断流：limit()
+ 跳过元素：skip(n)
+ 筛选：distinct()
+ 映射：map() flatMap()
+ 排序：sorted()

**Comparable和Comparator区别是什么？**

[Java 解惑：Comparable 和 Comparator 的区别](https://zhuanlan.zhihu.com/p/24081048)