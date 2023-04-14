### Windwos下如何设置远程访问
MySQL默认不允许远程登录
```mysql
use mysql; # 进入mysql database

查资料得知mysql8的分配权限不能带密码隐士创建账号了，要先创建账号再设置权限,以前可以一条命令创建账号并分配权限

# 创建一个用户 guoyawen, % 为允许所有ip访问 guo150246是密码。
create user 'guoyawen'@'%' identified by 'guo150246';
# 为guoyawen 授权， with grant option 是允许被授权的用户给其他用户授权
grant all privileges on *.* to 'guoyawen'@'%' with grant option;
all代表所有权限， 还可以使用 select, insert, update,delete
授权也可以使用
update user set host = '%' where user = 'root'

# 刷新权限
flush privileges;
这句表示从mysql数据库的grant表中重新加载权限数据
因为MySQL把权限都放在了cache中，所以在做完更改后需要重新加载。

# 查看user表中的用户和对应的权限
select User,authentication_string,Host from user;

# 取消操作权限
REVOKE select, insert ON what FROM root
```

### Linux下如何设置远程访问
1. 首先需要改变MySQL的配置，执行：
`vim /etc/mysql/mysql.conf.d/mysqld.cnf`
2. 找到bind-address=127.0.0.1并注释掉
3. 执行`service mysql restart`重启MySQL服务
4. 开启MySQL远程访问
	1. 首先登录MySQL
	2. `grant all privileges on *.* to 'guoyawen'@'%' with grant option;`
	3. 执行`flush privileges;`
