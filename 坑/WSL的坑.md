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