---
title: SQL函数
date: 2021-04-05 13:55:00
tags:
categories:
- 数据库
---

### 时间函数

DATE_FORMAT()：格式化输出日期 [lintcode](https://www.lintcode.com/learning/32/1779)

```sql
SELECT DATE_FORMAT(date,format);
```

EXTRACT()：提取指定的时间信息 [lintcode](https://www.lintcode.com/learning/32/1842)

```sql
SELECT EXTRACT(unit FROM date) FROM `table`;
```

NOW()：可以用来返回当前日期和时间 格式：YYYY-MM-DD hh:mm:ss

CURDATE()：可以用来返回当前日期 格式：YYYY-MM-DD

CURTIME()：可以用来返回当前时间 格式：hh:mm:ss

### 时间函数运算

DATE_SUB()：减少时间 [lintcode](https://www.lintcode.com/learning/34/1815)

```sql
SELECT DATE_SUB(date, INTERVAL expr type) FROM table_name;
```

DATE_ADD() ：增加时间 [lintcode](https://www.lintcode.com/learning/34/1814)

```sql
SELECT DATE_ADD(date, INTERVAL expr type) FROM table_name;
```

### 时间差值函数

DATEDIFF ()：计算日期差，天数 [lintcode](https://www.lintcode.com/learning/32/1777)

```sql
SELECT DATEDIFF(时间1,时间2) AS date_diff FROM courses;
```

TIMESTAMPDIFF()：计算两个日期相差的年月日 [lintcode](https://www.lintcode.com/learning/32/1777)

```sql
SELECT TIMESTAMPDIFF (类型,时间1,时间2) AS year_diff;
```

### NULL函数

ISNULL：会根据该字段是否为NULL值返回0或1，如果是NULL则返回1，不是则返回0

IFNULL(column_name, value：)函数接收两个参数，第一个参数是列名，第二个如果该列某个字段是NULL则返回value值

COALESCE(column_name, value)：是和IFNULL是同一个用法

ROUND(X)： 返回参数X的四舍五入的一个整数。

ROUND(X, D)： 返回参数X的四舍五入的有 D 位小数的一个数字。如果D为0，结果将没有小数点或小数部分。

REGEXP：正则匹配

```sql
// 查询以字母 'D' 到 'O' 开头的课程
select name from courses where name regexp '^[D-O].*';
```

CONCAT：连接函数

```sql
concat(ban,"%")
```

### 条件判断

```sql
case 字段名 when 1 then 0 end

IF(condition, value_if_true, value_if_false)
```

```sql
--简单Case函数
CASE sex
WHEN '1' THEN '男'
WHEN '2' THEN '女'
ELSE '其他' END

--Case搜索函数
CASE WHEN sex = '1' THEN '男'
WHEN sex = '2' THEN '女'
ELSE '其他' END
```