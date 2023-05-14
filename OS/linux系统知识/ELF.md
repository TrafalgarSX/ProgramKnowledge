
### Object Files
TODO

### Program Loading and Dynamic Linking
TODO

### readelf和ldd的区别
ldd和readelf是两个不同的工具，用于不同的目的。

ldd是一个Linux命令行工具，用于显示可执行文件或共享库依赖的动态链接库。**它会列出可执行文件或共享库所依赖的所有动态链接库**，并且会显示它们的路径和版本信息。ldd通常用于诊断应用程序或共享库的依赖关系问题。

readelf是一个Linux命令行工具，用于查看ELF格式的可执行文件或共享库的信息。**ELF是一种二进制文件格式，用于在Linux系统上编译和链接程序。readelf可以用来查看ELF文件的头部信息、节表、符号表、重定位表等。它还可以用来查看ELF文件的版本信息、动态链接库依赖关系、符号信息等**。readelf通常用于调试和分析可执行文件或共享库。

因此，ldd和readelf是两个不同的工具，用于不同的目的。ldd用于查看可执行文件或共享库依赖的动态链接库，而readelf用于查看可执行文件或共享库的ELF格式的信息。

### 参考
[System V Application Binary Interface - DRAFT](https://refspecs.linuxfoundation.org/elf/gabi4+/contents.html)