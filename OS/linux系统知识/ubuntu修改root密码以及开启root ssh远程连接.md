**搞这个是因为 windows 的 ssh 服务折腾了好久还是不能用， 所以就用 wsl 的 ssh 服务曲线救国，最终搞成功了。**

**这样就可以在同一个局域网下，一个主机  `git clone` 另一台机器的 git 仓库。**
# 1. ubuntu root 默认密码

ubuntu 默认的 root 用户是没有固定密码的，它的密码是随机产生并且动态改变的，即每次开机都有一个新的 root 密码，所以拿到一台新的Ubuntu系统服务器后，我们需要设置一个固定的root密码。

**修改root用户密码**

```shell
sudo passwd
```

然后它会提示你输入安装操作系统时初始用户的密码，输入完之后，enter，终端会提示我们输入新的密码并确认，此时的密码就是root新密码。

[[创建或者修改root用户密码]]

## 2. 开启root ssh远程连接

note: **普通的 ssh 服务开启后， 可以通过 ssh 访问普通用户， 不能访问 root 用户，下面的步骤可以设置 sssh 访问 root 用户。**

检查Ubuntu是否已安装ssh服务

```shell
sudo service sshd status
```

若提示错误，则说明还没安装，需要先安装ssh服务

```shell
sudo apt-get install openssh-server
```

再开启ssh服务

```shell
sudo service sshd start
```

> 另外，若虚拟机没办法被ssh远程工具连接上，也需要排查是否安装了ssh服务，再排查网络端口问题

编辑配置文件

```shell
sudo vim /etc/ssh/sshd_config
```

![[Pasted image 20240624005509.png]]

将
```shell
PermitRootLogin prohibit-password
```

更改为：

```shell
PermitRootLogin yes
```

如果`PermitRootLogin prohibit-password`被注释，则取消注释并更改为`PermitRootLogin yes`

修改完成后，重启ssh服务

```shell
sudo systemctl restart sshd
```

**再次使用root远程登录，即可成功**

## 3. 开启防火墙

Ubuntu使用的防火墙名为UFW(Uncomplicated Fire Wall)，是一个iptable的管理工具。因为iptable是根据系统管理员编写的一系列规则筛选网络数据包，比较复杂，所以UFW对其进行了简化。

#### 查看本地端口开启情况

```shell
sudo ufw status
```

#### 关闭防火墙

```shell
sudo ufw disable
```

#### 开启防火墙，允许访问特定端口

```shell
sudo ufw enable
```

#### 简单开启/禁用

```shell
sudo ufw allow|deny [service]
sudo ufw allow smtp #允许所有的外部IP访问本机的25/tcp端口(smtp)
sudo ufw deny smtp #禁止外部访问smtp服务
sudo ufw allow 22/tcp #允许所有的外部IP访问本机的22/tcp端口(ssh)
sudo ufw allow 53 #允许外部访问53端口(tcp/udp)
sudo ufw allow from 192.168.1.100 #允许此IP访问所有的本机端口
sudo ufw allow proto tcp from 192.168.1.30 to 192.1681.5 port 65000 #允许从192.168.1.30到192.1681.5的SSH连接删除规则
sudo ufw delete [rule]
sudo ufw delete allow smtp #删除某条规则
```

# reference
[ubuntu修改root密码以及开启root ssh远程连接\_ubuntu root密码设置-CSDN博客](https://blog.csdn.net/weixin_43085712/article/details/131044653?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ECtr-1-131044653-blog-114629865.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ECtr-1-131044653-blog-114629865.235%5Ev43%5Epc_blog_bottom_relevance_base4&utm_relevant_index=1)