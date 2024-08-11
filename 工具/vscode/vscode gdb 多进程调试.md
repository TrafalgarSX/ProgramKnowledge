
[使用gdb调试多进程及多线程程序\_follow-exec-mode-CSDN博客](https://blog.csdn.net/qq_34992845/article/details/72858566?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-72858566-blog-122217880.235^v43^pc_blog_bottom_relevance_base4&spm=1001.2101.3001.4242.2&utm_relevant_index=4)


```json
{
    "cmake.debugConfig": {
        // gdb debug multi process
        "setupCommands": [
            {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description": "Set follow-fork-mode to child",
                "text": "set follow-fork-mode child"
            },
            {
                "description": "Set detach-on-fork to off",
                "text": "set detach-on-fork off"
            }
        ]
    }
}
```

## 多进程调试：

首先来了解下会可能会用到的调试命令：

> 1.默认设置下，在调试多进程程序时GDB只会调试主进程。  
> 但是如果设置follow-fork-mode的话，就可调试多个进程。  
> set follow-fork-mode parent|child:  
> 进入gdb后默认调试的是parent，若是想要调试child的话，需要设置set follow-fork-mode child,然后进入调试。但是这种方式在同一时间只能调试单个进程。
> 
> set follow-fork-mode:查看当前调试的fork的模式。

2.detach-on-fork on|off:  
设置为on的话，只调试父进程或子进程其中一个，需要根据follow-fork-mode决定，这是默认模式。  
设置成off的话，父子进程都在gdb的控制之下，其中一个进程正常调试，需要根据follow-fork-mode决定，另一个进程会被设置为暂停状态。  
show detach-on-fork：查看detach-on-fork的模式。

扩展：GDB将每一个被调试程序的执行状态记录在一个名为inferior的结构中。一般情况下一个inferior对应一个进程，每个不同的inferior有不同的地址空间。inferior有时候会在进程没有启动的时候就存在。

> 3.info inferiors:  
> 显示GDB调试的所有inferior，GDB会为它们分配ID。其中带有*的进程是当前正在调试的进程。

4.inferior num：切换到编号为num的进程进行调试。

> 5,。add-inferior[-copies n][infno]:  
> 增加n个inferior并执行程序为executable。如果不指定n则默认是增加一个inferior。  
> 如果不指定executable，则执行程序留空，增加后可使用file命令重新指定执行程序。这时候创建的inferior其关联的进程并没启动。