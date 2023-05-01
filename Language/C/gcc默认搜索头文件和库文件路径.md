### 头文件
gcc在编译时如何去寻找所需要的头文件：

- 头文件的搜索会从-I指定的目录开始；
- 然后搜索gcc的环境变量 C_INCLUDE_PATH，CPLUS_INCLUDE_PATH，OBJC_INCLUDE_PATH 设置的目录；
- 再搜索系统目录 /usr/include 和 /usr/local/include（centos7中该目录下是空的）；
- 最后搜索gcc的一系列自带目录（如/usr/include/c++/4.8.5）。

### 库文件
编译的时候：

- gcc会先搜索-L指定的目录；
- 再搜索gcc的环境变量LIBRARY_PATH；
- 再搜索系统目录：/lib和/lib64、/usr/lib 和/usr/lib64、/usr/local/lib和/usr/local/lib64，这是当初compile gcc时写在程序内的。

### 运行时动态库的搜索路径
动态库的搜索路径搜索的先后顺序是：

1.  编译目标代码时指定的动态库搜索路径；(通过gcc参数 -rpath执行)
2.  环境变量`LD_LIBRARY_PATH`指定的动态库搜索路径；
3.  配置文件`/etc/ld.so.conf`中指定的动态库搜索路径；
4.  默认的动态库搜索路径`/lib`；
5.  默认的动态库搜索路径`/usr/lib`。