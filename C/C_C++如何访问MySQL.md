### C/C++ 如何访问数据库
```
g++ MyDB.cpp test.cpp  -o test    `mysql_config --cflags --libs `
```
`mysql_config --cflags --libs `编译时需要带上这个才能编译通过

##### Linux如何连接？
```bash
apt install mysql-server mysql-client libmysqlclient-dev

C/C++代码中 #include<mysql/mysql.h>就可以连接数据库进行数据的增删改查了。
```

操纵数据库所用到的函数：
| 函数               | 解析                                |
| ------------------ | ----------------------------------- |
| mysql_init         | 初始化数据库句柄                    |
| mysql_options      | 设置字符编码                        |
| mysql_real_connect | 连接数据库                          |
| mysql_select_db    | 选择数据库                          |
| mysql_real_query   | 执行sql语句，成功返回0，失败返回非0 |
| mysql_store_result | 获取查询的结果集                    |
| mysql_fetch_row    | 获取查询到的一行数据                |
| mysql_free_result  | 释放结果集                          |
| mysql_close        | 关闭数据库                          |
| mysql_error        | 获取数据库当前操作失败的原因        |

操纵数据库所用到的变量类型：
| 类型      | 解析             |
| --------- | ---------------- |
| MYSQL     | 数据库句柄       |
| MYSQL_RES | 查询结果集       |
| MYSQL_ROW | 记录结果集结构体 |

#### 连接数据库和选择数据库
```c++
bool connectDB(MYSQL &mysql) {
	// 1.初始化数据库句柄
	mysql_init(&mysql);
	
	// 2.设置字符编码
	mysql_options(&mysql, MYSQL_SET_CHARSET_NAME, "gbk");

	// 3.连接数据库	              
	MYSQL *ret = mysql_real_connect(&mysql, "127.0.0.1", "guoyawen", "password", "test", 3306, NULL, 0);
	if (ret == NULL) {
		printf("数据库连接失败！失败原因：%s\n", mysql_error(&mysql));
		return false;
	}

	printf("数据连接成功！\n");

	// 选择数据库，成功返回0，失败返回非0
	int res = mysql_select_db(&mysql, "test");
	if (res) {
		printf("选择数据库失败！失败原因%s\n", mysql_error(&mysql));
		return false;
	}

	printf("数据库选择成功！\n");

	return true;
}

```

#### 增
```c++
bool addTableData(int id, char *name, int age, char *sex) {
	MYSQL mysql;			// 数据库句柄
	char sql[SQL_MAX];		// 存储sql语句

		// 连接数据库
	if (!connectDB(mysql)) {
		return false;
	}

	// C语言字符串组合
	snprintf(sql, SQL_MAX, "INSERT INTO student VALUES(%d, '%s', %d, '%s');", id, name, age, sex);
	printf("插入sql语句：%s\n", sql);
	// 如果id字段用了auto_increment进行修饰，那么可以使用以下数据库语句：
	// snprintf(sql, SQL_MAX, "INSERT INTO student(name, age, sex) VALUES('%s', %d, '%s');", name, age, sex);

	// 执行sql语句，成功返回0
	//int ret = mysql_query(&mysql, sql);		
	int ret = mysql_real_query(&mysql, sql, (unsigned long)strlen(sql));		
	printf("执行插入语句，插入返回结果：%d\n", ret);

	if (ret) {
		printf("插入表数据失败！失败原因：%s\n", mysql_error(&mysql));
		return false;
	}
	printf("插入表数据成功！\n");

	// 关闭数据库
	mysql_close(&mysql);

	return true;
}

```
#### 删
```c++
bool delTableData(int id) {
	MYSQL mysql;			// 数据库句柄
	char sql[SQL_MAX];		// 存储sql语句

		// 连接数据库
	if (!connectDB(mysql)) {
		return false;
	}

	// C语言字符串组合
	snprintf(sql, SQL_MAX, "DELETE FROM student WHERE id = %d;", id);
	printf("删除sql语句：%s\n", sql);

	// 执行sql语句，成功返回0
	//int ret = mysql_query(&mysql, sql);
	int ret = mysql_real_query(&mysql, sql, (unsigned long)strlen(sql));
	printf("执行删除语句，插入返回结果：%d\n", ret);

	if (ret) {
		printf("删除表数据失败！失败原因：%s\n", mysql_error(&mysql));
		return false;

	}
	printf("删除表数据成功！\n");

	// 关闭数据库
	mysql_close(&mysql);

	return true;
}

```
#### 查
```c++
bool queTableData() {
	MYSQL mysql;		// 数据库句柄
	MYSQL_RES* res;		// 查询结果集
	MYSQL_ROW row;		// 记录结构体
	char sql[SQL_MAX];	// SQL语句

	// 连接数据库
	if (!connectDB(mysql)) {
		return false;
	}

	// C语言组合字符串
	snprintf(sql, SQL_MAX, "SELECT id, name, age, sex FROM student;");
	printf("查询sql语句：%s\n", sql);

	// 查询数据
	//int ret = mysql_query(&mysql, "select * from student;");		// 等效于下面一行代码
	//int ret = mysql_query(&mysql, sql);
	int ret = mysql_real_query(&mysql, sql, (unsigned long)strlen(sql));
	printf("执行查询语句，查询返回结果：%d\n", ret);

	if (ret) {
		printf("数据查询失败！失败原因：%s\n", mysql_error(&mysql));
		return false;
	}
	printf("数据查询成功！\n");
	


	// 获取结果集
	res = mysql_store_result(&mysql);

	// 获取查询到的一行数据
	// 给row赋值，判断row是否为空，不为空就打印数据。
	while (row = mysql_fetch_row(res)) {
		printf("%d  ", atoi(row[0]));	// 转换为int类型，打印id
		printf("%s  ", row[1]);			// 打印姓名
		printf("%d  ", atoi(row[2]));	// 转换为int类型，打印年龄
		printf("%s  \n", row[3]);		// 打印性别
	}

	// 释放结果集
	mysql_free_result(res);

	// 关闭数据库
	mysql_close(&mysql);

	return true;
}

```
#### 改
```c++
bool altTableData(int id, int age) {
	MYSQL mysql;		// 数据库句柄
	char sql[SQL_MAX];	// sql语句

	// 连接数据库
	if (!connectDB(mysql)) {
		return false;
	}

	// C语言组合字符串
	snprintf(sql, SQL_MAX, "UPDATE student SET age = %d WHERE id = %d;", age, id);
	printf("修改sql语句：%s\n", sql);

	// 执行sql语句，成功返回0
	//int ret = mysql_query(&mysql, sql);
	int ret = mysql_real_query(&mysql, sql, (unsigned long)strlen(sql));
	printf("执行修改语句，修改返回结果：%d\n", ret);

	if (ret) {
		printf("数据修改失败！失败原因：%s\n", mysql_error(&mysql));
		return false;
	}

	printf("修改表数据成功！\n");

	// 关闭数据库
	mysql_close(&mysql);

	return true;
}

```

[C/C++ vs2017连接MySQL数据库 - 增删改查(详细步骤)\_vs连接mysql数据库增删改查代码\_cpp\_learners的博客-CSDN博客](https://blog.csdn.net/cpp_learner/article/details/116171955)