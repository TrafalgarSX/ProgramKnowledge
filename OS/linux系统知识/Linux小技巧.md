#### 如何删除cmake安装的软件
找到make install之后产生的这个文件install_manifest.txt
里面有安装的所有东西的路径，删除它们即可。
`cat install_manifest.txt | sudo xargs rm`
`xargs rm < install_manifest.txt`

#### cp命令不能显示进度和速度的替代方案
Linux复制文件的cp命令不会显示进度和速度，当我们复制大文件的时候，看不到复制的进度， 可以使用`rsync` 命令来替代
```
rsync -av --progress  src  dest
```

#### linux系统挂载usb设备
1. 识别usb设备
Once the USB drive has been plugged in, it will be registered as a new block device in /dev/ directory. To list all block devices, we can type:
```
lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 931.5G  0 disk 
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0   929G  0 part /var/lib/lxd/storage-pools/default
└─sda3   8:3    0     2G  0 part [SWAP]
sdb      8:16   1  57.3G  0 disk 
└─sdb1   8:17   1  57.3G  0 part 
```

可以看见sdb1 设备没有挂载

💡sdX, where X usually is a smaller case alphabet like 'b', 'c', 'd', and so on but is **seldom** 'a' as it usually denotes the primary block device containing the Operating System.

我们也可以用：
```
sudo fdisk -l 

从这条命令的输出可以看到 usb 设备的大小和其他信息
```

2. 创建一个挂载点
我们识别了 usb 设备后， 需要创建一个文件夹去挂载它。 通常我们在/mnt/文件夹下创建挂载点
```
sudo mkdir /mnt/mysub
```

3. 挂载设备
```
sudo mount /dev/sdb1 /mnt/myusb

我们可以改变这个文件夹的拥有者
sudo chown $USER /mnt/myusb

改变拥有者后， 就可以不再使用sudo命令访问这个文件夹了

再执行命令  lsblk, 就会看到 usb 设备已经被挂载成功，有了文件路径了
```

4. Unmount 设备
```
sudo umount /mnt/myusb
```


