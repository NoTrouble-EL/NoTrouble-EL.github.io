---
title: MySQL进阶及查询练习
date: 2021-03-19 19:29:38
tags:
- MySQL
categories: MySQL基础
mathjax: true
---

## 单表查询练习

1.查询出部门编号为30的所有员工

```mysql
SELECT * FROM emp WHERE deptno = 30;
```

2.查询所有销售员的姓名、编号和部门编号

```mysql
SELECT ename, empno, deptno FROM emp WHERE job = '销售员';
```

 <!-- more --> 

3.找出奖金高于工资的员工

```mysql
SELECT * FROM emp WHERE comm > sal;
```

4.找出奖金高于工资60%的员工

```mysql
SELECT * FROM emp WHERE comm > sal*0.6;
```

5.找出部门编号为10中所有经理，和部门编号为20中所有销售员的详细资料

```mysql
SELECT * FROM emp WHERE (deptno = 10 AND job='经理') OR (deptno = 20 AND job='销售员');
```

6.找出部门编号为10中所有经理，和部门编号为20中所有销售员，还有既不是经理又不是销售员但工资大于等于20000的所有员工详细资料

```mysql
SELECT * FROM emp WHERE (deptno = 10 AND job='经理') OR (deptno = 20 AND job='销售员') OR
(job NOT IN('经理','销售员') AND sal >= 20000);
```

7.无奖金或奖金低于1000的员工

```mysql
SELECT * FROM emp WHERE comm IS NULL OR comm < 1000;
```

8.查询名字由三个字的员工

```mysql
SELECT * FROM emp WHERE ename LIKE '___';
```

9.查询所有员工详细信息，用编号升序排序

```mysql
SELECT * FROM emp ORDER BY empno ASC;
```

10.查询所有员工详细信息，用工资降序排序，如果工资相同使用入职日期升序排序

```mysql
SELECT * FROM emp ORDER BY sal DESC, hiredate ASC;
```

11.查询每个部门的平均工资

```mysql
SELECT deptno, AVG(sal) AS 平均工资 FROM emp GROUP BY deptno;
```

12.查询每个部门的雇员数量

```mysql
SELECT deptno, COUNT(*) FROM emp GROUP BY deptno;
```

13.查询每种工作的最高工资，最低工资，人数

```mysql
SELECT job, MAX(sal), MIN(sal), COUNT(*) FROM emp GROUP BY job;
```

---

## MySQL编码问题

1.查看MySQL数据库编码

```mysql
SHOW VARIABLES LIKE 'char%';
```

2.编码解释

* character_set_client：MySQL使用该编码

character_set_client：utf8，无论客户端发送的是什么编码的数据，mysql都当成是utf8的数据：若客户端发送的是GBK，服务器会当成utf8，必然乱码！处理手段：让客户端发送utf8 或 把character_set_client：修改为gbk。

```mysql
SET character_set_client = gbk;//只在当前窗口有效
```

* character_set_result：把数据用什么编码发送给客户端。

若服务器发送给客户端是utf8的数据，客户端会把它当成gbk，在cmd窗口中只显示gbk，必然乱码！处理手段：把character_set_client：修改为gbk。

在my.ini中进行配置，他可以修改client、result、connection。

---

## MySQL备份与恢复数据

数据库 –> sql语句

sql语句 –> 数据库

**1.数据库导出SQL脚本**(备份数据库内容，不是备份数据库)

```mysql
mysqldump -u用户名 -p密码 -hip地址 数据库名>生成sql脚本的路径
```

**2.执行SQL脚本**

```mysql
mysql -u用户名 -p密码 -hip地址 数据库名<脚本文件路径
```

---

## 约束

**1.主键约束**

* 非空性
* 唯一性
* 被引用

当表的某一列被指定为主键后，该列就不能为空，不能有重复出现。

**创建表时指定主键的两种方式：**

```mysql
CREATE TABLE emp(
    empno INT PRIMARY KEY,
    ename VARCHAR(50);
);

INSERT INTO exp VALUES(1,"zhangsan");
```

```mysql
CREATE TABLE exp(
	empno INT,
	ename VARCHAR(50),
	PRIMARY KEY(empno)
);
```

**修改表时指定主键：**

```mysql
ALTER TABLE 表名 ADD PRIMARY KEY(empno); 
```

**删除主键：**

```mysql
ALTER TABLE 表名 DROP PRIMARY KEY;
```

**2.主键自增长**

因为主键列的特性是：必须唯一，不能为空，所以我们通常会指定主键类为整型，然后设置自动增长，这样可以保证在插入数据时主键列的唯一和非空特性。

```mysql
CREATE TABLE t_stu(
	sid INT PRIMARY KEY AUTO_INCREMENT,
	sname VARCHAR(20),
	age INT,
	gender VARCHAR(10)
);

INSERT INTO t_stu VALUES(NULL,'zhangsan',28,'male');
```

**3.非空约束**

因为某些列不能设置为NULL值，所以可以对列添加非空约束。

```mysql
CREATE TABLE stu(
	sid INT PRIMARY KEY AUTO_INCREMENT,
	sname VARCHAR(20) NOT NULL,
	age INT,
	gender VARCHAR(10)
);
```

**4.唯一约束**

某些列不能设置重复的值，所以可以对列添加唯一约束。

```mysql
CREATE TABLE stu(
	sid INT PRIMARY KEY AUTO_INCREMENT,
	sname VARCHAR(20) NOT NULL UNIQUE,
	age INT,
	gender VARCHAR(10)
);
```

**5.概念模型**

实体之间存在着关系，关系有三种：

* 一对多
* 一对一
* 多对多

**6.外键约束**

外键必须是另一表的主键的值(外键要引用主键)，外键可以重复，外键可以为空。一张表中可以有多个外键。

```mysql
CREATE TABLE emp(
	empno INT PRIMARY KEY AUTO_INCREMENT,
    ename VARCHAR(50),
    deptno INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY(deptno) REFERENCES dept(deptno)
);
```

修改表添加外键约束

```mysql
ALTER TABLE emp ADD CONSTRAINT fk_emp_dept FOREIGN KEY(deptno) REFERENCES dept(deptno);
```

 **7.一对一关系**

```mysql
CREATE TABLE husband(
	hid INT PRIMARY KEY AUTO_INCREMENT,
    hname VARCHAR(50)
);

INSERT INTO husband VALUES(NULL,'刘备');
INSERT INTO husband VALUES(NULL,'关羽');
INSERT INTO husband VALUES(NULL,'张飞');

SELECT * FROM husband;

CREATE TABLE wife(
	wid INT PRIMARY KEY AUTO_INCREMENT,
    wname VARCHAR(50),
    CONSTRAINT fk_wife_husband FOREIGN KEY(wid) REFERENCES husband(hid)
);

INSERT INTO wife VALUES(1,'杨贵妃');
INSERT INTO wife VALUES(2,'妲己');
```

**8.多对多关系**

在表中建立多对多关系需要使用中间表，即需要三张表，在中间表中使用两个外键，分别引用其他两个表的主键。

```mysql
CREATE TABLE student(
	sid INT PRIMARY KEY AUTO_INCREMENT,
    sname VARCHAR(50) 
);

CREATE TABLE teacher(
	tid INT PRIMARY KEY AUTO_INCREMENT,
    tname VARCHAR(50)
);

CREATE TABLE stu_tea(
	sid INT,
    tid INT
    CONSTRAINT fk_student FOREIGN KEY(sid) REFERENCES student(sid),
    CONSTRAINT fk_teacher FOREIGN KEY(tid) REFERENCES teacher(tid)
);

INSERT INTO student VALUES(NULL,'刘德华');
INSERT INTO student VALUES(NULL,'梁朝伟');
INSERT INTO student VALUES(NULL,'黄日华');
INSERT INTO student VALUES(NULL,'苗侨伟');
INSERT INTO student VALUES(NULL,'汤朕业');

INSERT INTO teacher VALUES(NULL,'崔老师');
INSERT INTO teacher VALUES(NULL,'刘老师');
INSERT INTO teacher VALUES(NULL,'石老师');

INSERT INTO stu_tea VALUES(1,1);
INSERT INTO stu_tea VALUES(2,1);
INSERT INTO stu_tea VALUES(3,1);
INSERT INTO stu_tea VALUES(4,1);
INSERT INTO stu_tea VALUES(5,1);

INSERT INTO stu_tea VALUES(2,2);
INSERT INTO stu_tea VALUES(3,2);
INSERT INTO stu_tea VALUES(4,2);

INSERT INTO stu_tea VALUES(3,3);
INSERT INTO stu_tea VALUES(4,3);
INSERT INTO stu_tea VALUES(5,3);
```

---

## 多表查询

* 合并结果集
* 连接查询
* 子查询

**合并结果集**

* 要求被合并的表中，列的类型和列数相同
* UNION，去除重复行
* UNION ALL 不去重复

```mysql
CREATE TABLE ab(
	a INT,
    b VARCHAR(50)
);

INSERT INTO ab VALUES(1,'1');
INSERT INTO ab VALUES(2,'2');
INSERT INTO ab VALUES(3,'3');

CREATE TABLE cd(
	c INT,
    d VARCHAR(50)
);

INSERT INTO ab VALUES(3,'3');
INSERT INTO ab VALUES(4,'4');
INSERT INTO ab VALUES(5,'5');

SELECT * FROM ab
UNION ALL
SELECT * FROM cd;

SELECT * FROM ab
UNION
SELECT * FROM cd;
```

**连接查询**

**1.内连接**

```mysql
#笛卡尔积
SELECT * FROM emp, dept;
```

```mysql
#用条件去除笛卡尔积产生的垃圾数据
SELECT * FROM emp, dept WHERE exp.deptno=dept.deptno;
```

```mysql
SELECT emp.ename, emp.sal, dept.dname FROM emp, dept WHERE exp.deptno=dept.deptno;
```

```mysql
SELECT e.ename, e.sal, d.dname FROM emp e, dept d WHERE e.deptno=d.deptno;
```

```mysql
#标准写法
SELECT * FROM 表1 别名1 INNER JOIN 表2 别名2 ON 别名1.xx = 别名2.xx;
```

```mysql
#标准写法
SELECT e.ename, e.sal, d.dname FROM emp e INNER JOIN dept d ON e.deptno=d.deptno;
```

```mysql
SELECT * FROM 表1 别名1 NATURAL JOIN 表2 别名2;
```

**2.外连接**

外连接有一主一次，左外即为左主；即emp为主，那么主表中所有的记录无论满足不满足条件都打印出来。当不满足条件时，右表部分使用null补位。

```mysql
#左外连接
SELECT e.ename, e.sal, d.dname FROM emp e LEFT OUTER JOIN dept d ON e.deptno=d.deptno;
```

```mysql
SELECT e.ename, e.sal, IFNULL(d.dname,'无部门') AS dname FROM emp e LEFT OUTER JOIN dept d ON e.deptno=d.deptno;
```

```mysql
#右外连接
SELECT e.ename, e.sal, d.dname FROM emp e RIGHT OUTER JOIN dept d ON e.deptno=d.deptno;
```

```mysql
#全外连接 = 左外连接+右外连接+合并结果集
SELECT e.ename, e.sal, d.dname FROM emp e LEFT OUTER JOIN dept d ON e.deptno=d.deptno
UNION
SELECT e.ename, e.sal, d.dname FROM emp e RIGHT OUTER JOIN dept d ON e.deptno=d.deptno;
```

---

## 子查询

查询中有查询

```mysql
#查询本公司工资最高的员工的详细信息
SELECT * FROM emp WHERE sal=(SELECT MAX(sal) FROM emp);
```

子查询出现的位置：

* WHERE后作为条件存在
* FROM后作为表存在

条件：

* 单行单列：SELECT * FROM 表1 别名1 WHERE 列1 [=,>,<,>=,<=,!=]\(SELECT 列 FROM 表2 别名2 WHERE 条件)

```mysql
SELECT * FROM emp WHERE sal > (SELECT AVG(sal) FROM emp)
```

* 多行单列：SELECT * FROM 表1 别名1 WHERE 列1 [IN,ALL,ANY]\(SELECT 列 FROM 表2 别名2 WHERE 条件)

```mysql
SELECT * FROM emp WHERE sal > ALL(SELECT sal FROM emp WHERE deptno = 20)
```



* 单行多列：SELECT * FROM 表1 别名1 WHERE (列1,列2) IN(SELECT 列1,列2 FROM 表2 别名2 WHERE 条件)
* 多行多列：SELECT * FROM 表1 别名1 ,(SELECT ….)别名2 WHERE 条件

---

## 多表查询练习

```mysql
#查询至少一个员工的部门，显示部门编号、部门名称、部门位置、部门人数

SELECT d.*, z1.cnt
SELECT * FROM dept d, (SELECT deptno, COUNT(*) cnt FROM EMP GROUP BY deptno) z1 
WHERE d.deptno=z1.deptno
```

```mysql
#列出所有员工的姓名及其直接上级的姓名

SELECT e1.ename, IFNULL(e2.ename,'BOSS') 领导 FROM emp e1 LEFT OUTER JOIN emp e2 ON e1.mgr=e2.empno;
```

```mysql
# 列出受雇日期早于直接上级的所有员工编号，姓名，部门名称

#先不查部门名称，只查部门编号！
SELECT e1.empno, e1.ename, e1.deptno 
FROM emp e1, emp e2
WHERE e1.mgr=e2.empno AND e1.hiredate < e2.hiredate;

SELECT e1.empno, e1.ename, d.dname
FROM emp e1, emp e2, dept d
WHERE e1.mgr=e2.empno AND e1.hiredate < e2.hiredate AND e1.deptno = d.deptno;
```

```mysql
# 列出部门名称和这些部门的员工信息，同时列出那些没有员工的部门

SELECT * 
FROM emp e RIGHT OUTER JOIN dept d
ON e.deptno=d.deptno
```

```mysql
#列出最低薪金大于15000的各种工作及从事此工作的员工人数

SELECT job, COUNT(*) 
FROM emp e 
GROUP BY job
HAVING MIN(sal) > 15000;
```

```mysql
#列出在销售工作的员工的姓名，假定不知道销售部门的部门编号

SELECT e.ename
FROM emp e
WHERE e.deptno=(SELECT deptno FROM dept WHERE dname='销售部')
```

```mysql
#列出薪金高于公司平均薪金的所有员工信息，所在部门名称，上级领导，工资等级

SELECT e.*, d.name, m.ename, s.grade
FROM emp e LEFT OUTER JOIN dept d ON e.deptno=d.deptno
		  LEFT OUTER JOIN emp m ON e.mgr=m.empno
		  LEFT OUTER JOIN salgrade s ON e.sal BETWEEN s.losal AND s.hisal
WHERE e.sal > (SELECT VAG(sal) FROM emp)
```

```mysql
#列出与庞统从事相同工作的所有员工及部门名称

SELECT e.*, d.dname
FROM emp e, dept d
WHERE e.deptno = d.deptno AND job=(SELECT job FROM emp WHERE ename='庞统');
```

```mysql
#列出薪金高于在部门30工作的所有员工的薪金的员工姓名和薪金、部门名称

SELECT e.ename, e.sal, d.dname
FROM emp e, dept d
WHERE e.deptno = d.deptno AND sal > ALL (SELECT sal FROM emp WHERE deptno=30);
```

```mysql
#查出年份、利润、年度增长比
SELECT y1.*, IFNULL(CONCAT((y1.zz-y2.zz)/y2.zz*100,'%'),'0%') 增长率 
FROM tb_year y1 LEFT OUTER JOIN tb_year y2
ON y1.year=y2.year+1;
```



