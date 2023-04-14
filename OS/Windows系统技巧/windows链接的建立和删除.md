### 建立
```bat
##建立d:develop链接目录，指向远程的目标服务器上的e盘的对应目录。
mklink /d d:\develop \\138.20.1.141\e$\develop

##建立d:develop链接目录，指向远程的目标服务器上的e盘的对应目录。
mklink /d d:\recivefiles \\138.20.1.141\e$\recivefiles
```

### 删除
```
1.  #删除虚拟的链接目录，并不会删除远程文件夹真实文件，注意千万不能用del，del会删除远程的真实文件。  
    
2.  rmdir d:\recivefiles  
    
3.  rmdir d:\develop
```

### 命令格式
```bat
命令格式：mklink /d(定义参数) \MyDocs(链接文件) \Users\User1\Documents(原文件)

/d：建立目录的符号链接符号链接(symbolic link)  符号链接

/j：建立目录的软链接（联接）(junction)   目录联接

/h：建立文件的硬链接(hard link)   为文件创建别名，为某文件创建别名，可让不同的路径对应同一个文件的数据。

powershell 创建硬链接
New-Item Hardlink.txt -ItemType HardLink -Target C:\...\Demo.txt

New-Item [链接名称] -Itemtype SymbolicLink/HardLink -Target [目标绝对路径]

choco install linkshellextension -y  也可以用这个软件进行管理
```


在一台 Windows 电脑中，硬盘被分割为卷用以保存数据，在每一个卷上，数据对象被 NTFS 文件系统赋予了独一无二的**文件 ID** 以及**与之对应的文件路径**，文件路径和文件 ID 对应，文件 ID 和数据对象绑定，最终才呈现为可供用户打开、编辑的文件。

### 参考
[符号链接、硬链接及其在 Windows 上的应用举例 - 少数派](https://sspai.com/post/66834)
