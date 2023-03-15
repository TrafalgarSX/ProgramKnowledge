### 安装Ubuntu桌面
1. 安装Gnome
```bash
sudo apt update
sudo apt upgrade
sudo apt install ubuntu-desktop
```
2. 安装xfce
```bash
sudo apt install xubuntu-desktop
```
3. 安装KDE
```bash

```

#### 可以选择下载显示管理器
```bash
sudo apt install gdm3  or sddm or slim or lightdm
```

### 安装Ubuntu桌面后如何进行远程连接
Xrdp 是一个微软远程桌面协议（RDP）的开源实现，它允许你通过图形界面控制远程系统。通过 RDP，你可以登录远程机器，并且创建一个真实的桌面会话，就像你登录本地机器一样。

1. 使用Xrdp。用xrdp就可以使用windows上的远程桌面连接进行远程连接服务器。
```bash
sudo apt install xrdp
sudo systemctl status xrdp
//默认情况下，Xrdp 使用`/etc/ssl/private/ssl-cert-snakeoil.key`,它仅仅对“ssl-cert” 用户组成语可读。运行下面的命令，将`xrdp`用户添加到这个用户组：
sudo adduser xrdp ssl-cert
//重启服务，使得修改生效
sudo systemctl restart xrdp

//上面配置完后，windows远程登录会闪退
echo xfce4-session > ~/.xsession
touch .session
sudo vim /etc/xrdp/startwm.sh
//在 /etc/x11/Xsession前面加一行 xfce4-session
sudo service xrdp restart
//上面不知道哪一行起了作用，但是真的不崩溃了

//设置账户用户密码，应该需要设置，否则只能用root登录，我遇到的情况是这样，后续再看
sudo passwd root
```
xrdp服务监听 3389端口，通过xrdp方式远程访问Ubuntu服务端时，Ubuntu需要打开3389端口

#### 配置防火墙
Xrdp 守护程序在所有的网络接口上监听端口`3389`。如果你在你的 Ubuntu 服务器上运行一个防火墙，你需要打开 Xrdp 端口。
想要允许从某一个指定的 IP 地址或者 IP 范围访问 Xrdp 服务器，例如`192.168.33.0/24`,你需要运行下面的命令：
```bash
sudo ufw allow from 192.168.33.0/24 to any port 3389
```
**TODO 理解 什么是 SSH隧道，以前看过，又忘了。**
想要增加安全，你可以考虑 Xrdp 仅仅监听 localhost，并且创建一个 SSH 隧道，将本地机器的3389端口到远程服务器的同样端口之间的流量加密。

2. 使用VNC方式  TODO




### 参考
[ubuntu20.10中设置桌面共享的三种方式(任选其一) - 星宇x - 博客园](https://www.cnblogs.com/xingyu666/p/14132923.html)
[ubuntu远程桌面连接方式(vnc、xrdp、vino、xorg等概念) – late哥哥笔记](https://www.lategege.com/?p=691)