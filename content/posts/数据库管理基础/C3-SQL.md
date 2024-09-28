---
title: "C3-SQL"
date: 2024-04-22T19:18:04-08:00
categories: 
- "数据库管理基础"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "`CREATE SCHEMA AUTHORIZATION ` `DROP SCHEMA ` ```sql CREATE TABLE ( [] [,同上] [,]) ``` ```sql e.g. CR..."
---

# 第三章-关系数据库标准语言SQL

## 数据定义

### 创建模式

`CREATE SCHEMA <模式名> AUTHORIZATION <用户名>`

- 命令后可接受
    - `CREATE TABLE`
    - `CREATE VIEW`
    - `GRANT`

- 执行创建模式语句必须有DBA权限

### 删除模式

`DROP SCHEMA <模式名> <CASCADE|RESTRICT>`

- CASCADE 级联：删除模式同时删除模式内所有数据库对象
- RESTRICT 限制：只在模式下无任何下属对象才删除，否则拒绝执行

### 创建基本表

```sql
CREATE TABLE <表名>
(<列名> <数据类型> [<列级完整性约束条件>]
[,同上]
[,<表级完整性约束条件>])
```

```sql
e.g.
CREATE TABLE Course
(Cno CHAR(4) PRIMARY KEY,
 Cname CHAR(40) UNIQUE,
 Cpno CHAR(4),
 FOREIGN KEY(Cpno) REFERENCES Course(Cno)
);
```

### 修改基本表

```sql
ALTER TABLE <表名>
[ADD[COLUMN] <列名> <数据类型> [完整性约束]]
[ADD <表级完整性约束>]
[DROP [COLUMN] <列名> [CASCADE|RESTRICT]]
[DROP CONSTRAINT<完整性约束名>[CASCADE|RESTRICT]]
[ALTER COLUMN <列名><数据类型>];
```

### 删除基本表

`DROP TABLE <表名> [CASCADE|RESTRICT]`

### 建立索引

- DBMS一般会自动建立PRIMARY KEY和UNIQUE列上的索引

`CREATE [UNIQUE] [CLUSTER] INDEX<索引名> ON <表名> (<列名>[<次序>],[同前]···) `

```sql
e.g.
CREATE UNIQUE INDEX SCno ON SC(Sno ASC, Cno DESC);
```

- ASC-升序	DESC-降序

- 有关聚簇索引CLUSTER
    - 在最经常查询的列上建立CLUSTER可以提高查询效率
    - 一个基本表只能建立一个聚簇索引
    - 经常更新的列不宜建立聚簇索引

## 数据查询

```sql
SELECT [ALL|DISTINCT] <目标列表达式> [,<目标列表达式>]···
FROM <表|视图> [,<表|视图>]···
	[WHERE<条件表达式>]
	[GROUP BY<列名>[HAVING<条件表达式>]]
	[ORDER BY<列名>[ASC|DESC]];
```

- DISTINCT-消除取值重复的行	ALL-缺省

- 目标列表达式 包括

    - 算数表达式`2024-Sage`
    - 字符串常量`'Year of Birth'`

    	- 函数`LOWER(Sdept)`

    	- 列别名`BIRTH 2024-Sage`

- WHERE查询满足条件的元组

    - | 查询条件 | 谓词                           |
        | -------- | ------------------------------ |
        | 比较     | =,>,<,>=,<=,!=,<>,!>,!<        |
        | 确定范围 | BETWEEN value1 AND value2      |
        | 确定集合 | IN (element1, element2, ...)   |
        | 字符匹配 | LIKE 'pattern' ESCAPE '转义符' |
        | 空值     | IS NULL                        |
        | 多重条件 | AND, OR, NOT                   |

- ORDER BY中，空值默认为最大值
- **聚集函数**
    - `COUNT SUM AVG MAX MIN`
    - `聚集函数 ([DISTINCT|ALL] <列名>)`
    - `COUNT ([DISTINCT|ALL] *)`

- GROUP BY作用
    - 细化聚聚函数。分组后，聚集函数将分别作用于每个组

- WHERE与HAVING
    - WHERE作用于基本表或视图，从中选择满足表达式的元组。HAVING作用于组，从中选择满足条件的组。
    - WHERE不能用聚集函数

### 连接查询

#### 等值连接和非等值连接

- WHERE中连接两个表的条件格式
    - `[<表名1>]<列名1> <比较运算符> [<表名2>]<列名2>`
    - `[<表名1>]<列名1> BETWEEN [<表名2>]<列名2> AND [<表名3>]<列名3>`

- 如果比较运算符是`=`,这个连接是等值连接。若在等值连接中把目标列中重复的属性列删掉则为自然连接

#### 自身连接

- 一个表和自己连接。必须给表起别名，且属性前有表别名前缀。

```sql
e.g.
SELECT FIRST.Cno, SECOND.Cpno
FROM Course FIRST, Course SECOND
WHERE FIRST.Cpno == SECOND.Cno;
```

#### 外连接

- `LEFT|RIGHT OUT JOIN <表名> ON`

```sql
e.g.
SELECT Student.Sno, Sname, Ssex, Sage, Sdept, Cno
FROM Student LEFT OUT JOIN SC ON (Student.Sname=Cno);
```

#### 多表连接

```sql
e.g.
SELECT Student.Sno, Sname, Cname, Grade
FROM Student, SC, Course
WHERE Student.Sno = SC.Sno AND SC.Cno = Course.Cno;
```

### 嵌套查询

- 将一个查询块嵌套在另外一个查询块的WHERE或HAVING里面
    - 查询块：一个SELECT-FROM-WHERE语句
- 子查询不能用ORDER BY字句

- 相关子查询/不相关子查询: 是否需要用到父查询的信息

- `ALL`和`ANY`量词

    - 用在比较符号后面，集合或者查询结果前面

    - ```sql
        e.g.
        WHERE Sage<ALL
        	(SELECT Sage FROM Student WHERE Sdept='CS')
        ```

- `EXIST`量词

    - ```sql
        e.g.
        SELECT Sname FROM Student
        WHERE EXISTS
        	(SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1')
        ```

    - 遍历Student每一列，如果这一列能让子查询结果非空，EXISTS就会返回真，这一列也会加入查询结果里去。
    - `EXIST`后面子查询都用`*`，因为子查询只返回是否非空，给列名没啥用
    - `EXIST`前面可加`NOT`

- 量词间的替换

    - IN、比较运算符、ANY、ALL都可以被EXIST等价替换
    - 反过来不一定行
