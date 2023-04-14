Linux服务管理方式有两种： service 和 systemctl
### service命令
service是一个运行System V init（/etc/init.d目录下的参数）的脚本命令
service命令其实是去/etc/init.d目录下，去执行相关程序
```bash
service redis start 
service mysqld start

等价于
sudo /etc/init.d/mysqld start
```
缺点：
1. 启动时间长。 init进程是串行启动，只有前一个进程启动完，才会启动下一个进程。
2. 启动脚本复杂。init进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况。

### systemctl命令
Systemd就是为了解决这些问题而诞生的。**它包括System and Service Manager,为系统的启动和管理提供了一套完整的解决方案**
systemd是Linux系统最新的初始化系统(init),作用是提高系统的初始速度，尽可能启动较少的进程，尽可能更多进程并发启动。
**使用了 Systemd，就不需要再用init 了。Systemd 取代了initd（Initd 的PID 是0） ，成为系统的第一个进程（Systemd 的PID 等于 1），其他进程都是它的子进程。**

systemd对应的进程管理命令是systemctl
1) systemctl命令兼容了service
即systemctl也会去/etc/init.d目录下，查看，执行相关程序
```
//reboot
sudo systemctl reboot

systemctl redis start
systemctl redis stop
# 开机自动启动
systemctl enable redis
```
2) systemctl命令管理systemd的资源Unit
systemd的Unit放在目录/usr/lib/systemd/system(Centos)或/etc/systemd/system(Ubuntu)

hostnamectl命令用于查看当前主机的信息。
localectl命令用于查看本地化设置
timedatectl用于查看当前时区设置


### 总结
-   `init` 是最初的进程管理方式
-   `service` 是`init` 的另一种实现
-   `systemd` 则是一种取代 `initd` 的解决方案


[Systemd 入门教程：命令篇 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)