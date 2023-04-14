#### 如何删除cmake安装的软件
找到make install之后产生的这个文件install_manifest.txt
里面有安装的所有东西的路径，删除它们即可。
`cat install_manifest.txt | sudo xargs rm`
`xargs rm < install_manifest.txt`