### KVM 虚拟化环境搭建 - ProxmoxVE
----
KVM整套解决方案：

- KVM: 内核级别的虚拟化功能， 主要模拟指令执行和 I/O
- QEMU： 提供用户操作界面， VNC/SPICE 等远程终端服务
- Libvirtd: 虚拟化服务， 运行在 Hypervisor 上提供TCP接口用于操作虚拟机的创建和启停

第一个是 Linux 内核自带，后两个是各大发行版自带的标准组件。这里的 qemu 不是原生的 Fabrice 的 qemu，而是定制的 kvm 版本的 qemu 。

可以用 qemu-system-x86 程序写很长的一串参数来启动你的虚拟机，但是这样十分不友好，所以有了 **Libvirtd** 这个东西，将物理机的所有资源：存储/网络/CPU 管理起来，并且提供统一的服务接口。

那么 KVM + Libvirtd 有几种不同层次的玩法：

- 初级： 在 /etc/libvirtd/qemu 下面用 xml 描述每一台虚拟机的配置，然后用 virsh 在命令行管理虚拟机，最后用 VNC/SPICE 按照配置好的端口链接过去，模拟终端操作。
- 中级： 使用各种 libvirtd 的前端，比如基于桌面 GUI 的 Virt Manager 给你界面上直接编辑和管理虚拟机，桌面版本的 VNC/SPICE 会自动弹出来，像 VmWare 一样操作。
- **高级**：使用基于 Web 的各种 virt manager 进行集群管理，比如轻量级的 WebVirtMgr / Kimchi，适合小白的 Proxmox VE。基本是用 WebVnc/Web
- 超级：上重量级的 OpenStack，搭配自己基于 libvirt （libvirtd 的客户端库，比如有 python-libvirt 的封装）写的各种自动化脚本。

现在公司用的方案就是第三种，前两种方案不好，OpenStack 又基本需要一个 DevOps 团队才玩得转。

所以作为个人或者中小团队，买了台硬件过来，想把它变成一套小型的阿里云，腾讯云的系统，**可以在 web 上创建/配置虚拟机，装系统，管理硬件资源，进行迁移备份等**，基本就是第三套解决方案。

#### 如何安装 PVE 
最简单的做法是直接下载 ProxmoxVE 社区版的 ISO ，刻录到 U 盘里，按照安装普通操作系统一样的安装到物理机上，立马把你的物理机变成一台 Hypervisor

Proxmox VE 安装后启动，你可以登陆进去，ProxmoxVE 基于 Debian 9 ，进去可以用 apt-get 进行版本升级。接着按提示打开网页：https://your-ip:8006/ 用系统 root 密码登陆

ProxmoxVE 可以方便的管理各种硬件资源（计算，存储，网络）和虚拟机系统，你可以方便的新建一台虚拟机并进行硬件配置

配置好了以后启动虚拟机，选择“console”就可以使用 webvnc 终端安装操作系统了

ProxmoxVE 除了上面这些功能外，还能方便的对虚拟机进行：复制，快照，迁移。你如果有硬盘阵列，它还能使用 ZFS 帮你做软件 Raid，保证数据安全性，不需要学习复杂的 zfs 命令行，web上点点点就出来了。

个人使用的话，二十分钟就可以在安装完 Proxmox VE，里面创建三个虚拟机，一个跑黑群晖或者各种 Docker 容器，一个开发用 Ubuntu/Debian ，另外一个跑个 Windows 10，**网络设置成桥接模式，同一个路由器下可以直接访问，最后把他们的电源选项都配置成随同物理机开机自动启动**，妥了，基本满足日常使用。

