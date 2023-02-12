---
title: MySQL索引
date: 2020-05-02 23:51:13
tags:
categories:
- 数据库
---

## 介绍

索引用于快速找出在某个列中有一特定值的行。不使用索引，MySQL必须从第一条记录开始读完整个表，直到找出相关的行，表越大查询数据所花费的时间就越多。如果表中查询的列有索引，MySQL能够快速到达一个位置去搜索数据文件，而不必查看所有数据，那么将会节省很大一部分时间。

例如：有一张person表，其中有2W条记录，记录着2W个人的信息。有一个Phone的字段记录每个人的电话号码，现在想要查询出电话号码为xxxx的人的信息。

如果没有索引，那么将从表中第一条记录一条条往下遍历，直到找到该条信息为止。

如果有了索引，那么会将 Phone 字段，通过一定的方法进行存储，好让查询该字段上的信息时，能够快速找到对应的数据，而不必在遍历2W条数据了。其中MySQL中的索引的存储类型有两种：BTREE、HASH。 也就是用树或者Hash值来存储该字段，更详细的查找逻辑就需要会算法的知识了。我们现在只需要知道索引的作用，功能是什么就行。

## 优缺点

**优点**

1. 所有的MySql列类型(字段类型)都可以被索引，也就是可以给任意字段设置索引。
2. 大大加快数据的查询速度。

**缺点**

1. 创建索引和维护索引要耗费时间，并且随着数据量的增加所耗费的时间也会增加。
2. 索引也需要占空间，我们知道数据表中的数据也会有最大上限设置的，如果我们有大量的索引，索引文件可能会比数据文件更快达到上限值。
3. 当对表中的数据进行增加、删除、修改时，索引也需要动态的维护，降低了数据的维护速度。

**使用原则**

1. 对经常更新的表就避免对其设置过多的索引，对经常用于查询的字段应该创建索引。
2. 数据量小的表最好不要使用索引，因为由于数据较少，可能查询全部数据花费的时间比遍历索引的时间还要短，索引就可能不会产生优化效果。
3. 在一个列上(字段上)不同值较少的不要建立索引，比如在学生表的"性别"字段上只有男，女两个不同值。相反的，在一个字段上不同值较多的可是建立索引。

## 分类

索引是在**存储引擎**中实现的，也就是说不同的存储引擎，会使用不同的索引：

MyISAM和InnoDB存储引擎：只支持BTREE索引， 也就是说默认使用BTREE，不能够更换。（但是innoDB存储引擎支持hash索引是自适应的，innoDB存储引擎会根据表的使用情况自动为表生成hash索引，不能人为干预是否在一张表中生成hash索引。）

MEMORY/HEAP存储引擎：支持HASH和BTREE索引。

**存储引擎的类型及特点**

|引擎名称|优点|缺陷|应用场景|
|:--:|:--:|:--:|:--:|
|MyISAM|独立于操作系统，这说明可以轻松地将其从Windows服务器移植到Linux服务器|不支持事务/行级锁/外键约束|适合管理邮件或Web服务器日志数据|
|InnoDB|健壮的事务型存储引擎；支持事务/行级锁/外键约束/自动灾难恢复/AUTO_INCREMENT|-|需要事务支持，并且有较高的并发读取频率|
|MEMORY|为得到最快的响应时间，采用的逻辑存储介质是系统内存|当mysqld守护进程崩溃时，所有的Memory数据都会丢失；不能使用BLOB和TEXT这样的长度可变的数据类型|临时表|
|MERGE|是MyISAM类型的一种变种。合并表是将几个相同的MyISAM表合并为一个虚表|-|常应用于日志和数据仓库|
|ARCHIVE|归档的意思，支持索引，拥有很好的压缩机制|仅支持插入和查询功能|经常被用来当做仓库使用|

**索引的分类**

1. 单列索引：一个索引只包含单个列，但一个表中可以有多个单列索引。
   + 普通索引：MySQL中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点。
   + 唯一索引：索引列中的值必须是唯一的，但是允许为空值，
   + 主键索引：是一种特殊的唯一索引，不允许有空值。
2. 组合索引：一个的索引包含多个列，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用，使用组合索引时遵循**最左前缀**。
3. 全文索引：要求只有在MyISAM引擎上才能使用，只能在CHAR、VARCHAR、TEXT类型字段上使用全文索引。就是在一堆文字中，通过其中的某个关键字等，就能找到该字段所属的记录行，比如有"你是个大煞笔，二货 ..." 通过大煞笔，可能就可以找到该条记录。
4. 空间索引：空间索引是对空间数据类型的字段建立的索引，MySQL中的空间数据类型有四种，GEOMETRY、POINT、LINESTRING、POLYGON。在创建空间索引时，使用SPATIAL关键字。要求，引擎为MyISAM，创建空间索引的列，必须将其声明为NOT NULL。

## 执行计划

使用`explain`语句去查看分析结果 

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/执行计划.png" width="880px"/>

各字段含义

+ `id`：select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序 
+ `select_type`：查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询
+ `type`：访问类型，sql查询优化中一个很重要的指标，结果值从好到坏依次是：`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`
+ `possible_keys`：查询涉及到的字段上存在索引，则该索引将被列出，但不一定被查询实际使用
+ `key`：实际使用的索引，如果为NULL，则没有使用索引。 
+ `key_len`：表示索引中使用的字节数，查询中使用的索引的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。key_len是根据表定义计算而得的，不是通过表内检索出的
+ `ref`：显示索引的那一列被使用了，如果可能，是一个常量const
+ `rows`：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数
+ `Extra`：不适合在其他字段中显示，但是十分重要的额外信息

## 使用

### 在创建表时创建索引

创建索引：单列索引（普通、唯一、主键）、组合索引、全文索引和空间索引。

格式：`CREATE TABLE 表名[字段名 数据类型] [UNIQUE|FULLTEXT|SPATIAL|...] [INDEX|KEY] [索引名字] (字段名[length]) `

**1、创建普通索引**

创建普通索引，创建索引时未指定索引的名，会自动帮我们用字段名当作索引名

```
CREATE TABLE book(
id INT NOT NULL PRIMARY KEY,
name VARCHAR(50) NOT NULL,
author VARCHAR(20) NOT NULL,
info VARCHAR(255) NULL,
INDEX(author));
```

查看表的创建

```
SHOW CREATE TABLE book;
-------------------------------结果----------------------------------
CREATE TABLE `book` (
  `id` int(11) NOT NULL,
  `name` varchar(50) NOT NULL,
  `author` varchar(20) NOT NULL,
  `info` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `author` (`author`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

用`EXPLAIN`关键字，来查看索引是否正在被使用，并且输出其使用的索引信息

```
EXPLAIN SELECT * FROM book WHERE author = '罗贯中';
```

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/普通索引.png" width="880px"/>

**2、创建唯一索引**

创建唯一索引

```
CREATE TABLE tab1(
id INT(5) NOT NULL,
name CHAR(20) NOT NULL,
UNIQUE INDEX uniqId(id)
);
```
查看表的创建

```
SHOW CREATE TABLE tab1;
---------------------------------结果--------------------------------
CREATE TABLE `tab1` (
`id` int(5) NOT NULL,
`name` char(20) NOT NULL,
UNIQUE KEY `uniqId` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

查看索引使用信息

```
EXPLAIN SELECT * FROM tab1 WHERE id = 1;
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/唯一索引.png" width="880px"/>

**3、创建主键索引**

创建主键索引

```
CREATE TABLE tab2(
id INT(4) NOT NULL,
name char(20) DEFAULT NULL,
PRIMARY KEY(id));
```

查看表的创建

```
SHOW CREATE TABLE tab2;
---------------------------------结果--------------------------------
CREATE TABLE `tab2` (
`id` int(4) NOT NULL,
`name` char(20) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

查看索引使用信息

```
EXPLAIN SELECT * FROM tab2 WHERE id = 1;
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/主键索引.png" width="880px"/>

**4、创建组合索引**

创建组合索引

```
CREATE TABLE tab3(
id INT(4) NOT NULL,
name CHAR(20) NOT NULL,
age INT(3) NOT NULL,
info VARCHAR(255),
INDEX multiIdx(id,name,age)
);
```
查看表的创建

```
SHOW CREATE TABLE tab3;
---------------------------------结果--------------------------------
CREATE TABLE `tab3` (
`id` int(4) NOT NULL,
`name` char(20) NOT NULL,
`age` int(3) NOT NULL,
`info` varchar(255) DEFAULT NULL,
KEY `multiIdx` (`id`,`name`,`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

查看索引使用信息

```
EXPLAIN SELECT * FROM tab3 WHERE id = 1 AND name = 'nana';
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/组合索引1.png" width="880px"/>

```
EXPLAIN SELECT * FROM tab3 WHERE age = 3 AND name = 'nana';
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/组合索引2.png" width="880px"/>

**最左前缀**：组合索引遵从了最左前缀，利用索引中最左边的列集来匹配行，这样的列集称为最左前缀。例如，这里由id、name和age3个字段构成的索引，索引行中就按id/name/age的顺序存放，索引组合中的字段可以是(id，name，age)、(id，name)或者(id)。如果要查询的字段不构成最左面的前缀原则，那么就不会用索引，比如，age或者（name，age）组合就不会使用索引查询。

**5、创建全文索引**

创建全文索引，支持的字段类型为CHAR、VARCHAR和TEXT，存储引擎为MyISAM

```
CREATE TABLE tab4(
id INT(4) NOT NULL,
name CHAR(20) NOT NULL,
age INT(3) NOT NULL,
info VARCHAR(255),
FULLTEXT INDEX fullTxtIdx(info)
)ENGINE=MyISAM;
```

查看表的创建

```
SHOW CREATE TABLE tab4;
---------------------------------结果--------------------------------
CREATE TABLE `tab4` (
`id` int(4) NOT NULL,
`name` char(20) NOT NULL,
`age` int(3) NOT NULL,
`info` varchar(255) DEFAULT NULL,
FULLTEXT KEY `fullTxtIdx` (`info`)
) ENGINE
```

查看索引使用信息

```
EXPLAIN SELECT * FROM tab4 WHERE MATCH(info) AGAINST('white');
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/全文索引.png" width="880px"/>

**6、创建空间索引**

创建空间索引

```
CREATE TABLE tab5(
geo GEOMETRY NOT NULL,
SPATIAL INDEX spatIdx(geo)
)ENGINE=MyISAM;
```

查看表的创建

```
SHOW CREATE TABLE tab5;
---------------------------------结果--------------------------------
CREATE TABLE `tab5` (
`geo` geometry NOT NULL,
SPATIAL KEY `spatIdx` (`geo`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
```

### 在创建表后创建索引

**在已经存在的表上创建索引**

`ALTER TABLE 表名 ADD[UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] [索引名] (索引字段名(长度))`

为表添加索引

```
ALTER TABLE book ADD INDEX BkNameIdx(name(30));
```

**使用CREATE INDEX创建索引**

`CREATE [UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] 索引名称 ON 表名(创建索引的字段名[length])`

为book表增加一个普通索引info

```
CREATE INDEX BkInfoIdx ON book(info(10));
```

### 删除索引

**使用ALTER DROP删除索引**

`ALTER TABLE 表名 DROP INDEX 索引名`

删除book表中的名称为BkInfoIdx的索引

```
ALTER TABLE book DROP INDEX BkInfoIdx;
```

**使用DROP INDEX删除索引**

`DROP INDEX 索引名 ON 表名;`

删除book表中的名称为BkNameIdx的索引

```
DROP INDEX BkNameIdx ON book;
```

### 查看所引

查看表的索引

```
SHOW INDEX FROM book;
```
<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/索引信息.png" width="880px"/>

各个字段含义

+ `Table`：创建索引的表
+ `Non_unique`：表示索引是否唯一，其中1代表：非唯一索引， 0代表：唯一索引
+ `Key_name`：索引名称
+ `Seq_in_index`：表示该字段在索引中的位置，单列索引该值为1，组合索引为每个字段在索引定义中的顺序(这个只需要知道单列索引该值为1，组合索引为别的)
+ `Column_name`：表示定义索引的列字段。
+ `Sub_part`：表示索引的长度，当字段值为null时，索引长度为null
+ `Null`：表示该字段是否能为空值
+ `Index_type`：表示索引类型。

## 技巧

**1.索引不会包含有NULL的列**

只要列中包含有NULL值，都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。

**2.使用短索引**

对字符串列进行索引，如果可以就应该指定一个前缀长度。例如，如果有一个char（255）的列，如果在前10个或20个字符内，多数值是唯一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。

**3.索引列排序**

mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作，尽量不要包含多个列的排序，如果需要最好给这些列建复合索引。

**4.like语句操作**

一般情况下不鼓励使用like操作，如果非使用不可，注意正确的使用方式。like ‘%aaa%’不会使用索引，而like ‘aaa%’可以使用索引。

**5.不要在列上进行运算**

**6.不使用NOT IN 、<>、!=操作，但<,<=，=，>,>=,BETWEEN,IN是可以用到索引的**

**7.索引要建立在经常进行select操作的字段上**

这是因为，如果这些列很少用到，那么有无索引并不能明显改变查询速度。相反，由于增加了索引，反而降低了系统的维护速度和增大了空间需求。

**8.索引要建立在值比较唯一的字段上**

**9.对于那些定义为text、image和bit数据类型的列不应该增加索引。因为这些列的数据量要么相当大，要么取值很少**

**10.在where和join中出现的列需要建立索引**

**11.where的查询条件里有不等号(where column != …),mysql将无法使用索引**

**12.如果where子句的查询条件里使用了函数(如：where DAY(column)=…),mysql将无法使用索引**

**13.在join操作中(需要从多个数据表提取数据时)，mysql只有在主键和外键的数据类型相同时才能使用索引，否则及时建立了索引也不会使用**

## 参考

+ [mysql索引的使用](https://www.cnblogs.com/nananana/p/10387720.html)
+ [SQL执行计划详解explain](https://www.cnblogs.com/yhtboke/p/9467763.html)
+ [mysql索引使用技巧及注意事项](https://www.cnblogs.com/liqianglog/p/11125982.html)