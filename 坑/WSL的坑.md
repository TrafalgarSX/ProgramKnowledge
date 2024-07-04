### 占用空间突然增大
在wsl上编译了几个工程后，我的C盘空间少了60G，震惊！！！😨😱
原因：
WSL使用虚拟硬盘(VHD)存储linux下的文件，随着Linux下文件越来越多，占用空间也会不断增长，不过有个最大限制256G。但是，在Linux中减少文件占用，WSL却没有相应的减少硬盘空间的占用。所以为了避免碰到256G的限制，或者硬盘空间告警，**在删除掉linux下的文件后，我们需要手动释放这部分空间（删除linux下的文件后，空间不会自动减少）。**

1.要找到wsl的vhd文件: `ext4.vhdx(%PROFILE%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\).`
2.执行命令：
```powershell
wsl --shutdown
diskpart
# open window Diskpart
select vdisk file="C:\Users\guoya\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\ext4.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```
这样C盘的空间就会回来。👌

### wsl 安装后是默认没有 root 用户密码的， 需要自己设置

[[创建或者修改root用户密码]]

### WSL移除PATH中Windows共享的位置
在wsl中 `echo $PATH`会看到大量`/mnt/c`开头的位置,这些都是从Windows系统的`PATH`变量中合并过来的,这样就容易出现在WSL中如果有和Windows中相同名称的命令,可能会出现调用紊乱,下面记录一下如何让WSL保持自己`PATH`变量的纯洁性

运行命令 `vim /etc/wsl.conf`
```
# 不加载Windows中的PATH内容
[interop]
appendWindowsPath = false

# 不自动挂载Windows系统所有磁盘分区
[automount]
enabled = false
```
保存退出后关闭WSL,再重新运行WSL 

💡一般不加载Windows中的PATH内容, **如果加载了， wsl 下命令运行会非常慢。需要什么自己单独加载。

如果没有起效，从windows命令行终止linux子系统，重新打开后即可：
```
wsl --list
# 适用于 Linux 的 Windows 子系统:
# Ubuntu-18.04 (默认)
wsl --terminate Ubuntu-18.04
```

#### 上面关闭了自动挂载，**那么没有自动挂载之后如何手动挂载呢？**
```
mkdir /mnt/g

# 使用drvfs格式将U盘或移动硬盘插入电脑后的盘符G，进行挂载
mount -t drvfs G: /mnt/g

# 需在Windows下正常弹出U盘或移动硬盘，需要先卸载/mnt/g
umount /mnt/g
```

#### 不加载 windows 的PATH内容， vscode 的 wsl 无法使用
单独添加 vscode 的 环境变量即可。
```bash
export PATH=$PATH:/mnt/c/Code/bin
```

注意不是 `/mnt/c/Code/` , 虽然 该路径下有 code.exe, 但是他在 wsl 是无法开起 code 的 wsl, `/mnt/c/Code/bin` 下的 code 才可以。


#### 配置wsl官方参考文档
[Automatically Configuring WSL - Windows Command Line](https://devblogs.microsoft.com/commandline/automatically-configuring-wsl/)

如何限制WSL使用的内存和CPU
使用VNC服务器在WSL上使用GUI 应用程序
[改善 WSL 体验的 5 件事 - Windows教程网](https://rishivoice.com/post/45874.html)