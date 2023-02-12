---
title: Markdown基本语法
date: 2020-04-16 23:54:01
tags:
categories:
---
## 标题

```
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```

## 字体

```
**这是加粗的文字**
*这是倾斜的文字*
***这是斜体加粗的文字***
~~这是加删除线的文字~~
```

**这是加粗的文字**

*这是倾斜的文字*

***这是斜体加粗的文字***

~~这是加删除线的文字~~

## 引用

```
> 这是引用的内容

>> 这是引用的内容
```

> 这是引用的内容

>> 这是引用的内容

## 分割线

```
---
----
***
*****
```
---

----

***

*****

## 图片

```
![](http://static.runoob.com/images/runoob-logo.png)
```
<img src="http://static.runoob.com/images/runoob-logo.png" width="140px"/>

## 超链接

```
[简书](http://jianshu.com "jianshu")
[百度](http://baidu.com "baidu")
```

[简书](http://jianshu.com "jianshu")

[百度](http://baidu.com "baidu")

## 列表

### 无序列表

```
- 列表内容
+ 列表内容
* 列表内容
```

- 列表内容
+ 列表内容
* 列表内容

注意：- + * 跟内容之间都要有一个空格

### 有序列表

```
1. 列表内容
2. 列表内容
3. 列表内容
```

1. 列表内容
2. 列表内容
3. 列表内容

注意：序号跟内容之间要有空格

### 嵌套列表

```
1. 一级列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表
2. 列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表
3. 列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表
```

1. 一级列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表
2. 列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表
3. 列表内容
   1. 二级列表
   2. 二级列表
   3. 二级列表

上一级和下一级之间敲三个空格即可

## 表格

```
|隔离级别|脏读|不可重复读|幻影读|
|:--:|:--:|:--:|:--:|
|READ-UNCOMMITTED|√|√|√|
|READ-COMMITTED|×|√|√|
|REPEATABLE-READ|×|×|√|
|SERIALIZABLE|×|×|×|
```

|隔离级别|脏读|不可重复读|幻影读|
|:--:|:--:|:--:|:--:|
|READ-UNCOMMITTED|√|√|√|
|READ-COMMITTED|×|√|√|
|REPEATABLE-READ|×|×|√|
|SERIALIZABLE|×|×|×|

## 代码

```
`create database test;`
```

`create database test;`

```
function fun(){
	echo "这是一句非常牛逼的代码";
}
fun();
```

## 流程图

```flow
st=>start: 开始
op=>operation: My Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
&```

## 参考

+ [Markdown基本语法](https://www.jianshu.com/p/191d1e21f7ed)