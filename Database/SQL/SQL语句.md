### SQL SELECT 语句

---

SELECT 语句用于从数据库中选取数据。

---

#### SQL SELECT 语句

SELECT 语句用于从数据库中选取数据。

结果被存储在一个结果表中，称为结果集。

#### SQL SELECT 语法
```sql
SELECT column1, column2, ...
FROM table_name;
```

与
```sql
SELECT * FROM table_name;
```

**参数说明：**

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table_name**：要查询的表名称。

### SQL SELECT DISTINCT 语句

---

SELECT DISTINCT 语句**用于返回唯一不同的值。**

---

#### SQL SELECT DISTINCT 语句

在表中，**一个列可能会包含多个重复值**，有时您也许希望仅仅列出不同（distinct）的值。

DISTINCT 关键词用于返回唯一不同的值。

#### SQL SELECT DISTINCT 语法
```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
```

**参数说明：**

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table_name**：要查询的表名称。

### SQL WHERE 子句

---

WHERE 子句用于过滤记录。

---

#### SQL WHERE 子句

WHERE 子句用于提取那些满足指定条件的记录。

##### SQL WHERE 语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

参数说明：

- **column1, column2, ...**：要选择的字段名称，可以为多个字段。如果不指定字段名称，则会选择所有字段。
- **table_name**：要查询的表名称。

#### 文本字段 vs. 数值字段

SQL **使用单引号来环绕文本值**（大部分数据库系统也接受双引号）。

在上个实例中 'CN' 文本字段使用了单引号。

如果是数值字段，请不要使用引号。

#### WHERE 子句中的运算符

下面的运算符可以在 WHERE 子句中使用：

|运算符|描述|
|---|---|
|=|等于|
|<>|不等于。**注释：** 在 SQL 的一些版本中，该操作符可被写成 !=|
|>|大于|
|<|小于|
|>=|大于等于|
|<=|小于等于|
|BETWEEN|在某个范围内|
|LIKE|搜索某种模式|
|IN|指定针对某个列的多个可能值|


### SQL AND & OR 运算符

---

AND & OR 运算符用于基于一个以上的条件对记录进行过滤。

---

#### SQL AND & OR 运算符

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

### Where 子句

搜索 empno 等于 7900 的数据：

```sql
Select * from emp where empno=7900;
```

#### Where +条件（筛选行）

条件：列，比较运算符，值

比较运算符包涵：= > < >= ,<=, !=,<> 表示（不等于）

```sql
Select * from emp where ename='SMITH';
```

例子中的 SMITH 用单引号引起来，表示是字符串，字符串要区分大小写。

### 逻辑运算

And:与 同时满足两个条件的值。

```sql
Select * from emp where sal > 2000 and sal < 3000;
```

查询 EMP 表中 SAL 列中大于 2000 小于 3000 的值。

Or:或 满足其中一个条件的值

```sql
Select * from emp where sal > 2000 or comm > 500;
```

查询 emp 表中 SAL 大于 2000 或 COMM 大于500的值。

Not:非 满足不包含该条件的值。

```sql
select * from emp where not sal > 1500;
```

查询EMP表中 sal 小于等于 1500 的值。

逻辑运算的优先级：
```sql
()    not        and         or
```

### 特殊条件

**1.空值判断： is null**

```sql
Select * from emp where comm is null;
```

查询 emp 表中 comm 列中的空值。

**2.between and (在 之间的值)**

```sql
Select * from emp where sal between 1500 and 3000;
```

查询 emp 表中 SAL 列中大于 1500 的小于 3000 的值。

注意：大于等于 1500 且小于等于 3000， 1500 为下限，3000 为上限，下限在前，上限在后，查询的范围包涵有上下限的值。

**3.In**

```sql
Select * from emp where sal in (5000,3000,1500);
```

查询 EMP 表 SAL 列中等于 5000，3000，1500 的值。

**4.like**

Like模糊查询

```sql
Select * from emp where ename like 'M%';
```

查询 EMP 表中 Ename 列中有 M 的值，M 为要查询内容中的模糊信息。

- **%** 表示多个字值，**_** 下划线表示一个字符；
- **M%** : 为能配符，正则表达式，表示的意思为模糊查询信息为 M 开头的。
- **%M%** : 表示查询包含M的所有内容。
- **%M_** : 表示查询以M在倒数第二位的所有内容。

### SQL ORDER BY 关键字

---

ORDER BY **关键字用于对结果集进行排序。**

---

#### SQL ORDER BY 关键字

ORDER BY 关键字用于**对结果集按照一个列或者多个列进行排序。**

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。

#### SQL ORDER BY 语法
```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column1, column2, ... ASC|DESC;
```

- **column1, column2, ...**：要排序的字段名称，可以为多个字段。
- **ASC**：表示按升序排序。
- **DESC**：表示按降序排序。

### SQL INSERT INTO 语句

---

INSERT INTO 语句用于向表中插入新记录。

---

#### SQL INSERT INTO 语句

INSERT INTO 语句用于向表中插入新记录。

#### SQL INSERT INTO 语法

INSERT INTO 语句可以有两种编写形式。

- 第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：

```sql
INSERT INTO table_name
VALUES (value1,value2,value3,...);
```

- - - - - - - - 第二种形式需要指定列名及被插入的值：

```sql
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
```

**参数说明：**

- **table_name**：需要插入新记录的表名。
- **column1, column2, ...**：需要插入的字段名。
- **value1, value2, ...**：需要插入的字段值。

### SQL UPDATE 语句

---

UPDATE 语句用于更新表中的记录。

---

#### SQL UPDATE 语句

UPDATE 语句用于更新表中已存在的记录。

##### SQL UPDATE 语法
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

参数说明：

- **table_name**：要修改的表名称。
- **column1, column2, ...**：要修改的字段名称，可以为多个字段。
- **value1, value2, ...**：要修改的值，可以为多个值。
- **condition**：修改条件，用于指定哪些数据要修改。

|   |   |
|---|---|
|![lamp](https://www.runoob.com/images/lamp.jpg)|**请注意 SQL UPDATE 语句中的 WHERE 子句！**  <br>WHERE 子句规定哪条记录或者哪些记录需要更新。如果您省略了 WHERE 子句，所有的记录都将被更新！|

# SQL DELETE 语句

---

DELETE 语句用于删除表中的记录。

---

### SQL DELETE 语句

DELETE 语句用于删除表中的行。

#### SQL DELETE 语法

```sql
DELETE FROM table_name
WHERE condition;
```

参数说明：

- **table_name**：要删除的表名称。
- **condition**：删除条件，用于指定哪些数据要删除。

  

|   |   |
|---|---|
|![lamp](https://www.runoob.com/images/lamp.jpg)|**请注意 SQL DELETE 语句中的 WHERE 子句！**  <br>WHERE 子句规定哪条记录或者哪些记录需要删除。如果您省略了 WHERE 子句，所有的记录都将被删除！|