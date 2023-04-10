### gdb附加进程调试
工作中有个需求是java程序jni调用C/C++的动态库。

在Linux远程服务器环境中，debug不方便，需要使用gdb进行调试，这种需求就需要使用gdb附加进程调试功能。

流程：
1. java 程序进入jdb调试并设置断点。与windows不同的是，windows下使用附加visual studio进程调试的时候需要在加载动态库之前设置附加进程调试（选择附加进程调试，并附加jps输出的java程序进程），linux下使用gdb附加进程调试需要在java程序加载了动态库之后，gdb才能加载到符号表，才能进行下一步的调试。
```
gdb attach processId

processId 可以通过两种方法获取：
1. 如果是附加在java程序上， 使用jps命令查看当前正在运行java程序的 进程id
2. 使用 ps aux | grep processname, 使用此命令查看当前附加进程的id, 这个方式适用与各种语言调用C/C++库
```

2. 进入gdb后，开始设置断点，当java程序命中 动态库的断点后，即可进入gdb进行调试，与正常C/C++程序一样即可。
3. gdb调试完成，要返回java程序中，需要运行  `cont` 或 `c` 命令，运行后，就可以返回jdb中继续进行调试。
4. 根据需求2. 3. 交替进入即可

>⚠💡第二步中，java命中动态库中的断点后，进入gdb中调试时，不能使用 next进行下一步，会卡死，报出 ...no linenumber之类的错误后无法调试，应该使用cont命令，就会命中动态库中设置的断点。 动态库调试结束后，运行cont命令返回jdb调试。

以上就是完整的流程。

gdb调试的三种方式
-   gdb filename  目标程序调试
-   gdb attach pid  附加进程调试
-   gdb filename corename  调试core文件

[[使用emacs进行debug]] 在命令行更为简单和直观，emacs目前应该是最好的gdb调试前端。

#### 调试core文件
有时候，服务器程序运行一段时间后会突然崩溃，这并不是我们希望看到的，需要解决这个问题。只要程序在崩溃的时候有 core 文件产生，就可以使用这个 core 文件来定位崩溃的原因。当然，Linux 系统默认是不开启程序崩溃产生 core 文件这一机制的，我们可以使用 ulimit -c 命令来查看系统是否开启了这一机制。

发现 core file size 那一行默认是 0，表示关闭生成 core 文件，可以使用“ulimit 选项名 设置值”来修改。
**ulimit -c unlimited**

我们希望这个选项永久生效，永久生效的方式是把“ulimit -c unlimited”这一行加到 /etc/profile 文件中去，放到这个文件最后一行即可。

```
gdb filename corename
```



visual studio可以当作gdb前端？看起来像是ssh远程连接到linux机器，然后通过ssh在本地编辑远程文件，远程linux的gdb调试  vs通过gdb-server查看
[使用Visual Studio 2017开发Linux程序 - dchao - 博客园](https://www.cnblogs.com/dongc/p/6599461.html)
[【GDB】VisualStudio 2017跨平台（Linux）调试|可视化GDB|visual GDB\_gdb可视化\_bandaoyu的博客-CSDN博客](https://blog.csdn.net/bandaoyu/article/details/89484744)