---
title: MySQL基础
date: 2021-03-05 14:48:36
tags:
- MySQL
categories: MySQL基础
mathjax: true
---

## 数据库的概述

**什么是数据库：**
数据库就是用来存储和管理数据的仓库！

**数据库存储数据的优点：**

* 可以存储大量的数据
* 方便检索
* 保持数据的一致性、完整性
* 安全，可共享
* 通过组合分析，可产生新数据

 <!-- more --> 

**数据库的发展历程：**

* 没有数据库，使用磁盘文件存储数据
* 层次结构模型数据库
* 网状结构模型数据库
* 关系结构模型数据库：使用二维表格来存储数据
* 关系-对象模型数据库

MySQL就是关系型数据库！

**常见数据库：**

* Oracle：甲骨文
* BD2：IBM
* SQL Server：微软
* Sybase：塞尔斯
* MySQL：甲骨文

**理解数据库：**

* RDBMS = manager + database
* database = N 个 table
* table：
  * 表结构：定义表的列名和列类型
  * 表记录： 一行一行的记录

我们现在所说的数据库泛指：关系型数据库管理系统(RDBMS-Relational database management system)，即数据库服务器。当我们安装了数据库服务器后，就可以在数据库服务器中创建数据库，每个数据库中还可以包含多张表。
数据库表就是一个多行多列的表格。在创建时，需要指定表的列数，以及列名称，列类型等信息。而不用指定表格的行数，行数是没有上限的。
当把表格创建好了之后，就可以向表格中添加数据了。向表格添加数据是以行为单位的。

## Java应用与数据库的关系

应用程序向数据库请求数据、并显示结果。数据库服务器响应和提供数据。

---

## 安装MySQL

安装文件存放路径不能有中文

## 删除MySQL

```mysql
net stop mysql
net start mysql
```

删除前先停止mysql；添加删除程序；到安装目录删除MySQL；删除：c:\Documents and settings\All Users\Application Data\MySQL c:\ProgramData\MySQL；查看注册表 regedit：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\MySQL、HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\MySQL、搜索mysql，最后重启电脑。

## MySQL安装路径以及配置信息

**MySQL安装成功后会在两个目录中存储文件：**

* D:\Program Files\MySQL Server
* c:\ProgramData\MySQL Server

**MySQL重要文件：**

* mysql.exe：客户端程序
* mysqld.exe：服务器程序
* my.ini：服务器配置文件

**my.ini：MySQL最为重要的配置文件：**

* 配置MySQL的端口
* 配置字符编码
* 配置二进制数据大小上限

## 开启关闭服务器以及登录退出客户端

开启服务器：net.start.mysql；关闭服务器：net.stop.mysql

登陆服务器：mysql -uroot -p123 -hlocalhost

* -u后面跟用户名
* -p后面跟密码
* -h后面跟ip

退出服务器：exit或quit

---

## SQL语言的概述

**SQL**

1.什么是SQL：结构化查询语言(Structured Query Language)。
2.SQL的作用：客户端使用SQL来操作服务器。
3.SQL标准(例如SQL99)。
4.SQL方言：某种DBMS不只能支持SQL标准，而且还会有一些自己独有的语法，这称为方言！例如limit语句只在MySQL中可以使用。

**SQL语法**

1.SQL语句可以在单行或多行书写，以分号结尾
2.可以使用空格和缩进增强语句的可读性
3.MySQL不区别大小写，建议使用大写

**SQL语句分类**

1.DDL(Data Definition Language)：数据定义语言，用来定义数据库对象：库、表、列
2.DML(Data Manipulation Language)：数据操作语言，用来定义数据库记录
3.DCL(Data Control Language)：数据控制语言，用来定义访问权限和安全等级
4.DQL(Data Query Language)：数据查询语言，用来查询记录

---

## DDL 操作数据库

**1.查询所有数据库**

```mysql
SHOW DATABASES;
```

2.切换数据库

```mysql
USE 数据库名;
```

**3.创建数据库**

```mysql
CREATE DATABASE [IF NOT EXISTS] 数据库名[CHARSET=UTF8];
```

4.删除数据库

```mysql
DROP DATABASE [IF EXISTS] 数据库名;
```

**5.修改数据库编码**

```mysql
ALTER DATABASE 数据库名 CHARACTER SET UTF8;
```

---

## 数据类型介绍

int：整型
double：浮点型，例如double(5,2)表示最多5位，其中必须有2位小数，即最大值为999.99；
decimal：浮点型，在表价钱方面使用该类型，不会出现精度损失问题；
char：固定长度字符串类型，char(255)，数据的长度不是指定长度，补足到指定长度！例如：身份证，时间
varchar：可变长度字符串类型：varchar(65535)。例如：用户名
text(clob)：字符串类型

* tinytext：$2^8-18$B
* text：$2^{16}-18$B
* mediumtext：$2^{24}-18$B
* longtext：$2^{32}-18$B

blob：字节类型 可变长度二进制类型

* tinyblob
* blob
* mediumblob
* longblob

date：日期类型，格式为：yyyy-MM-dd
time：时间类型，格式为：hh:mm:ss
timestamp：时间戳类型

---

## DDL操作表

**1.创建表**

```mysql
CREATE TABLE [IF NOT EXISTS] 表名(
	列名 列类型,
	列名 列类型,
	...
	列名 列类型
);
```

```mysql
CREATE TABLE tb_stu(
	number char(11),
	name varchar(50),
	age int,
	gender varchar(10)
);
```

**2.查看当前数据库中所有表名称**

```mysql
SHOW TABLES;
```

**3.查看指定表的创建语句**

```mysql
SHOW CREATE TABLE 表名;
```

4.查看表结构

```mysql
DESC 表名;
```

**5.删除表**

```mysql
DROP TABLE 表名;
```

6.修改表

```mysql
ALTER TABLE 表名 ADD(
	列名 列类型,
	列名 列类型,
	...
);
```

```mysql
ALTER TABLE 表名 MODIFY 列名 新列类型;
```

```mysql
ALTER TABLE 表名 DROP 列名;
```

```mysql
ALTER TABLE 原表名 RENAME TO 新表名
```

---

## DML

**1.插入数据**

```mysql
INSERT INTO 表名(
	列名1,列名2,列名3,...
)VALUES(
	列值1,列值2,列值3,...
);
```

```mysql
//插入所有列
INSERT INTO stu(
	number,name,age,gender
)VALUES(
	'ITCAST_0001', 'zhangsan', 28, 'male'
);
```

```mysql
//插入部分列，没有插入的列，为默认值NULL
INSERT INTO stu(
	number,name
)VALUES(
	'ITCAST_0002', 'lisi'
);
```

```mysql
//不给出插入列，那么默认值为插入所有列，值的顺序与创建列的顺序相同
INSERT INTO stu VALUES(
	'ITCAST_0003', 'wangwu', 54, 'male'
);
```

**2.修改数据**

```mysql
UPDATE 表名 SET 列名1=列值, 列名2=列值, ...[WHERE 条件];
```

条件必须是一个boolean类型的值或表达式：UPDATE t_person SET gender=‘男’, age = age + 1 WHERE sid=‘1’;
运算符：=、!=、<>、>、<、>=、<=、BETWEEN…AND、IN、IS NULL、NOT、OR、AND

**3.删除表记录**

```mysql
DELETE FROM 表名[WHERE 条件];
```

---

## DCL数据控制语言

**1.创建用户**

```mysql
CREATE USER 用户名@IP地址 IDENTIFIED BY '密码';//用户只能在指定的IP地址上登录
CREATE USER 用户名@'%' IDENTIFIED BY '密码';//用户可以在任意IP地址上登录
```

2.给用户授权

```mysql
GRANT 权限1, ... , 权限n ON 数据库.* TO 用户名@IP地址;
```

权限、用户、数据库
给用户分派在指定的数据库上的指定的权限

```mysql
GRANT ALL ON 数据库.* TO 用户名@IP地址;
```

**3.撤销权限**

```mysql
REVOKE 权限1, ... , 权限n ON 数据库.* FROM 用户名@IP地址;
```

撤销指定用户在指定数据库上的指定权限

**4.查看权限**

```mysql
SHOW GRANT FOR 用户名@IP地址;
```

**5.删除用户**

```mysql
DROP USER 用户名@IP地址;
```

---

## DQL基础查询

1.查询所有列

```mysql
SELECT * FROM 表名;
```

2.查询指定列

```mysql
SELECT 列1 [, 列2, ... , 列N] FROM 表名;
```

3.完全重复的记录只一次

```mysql
SELECT DISTINCT * 列1 [, 列2, ... , 列N] FROM 表名;
```

4.列运算

* 数量类型的列可以做加、减、乘、除

```mysql
SELECT 列名*1.5 FROM 表名;
SELECT 列名1+列名2 FROM 表名;//若与NULL相加为NULL
SELECT 列名1+IF NULL(列名2,0) FROM 表名;
```

* 字符串类型可以做连续运算

```mysql
SELECT CONCAT(列名1,列名2) FROM 表名;
SELECT CONCAT('我叫',列名1,',是个',列名2) FROM 表名;
```

* 转换NULL值，有时候需要把NULL值转换成其他值

```mysql
SELECT 列名1+IFNULL(列名2,0) FROM 表名;
```

* 给列起别名

```mysql
SELECT 列名1 [AS] 别名1,列名2 [AS] 别名2 FROM 表名;
```

## DQL条件查询

1.条件查询

```mysql
SELECT * FROM 表名 WHERE 条件;
```

2.模糊查询

```mysql
SELECT * FROM 表名 WHERE 列名 LIKE 条件;

SELECT * FROM emp WHERE ename LIKE '张_';
SELECT * FROM emp WHERE ename LIKE '张__';
SELECT * FROM emp WHERE ename LIKE '__';
SELECT * FROM stu WHERE sname LIKE '%刚';
SELECT * FROM stu WHERE sname LIKE '赵%';
SELECT * FROM stu WHERE sname LIKE '%小%';
```

3.排序

```mysql
SELECT * FROM 表名 ORDER BY 排序列;
SELECT * FROM 表名 ORDER BY 排序列 ASC;//升序
SELECT * FROM 表名 ORDER BY 排序列 DESC;//降序
```

```mysql
SELECT * FROM 表名 ORDER BY 排序列1 ASC, 排序列2 DESC;
SELECT * FROM 表名 ORDER BY 排序列1 ASC, 排序列2 DESC, 排序列3 ASC;
```

4.聚合函数

* COUNT

```mysql
SELECT COUNT(*) FROM 表名;
SELECT COUNT(列名) FROM 表名;
```

* MAX

```mysql
SELECT MAX(列名) FROM 表名;
```

* MIN

```mysql
SELECT MIN(列名) FROM 表名;
```

* SUM

```mysql
SELECT SUM(列名) FROM 表名;
```

* AVG

```mysql
SELECT AVG(列名) FROM 表名;
```

5.分组查询

```mysql
SELECT job, COUNT(*) FROM emp GROUP BY job;
SELECT gander, COUNT(*) FROM stu GROUP BY gander;
SELECT province, COUNT(*) FROM stu GROUP BY province;
SELECT deptno, COUNT(*) FROM emp WHERE sal > 15000 GROUP BY deptno;
SELECT deptno, COUNT(*) FROM emp WHERE sal > 15000 GROUP BY deptno HAVING COUNT(*) >= 2;
```

顺序：SELECT、FROM、WHERE、GROUP BY、HAVING、ORDER BY

6.limit子句

LIMIT用来限定查询结果的起始行，以及总行数；
例如：查询起始行为第5行，
一共查询3行记录，SELECT * FROM emp LIMIT 4,3;

```mysql
SELECT * FROM emp LIMIT 0,5;
SELECT * FROM emp LIMIT 8,5;
```

一页的记录数：10行；查询第3页。
（当前页-1）*每页记录数。