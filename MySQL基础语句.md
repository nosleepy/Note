---
title: MySQL基础语句
date: 2020-04-28 11:06:12
tags:
categories:
- 数据库
---

## 登录、退出

登录mysql

```
mysql -h 主机地址 -u root -p
```
退出mysql

```
exit / quit
```

## 操作数据库

查询所有数据库的名称

```
show databases;
```

创建数据库

```
create database 数据库名称;
```

删除数据库

```
drop database 数据库名称;
```

使用数据库

```
use 数据库名称;
```

## 操作表

查询某个数据库中所有的表名称

```
show tables;
```

查询表结构

```
desc 表名称;
```

创建表

```
create table 表名称
(
	列名称1 数据类型1,
	列名称2 数据类型2,
	列名称n 数据类型n
);
```

数据库类型

```
1.int：整数类型 //age int
2.double：小数类型 //score double(5,2)
3.date：日期，年月日格式，yyyy-MM-dd
4.datetime：日期，年月日时分秒格式，yyyy-MM-dd HH:mm:SS
5.timestamp：日期，年月日时分秒格式，yyyy-MM-dd HH:mm:SS //该字段不赋值或者为null，默认使用系统时间
6.varchar：字符串 // name varchar(20)，姓名最大20个字符
```

修改表

```
alter table 表名称 rename to 新的表名称; //修改表名称
alter table 表名称 add 列名称 数据类型; //增加一列
alter table 表名称 change 列名称 新的列名称 新的数据类型; //修改列的名称和数据类型
alter table 表名称 modify 列名称 新的数据类型//修改列的数据类型
alter table 表名称 drop 列名称; //删除列
```

删除表

```
drop table 表名称; //删除表
truncate table 表名称 //删除表，然后再创建一张一样的空表
```

## 增删查改

添加数据

```
insert into 表名称(列名1,列名2,...,列名n) values(值1,值2,...,值n);
```

删除数据

```
delete from 表名称 where 条件;
```

修改数据

```
update 表名称 set 列名1 = 值1,列名2 = 值2,...,列名n = 值n where 条件;
```

查询数据

```
select 字段列表 from 表名列表 where 条件列表 group by 分组字段 having 分组之后的条件 order by 排序 limit 分页限定;
```

一、基础查询

```
1.select name,age from student; //多个字段的查询
2.select distinct address from student; //去除重复
3.select name,math,english,math+english from student; //计算列
4.select name,math,english,math+english (as) total from student; //起别名
```

二、条件查询

1. where子句后跟条件
2. 运算符
   + `>、<、>=、<=、=、<>`
      + `select * from student where id > 10;`
   + `between ... and ...`
      + `select * from student where age between 20 and 30;`
   + `in (集合)`
      + `select * from student where age in (22,18,25);`
   + `like`
      + 占位符 `_` 代表单个任意字符，`%` 代表多个任意字符
      + `select * from student where name like '马%';`
	  + `select * from student where name like '_化%';`
	  + `select * from student where name like '___';`
	  + `select * from student where name like '%马%';`
   + `is null`
   + `and / &&`
   + `or / ||`
   + `not / !`

三、排序查询

1. 语法：order by 子句
2. order by 排序字段 排序方式
3. 升序：ASC（默认），降序：DESC
4. `select * from student order by math asc, english asc;`

四、聚合函数

1. count：计算个数
   + `select count(name) from student;`
   + `select count(*) from student;`
2. max：计算最大值
   + `select max(math) from student;`
3. min：计算最小值
   + `select min(math) from student;`
4. sum：计算和
   + `select sum(math) from student;`
5. avg：计算平均值
   + `select avg(math) from student;`

五、分组查询

1. 语法：group by 分组字段 having 分组之后的条件
2. 分组查询示例
   + `select sex, avg(math), count(id) from student group by sex; `
   + `select sex, avg(math), count(id) from student where math > 70 group by sex;`
   + `select sex, avg(math), count(id) from student where math > 70 group by sex having count(id) > 2;`
   + `select sex, avg(math), count(id) 人数 from student where math > 70 group by sex having 人数 > 2;`

六、分页查询

1. 语法：limit 开始的索引,每页查询的条数;(针对mysql)
2. 分页查询示例
   + `select * from student limit 0,3; //查询第一页的3条数据`
   + `select * from student limit 1,3; //查询第二页的3条数据`

## 约束

1、主键约束：primary key

```
1、添加
alter table stu modify name varchar(20) primary key;

2、删除
alter table stu drop primary key;

3、自动增长
auto_increment 数值类型的列完成自动递增
```

2、非空约束：not null

```
1、创建表时添加约束
create table stu
(
	id int,
	name varchar(20) not null
)

2、创建表完之后添加约束
alter table stu modify name varchar(20) not null;

3、删除非空约束
alter table stu modify name varchar(20);
```

3、唯一约束：unique

```
1、添加
alter table stu modify name varchar(20) unique;

2、删除
alter table stu drop index name;
```

4、外键约束：foreign key

```
1、创建表时可以添加外键
create table 表名
(
	外键列
	constraint 外键名称 foreign key (外键列名称) references 主表名称(主表列名称)
);

2、创建表之后添加外键
alter table emp add constraint emp_dep_fk foreign key (dep_id) references dep(id);

3、删除外键
alter table emp drop foreign key emp_dep_fk;
```

## 多表查询
  
1、内连接查询

+ 隐式内连接：使用where条件消除无用的数据
   + `select * from emp,dept where emp.dept_id = dept.id;`
   + `select emp.name,emp.gender,dept.name from emp,dept where emp.dept_id = dept.id;`
+ 显示内连接
   + 语法：`select 字段列表 from 表名1 inner join 表名2  on 条件;`
   + `select * from emp [inner] join dept on emp.dept_id = dept.id;`

2、外连接查询

+ 左外连接
   + 语法：`select 字段列表 from 表1 left [outer] join 表2 on 条件;`
   + `select t1.*,t2.name from emp t1 left join dept t2 on t1.dept_id=t2.id;`
   + 查询的是左表所有数据以及其交集部分
+ 右外连接
   + 语法：`select 字段列表 from 表1 right [outer] join 表2 on 条件;`
   + 查询的是右表所有数据以及其交集部分

3、子查询

+ 概念：查询中嵌套查询，称嵌套查询为子查询
+ 情况1：子查询的结果是单行单列的
子查询可以作为条件，使用运算符去判断，运算符 > < >= <= =
+ 情况2：子查询的结果是多行单列的，使用运算符in来判断
+ 情况3：子查询的结果是多行多列的，子查询可以作为一张虚拟表

```
#子查询情况1
select * from emp order by salary desc limit 0,1;
select max(salary) from emp;
select * from emp where salary = (select max(salary) from emp);
select * from emp where salary < (select avg(salary) from emp);
#子查询情况2
select id from dept where name in ("财务部","开发部");
select * from emp where dept_id in (select id from dept where name in ("财务部","开发部"));
#子查询情况3
select * from emp where join_date > "2011-11-11";
select * from dept t1,(select * from emp where join_date > "2011-11-11") t2 where t1.id = t2.dept_id;
```

## 事务

数据库隔离级别

+ READ UNCOMMITTED（读未提交）
+ READ COMMITTED（读已提交）
+ REPEATABLE READ（可重复读）
+ SERIALIZABLE（可串行化）

查看数据库隔离级别

```
select @@tx_isolation; // 当前会话
select @@global.tx_isolation; // 全局的
```

mysql数据库事务开关（临时有效）

```
set autocommit = 1; // 开启自动提交事务
set autocommit = 0; // 关闭自动提交事务
show variables like '%autocommit%'; // 查看事务开关
```

修改数据库的事务级别

```
set global transaction isolation level read uncommitted; //全局的
set session transaction isolation level read uncommitted; //当前会话
```

事务操作

```
-- 开启事务
begin;
-- 查询指定用户
select * from user where id = 1;
-- 修改指定用户余额
update user set money = 500 where id = 1;
-- 提交事务
-- commit;
-- 回滚事务
-- rollback;
```