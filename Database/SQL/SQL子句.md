### SQL SELECT TOP, LIMIT, ROWNUM 子句

---

#### SQL SELECT TOP 子句

SELECT TOP 子句用于**规定要返回的记录的数目。**

SELECT TOP 子句对于拥有数千条记录的大型表来说，是非常有用的。

> **注意:**并非所有的数据库系统都支持 SELECT TOP 语句。 MySQL 支持 LIMIT 语句来选取指定的条数数据， Oracle 可以使用 ROWNUM 来选取。

##### SQL Server / MS Access 语法

```sql
SELECT TOP number|percent column_name(s)
FROM table_name;
```
##### MySQL 语法
```sql
SELECT column_name(s)
FROM table_name
LIMIT number;
```
##### 实例
```sql
SELECT *
FROM Persons
LIMIT 5;
```
##### Oracle 语法
```sql
SELECT column_name(s)
FROM table_name
WHERE ROWNUM <= number;
```
##### 实例
```sql
SELECT *
FROM Persons
WHERE ROWNUM <=5;
```

### SQL LIKE 操作符

---

LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

---

#### SQL LIKE 操作符

LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

##### SQL LIKE 语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column LIKE pattern;
```

参数说明：

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table_name**：要查询的表名称。
- **column**：要搜索的字段名称。
- **pattern**：搜索模式。

```sql
SELECT * FROM Websites
WHERE name LIKE '%k'; 
```

### SQL 通配符

---

**通配符可用于替代字符串中的任何其他字符。**

---

#### SQL 通配符

在 SQL 中，**通配符与 SQL LIKE 操作符一起使用。**(pattern部分)

SQL 通配符用于搜索表中的数据。

在 SQL 中，可使用以下通配符：

|通配符|描述|
|---|---|
|%|替代 0 个或多个字符|
|_|替代一个字符|
|[_charlist_]|字符列中的任何单一字符|
|`[^charlist]`  <br>或  <br>[!_charlist_]|不在字符列中的任何单一字符|

```sql
选取 url 以字母 "https" 开始的所有网站
SELECT * FROM Websites  
WHERE url LIKE 'https%';

以一个任意字符开始，然后是 "oogle" 的所有客户
SELECT * FROM Websites  
WHERE name LIKE '_oogle';

MySQL 中使用 **REGEXP** 或 **NOT REGEXP** 运算符 (或 RLIKE 和 NOT RLIKE) 来操作正则表达式。

下面的 SQL 语句选取 name 以 "G"、"F" 或 "s" 开始的所有网站：
SELECT * FROM Websites
WHERE name REGEXP '^[GFs]'; 
```

### SQL IN 操作符

---

#### IN 操作符

IN 操作符允许您在 WHERE 子句中规定多个值。

##### SQL IN 语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column IN (value1, value2, ...);
```

参数说明：

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table_name**：要查询的表名称。
- **column**：要查询的字段名称。
- **value1, value2, ...**：要查询的值，可以为多个值。

```sql
SELECT * FROM Websites  
WHERE name IN ('Google','菜鸟教程');
```

### SQL BETWEEN 操作符

---

BETWEEN 操作符用于**选取介于两个值之间的数据范围内的值。**

---

#### SQL BETWEEN 操作符

BETWEEN 操作符选取介于两个值之间的数据范围内的值。这些值可以是数值、文本或者日期。

##### SQL BETWEEN 语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column BETWEEN value1 AND value2;
```

参数说明：

- column1, column2, ...：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- table_name：要查询的表名称。
- column：要查询的字段名称。
- value1：范围的起始值。
- value2：范围的结束值。

```sql
SELECT * FROM Websites  
WHERE alexa BETWEEN 1 AND 20;

SELECT * FROM Websites
WHERE alexa NOT BETWEEN 1 AND 20; 
```

|   |   |
|---|---|
|![lamp](https://www.runoob.com/images/lamp.jpg)|**请注意，在不同的数据库中，BETWEEN 操作符会产生不同的结果！**  <br> 在某些数据库中，BETWEEN 选取介于两个值之间但不包括两个测试值的字段。  <br>在某些数据库中，BETWEEN 选取介于两个值之间且包括两个测试值的字段。  <br>在某些数据库中，BETWEEN 选取介于两个值之间且包括第一个测试值但不包括最后一个测试值的字段。<br><br> **因此，请检查您的数据库是如何处理 BETWEEN 操作符！**|

### SQL 别名

---

通过使用 SQL，可以**为表名称或列名称指定别名。**

---

#### SQL 别名

通过使用 SQL，可以为表名称或列名称指定别名。

基本上，**创建别名是为了让列名称的可读性更强。**

##### 列的 SQL 别名语法

```sql
SELECT column_name AS alias_name
FROM table_name;
```

##### 表的 SQL 别名语法
```sql
SELECT column_name(s)
FROM table_name AS alias_name;
```

demo:
```sql
// 别名
SELECT w.name, w.url, a.count, a.date
FROM Websites AS w, access_log AS a
WHERE a.site_id=w.id and w.name="菜鸟教程"; 
// 没有别名
SELECT Websites.name, Websites.url, access_log.count, access_log.date
FROM Websites, access_log
WHERE Websites.id=access_log.site_id and Websites.name="菜鸟教程";
```

💡在下面的情况下，使用别名很有用：

- 在查询中涉及**超过一个表**
- 在查询中**使用了函数**
- 列名称很长或者可读性差
- 需要把两个列或者多个列结合在一起

### SQL 连接(JOIN)

---

SQL join 用于把来自两个或多个表的行结合起来。

下图展示了 LEFT JOIN、RIGHT JOIN、INNER JOIN、OUTER JOIN 相关的 7 种用法。

![[SQL_JOIN.png]]

### SQL JOIN

SQL JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。

最常见的 JOIN 类型：**SQL INNER JOIN（简单的 JOIN）**。 SQL INNER JOIN 从多个表中返回满足 JOIN 条件的所有行。

#### 语法：
```sql
SELECT column1, column2, ...
FROM table1
JOIN table2 ON condition;
```

**参数说明：**

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table1**：要连接的第一个表。
- **table2**：要连接的第二个表。
- **condition**：连接条件，用于指定连接方式。

```sql
SELECT Websites.id, Websites.name, access_log.count, access_log.date  
FROM Websites  
INNER JOIN access_log  
ON Websites.id=access_log.site_id;
```

#### 不同的 SQL JOIN

在我们继续讲解实例之前，我们先列出您可以使用的不同的 SQL JOIN 类型：

- **INNER JOIN**：如果**表中有至少一个匹配，则返回行**
- **LEFT JOIN**：即使右表中没有匹配，也从左表返回所有的行, **无论如何左表的行都会被返回**
- **RIGHT JOIN**：即使左表中没有匹配，也从右表返回所有的行， **无论如何左表的行都会被返回**
- **FULL JOIN**：**只要其中一个表中存在匹配**，则返回行

### SQL INNER JOIN 关键字

---

#### SQL INNER JOIN 关键字

INNER JOIN 关键字在表中存在至少一个匹配时返回行。

##### SQL INNER JOIN 语法
```sql
SELECT column_name(s)  
FROM table1
INNER JOIN table2  
ON table1.column_name=table2.column_name;
```

或：
```sql
SELECT column_name(s)  
FROM table1  
JOIN table2  
ON table1.column_name=table2.column_name;
```

**参数说明：**

- columns：要显示的列名。
- table1：表1的名称。
- table2：表2的名称。
- column_name：表中用于连接的列名。

💡**注释：** INNER JOIN 与 JOIN 是相同的。

![[INNER_JOIN.png]]

### SQL LEFT JOIN 关键字

---

#### SQL LEFT JOIN 关键字

LEFT JOIN 关键字**从左表（table1）返回所有的行**，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。

##### SQL LEFT JOIN 语法
```sql
SELECT column_name(s)
FROM table1  
LEFT JOIN table2
ON table1.column_name=table2.column_name;
```

或：
```sql
SELECT column_name(s)  
FROM table1  
LEFT OUTER JOIN table2  
ON table1.column_name=table2.column_name;
```

**注释：** 在某些数据库中，LEFT JOIN 称为 LEFT OUTER JOIN。

![[LEFT_JOIN.png]]

![[LEFT_JOIN_RESULT.png]]

### SQL RIGHT JOIN 关键字

---

#### SQL RIGHT JOIN 关键字

RIGHT JOIN 关键字**从右表（table2）返回所有的行，即使左表（table1）中没有匹配**。如果左表中没有匹配，则结果为 NULL。

##### SQL RIGHT JOIN 语法
```sql
SELECT column_name(s)  
FROM table1  
RIGHT JOIN table2  
ON table1.column_name=table2.column_name;
```

或：
```sql
SELECT column_name(s)  
FROM table1  
RIGHT OUTER JOIN table2  
ON table1.column_name=table2.column_name;
```

**注释：** 在某些数据库中，RIGHT JOIN 称为 RIGHT OUTER JOIN。
![[RIGHT_JOIN.png]]
![[RIGHT_JOIN_DEMO.png]]

# SQL FULL OUTER JOIN 关键字

---

#### SQL FULL OUTER JOIN 关键字

FULL OUTER JOIN **关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.**

FULL OUTER JOIN 关键字结合了 LEFT JOIN 和 RIGHT JOIN 的结果。

##### SQL FULL OUTER JOIN 语法
```sql
SELECT column_name(s)  
FROM table1  
FULL OUTER JOIN table2  
ON table1.column_name=table2.column_name;
```

1. A inner join B 取交集。

2. A left join B 取 A 全部，B 没有对应的值为 null。

3. A right join B 取 B 全部 A 没有对应的值为 null。

4. A full outer join B 取并集，彼此没有对应的值为 null。

# SQL UNION 操作符

---

SQL UNION 操作符合并两个或多个 SELECT 语句的结果。

---

## SQL UNION 操作符

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

### SQL UNION 语法
```sql
SELECT column_name(s) FROM table1  
UNION  
SELECT column_name(s) FROM table2;
```

**注释：** 默认地，UNION **操作符选取不同的值**。如果允许重复的值，请使用 UNION ALL。

##### SQL UNION ALL 语法
```sql
SELECT column_name(s) FROM table1  
UNION ALL  
SELECT column_name(s) FROM table2;
```

**注释：** UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。

[SQL UNION 操作符 | 菜鸟教程](https://www.runoob.com/sql/sql-union.html)

### SQL SELECT INTO 语句

---

通过 SQL，您可以**从一个表复制信息到另一个表。**

SELECT INTO 语句从一个表复制数据，然后把数据插入到另一个新表中。

---

#### SQL SELECT INTO 语句

SELECT INTO 语句从一个表复制数据，然后把数据插入到另一个新表中。

> **注意：**
> 
> MySQL 数据库不支持 SELECT ... INTO 语句，但支持 [INSERT INTO ... SELECT](https://www.runoob.com/sql/sql-insert-into-select.html) 。
> 
> 当然你可以使用以下语句来拷贝表结构及数据：
> 
> CREATE TABLE 新表
> AS
> SELECT * FROM 旧表 

##### SQL SELECT INTO 语法

我们可以复制所有的列插入到新表中：
```sql
SELECT *  
INTO newtable [IN externaldb]  
FROM table1;
```

或者只复制希望的列插入到新表中：
```sql
SELECT column_name(s)  
INTO newtable [IN externaldb]  
FROM table1;
```

|   |   |
|---|---|
|![lamp](https://www.runoob.com/images/lamp.jpg)|**提示：**新表将会使用 SELECT 语句中定义的列名称和类型进行创建。您可以使用 AS 子句来应用新名称。|

### SQL INSERT INTO SELECT 语句

---

通过 SQL，您可以从一个表复制信息到另一个表。

INSERT INTO SELECT 语句从一个表复制数据，然后把数据插入到一个已存在的表中。

---

#### SQL INSERT INTO SELECT 语句

INSERT INTO SELECT 语句从一个表复制数据，然后把数据**插入到一个已存在的表中**。目标表中任何已存在的行都不会受影响。

##### SQL INSERT INTO SELECT 语法

我们可以从一个表中复制所有的列插入到另一个已存在的表中：
```sql
INSERT INTO table2  
SELECT * FROM table1;  
```

或者我们可以只复制指定的列插入到另一个已存在的表中：
```sql
INSERT INTO table2  
(column_name(s))  
SELECT column_name(s)  
FROM table1;
```

**select into from 和 insert into select 都是用来复制表**

两者的主要区别为： **select into from** 要求**目标表不存在**，因为在插入时会自动创建；**insert into select from** 要求目标表存在。

1. 复制表结构及其数据：
```sql
create table table_name_new as select * from table_name_old
```

2. 只复制表结构：
```sql
create table table_name_new as select * from table_name_old where 1=2;
```

或者：
```sql
create table table_name_new like table_name_old
```

3. 只复制表数据：

如果两个表结构一样：
```sql
insert into table_name_new select * from table_name_old
```

如果两个表结构不一样：
```sql
insert into table_name_new(column1,column2...) select column1,column2... from table_name_old
```


### SQL 约束（Constraints）

---

#### SQL 约束（Constraints）

SQL 约束用于规定表中的数据规则。

**如果存在违反约束的数据行为，行为会被约束终止。**

约束可以在创建表时规定（**通过 CREATE TABLE 语句**），或者在表创建之后规定（**通过 ALTER TABLE 语句**）。

##### SQL CREATE TABLE + CONSTRAINT 语法
```sql
CREATE TABLE table_name  
(  
column_name1 data_type(size) constraint_name,  
column_name2 data_type(size) constraint_name,  
column_name3 data_type(size) constraint_name,  
....  
);
```

在 SQL 中，我们有如下约束：

- **NOT NULL** - 指示某列不能存储 NULL 值。
- **UNIQUE** - 保证某列的每行必须有唯一的值。
- **PRIMARY KEY** - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
- **FOREIGN KEY** - 保证一个表中的数据匹配另一个表中的值的参照完整性。
- **CHECK** - 保证列中的值符合指定的条件。
- **DEFAULT** - 规定没有给列赋值时的默认值。

在下面的章节，我们会详细讲解每一种约束。

```sql
create table if not exists per(
  id bigint auto_increment comment '主键',
  name varchar(20) not null comment '人员姓名',
  work_id bigint not null comment '工作id',
  create_time date default '2021-04-02',
  primary key(id),
  foreign key(work_id) references work(id)
)

create table if not exists work(
  id bigint auto_increment comment '主键',
  name varchar(20) not null comment '工作名称',
  create_time date default '2021-04-02',
  primary key(id)
)
```

