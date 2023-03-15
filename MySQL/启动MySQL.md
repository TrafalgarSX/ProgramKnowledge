### Linux下如何启动MySQL
初始化 MySQL
`mysqld --initialize`
启动MySQL
`systemctl start mysqld`
查看MySQL运行状态
`systemctl status mysqld`

### Windows下如何启动MySQL
`mysqld --initialize`
启动mysql 服务`net start mysql`
```
可能会出现
net start mysql
服务名无效。

这是因为系统中并没有注册mysql到服务中，那如何注册到win服务中呢？
mysqld --install 
Service successfully install 代表已经安装成功

net start mysql

如果出现服务无法启动情况：
删除mysql下的data文件，重新执行mysqld --initialize 就可在当前路径下生成data文件夹，
再执行net start msyql
```
关闭mysql服务 `net stop mysql`

### MySQL管理
#### Windows系统下
启动：
`mysqld --console` 一种方式，上面是另一种方式
关闭
`mysqladmin -uroot shutdown`

#### Linux系统下
检查MySQL服务器是否启动：
`ps -ef | grep mysqld`
启动：
`./mysqld_safe &`
`service mysql restart`
关闭
`./mysqladmin -u root -p shutdown`

### MySQL安装后需要做的
MySQL安装成功后，默认的root用户密码为空，你可以使用以下命令来创建root用户的密码：
`mysqladmin -u root password "new_password"`

#### Windows下的配置
在安装目录下，创建my.ini配置文件
配置以下基本信息：
```
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=your install path
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
# datadir=your data path
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

### 如何运行MySQL客户端
`mysql -hlocalhost -uroot -proot`
-h 表示服务器域名  -u为数据库用户名  -p为密码

show database;
use database;
show tables

### 修改MySQL密码
`mysqladmin -u root -p password new_password`