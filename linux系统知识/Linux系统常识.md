### Unix系统起源和发展
![[Pasted image 20230219002241.png]]

### Unix信号
信号（英语：Signals）是Unix、类Unix以及其他POSIX兼容的操作系统中**进程间通讯**的一种有限制的方式。它是一种**异步**的通知机制，用来提醒进程一个事件已经发生。当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，此时，任何非原子操作都将被中断。如果进程定义了信号的处理函数，那么它将被执行，否则就执行默认的处理函数。

信号类似于中断，不同之处在于：
* 中断由处理器调解并由内核处理
* 信号由内核调解(可能通过系统调用)并由进程处理

* 在一个运行的进程的控制终端键入特定的组合键可以向它发送某些信号：
	* `Ctrl-C`发送INT信号（SIGINT）；默认情况下，这会导致进程终止。
	* `Ctrl-Z`发送TSTP信号（SIGTSTP）；默认情况下，这会导致进程挂起。
	* `Ctrl-\`发送QUIT信号（SIGQUIT）；默认情况下，这会导致进程终止并且将内存中的信息转储到硬盘（核心转储）。
（这些组合键可以通过stty命令来修改。）

* kill()系统调用会在权限允许的情况下向进程发送特定的信号，类似地，**kill**命令允许用户向进程发送信号。raise(3)库函数可以将特定信号发送给当前进程。
* **像除数为零、段错误**这些异常也会产生信号（这里分别是SIGFPE和SIGSEGV，默认都会导致进程终止和核心转储）。
* 内核可以向进程发送信号以告知它一个事件发生了。例如当进程将数据写入一个已经被关闭的管道是将会收到SIGPIPE信号，默认情况下会使进程关闭。
信号列表: `<signal.h>` 中有定义。

### POSIX（Portable Operating System Interface）

POSIX defines both the system and user-level application programming interfaces (APIs), along with command line shells and utility interfaces, for software compatibility (portability) with variants of Unix and other operating systems.

glibc 为程序员提供丰富的 API（Application Programming Interface），这些API都是遵循POSIX标准的，API的函数名，返回值，参数类型等都必须按照POSIX标准来定义。

**POSIX兼容也就指定这些接口函数兼容，但是并不管API具体如何实现。**

### What is shebang?
在[计算](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97 "计算")领域中，**Shebang**（也称为**Hashbang**）是一个由[井号](https://zh.wikipedia.org/wiki/%E4%BA%95%E5%8F%B7 "井号")和[叹号](https://zh.wikipedia.org/wiki/%E5%8F%B9%E5%8F%B7 "叹号")构成的字符序列_`#!`_，其出现在文本文件的第一行的前两个字符。 在文件中存在Shebang的情况下，[类Unix操作系统](https://zh.wikipedia.org/wiki/%E7%B1%BBUnix%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F "类Unix操作系统")的[程序载入器](https://zh.wikipedia.org/w/index.php?title=%E7%A8%8B%E5%BA%8F%E8%BD%BD%E5%85%A5%E5%99%A8&action=edit&redlink=1 "程序载入器（页面不存在）")会分析Shebang后的内容，将这些内容作为解释器指令，并调用该指令，并将载有Shebang的文件路径作为该解释器的参数

由于`#`符号在许多[脚本语言](https://zh.wikipedia.org/wiki/%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%80 "脚本语言")中都是注释标识符，Shebang的内容会被这些脚本解释器自动忽略。

hebang的名字来自于[SHArp](https://zh.wikipedia.org/wiki/%E5%8D%87%E9%9F%B3%E7%AC%A6 "升音符")和[bang](https://zh.wikipedia.org/wiki/%E5%8F%B9%E5%8F%B7 "叹号")，或_haSH bang_的[缩写](https://zh.wikipedia.org/wiki/%E7%BC%A9%E5%86%99 "缩写")，指代Shebang中`#!`两个符号的典型Unix名称。 Unix术语中，井号通常称为_sharp_，_hash_或_mesh_；而叹号则常常称为_bang_。

### 守护进程
#### init
历史上，Linux的启动一直采用init进程
在类Unix的计算机操作系统中，init是在启动计算机系统期间启动的第一个进程。
init是一个守护进程，他将持续运行，知道系统关闭。**他是所有其他进程的直接或间接的父进程**
因为init的参数全在/etc/init.d目录下，所以使用init启动一个服务，应该这样：
```
sudo /etc/init.d/nginx start
```

根据Linux管理，字母d是守护进程(daemon)的缩写。Systemd这个名字的含义，就是它要守护整个系统。
什么是守护进程管理工具
TODO
systemd是什么？？？
supervisorctl是什么？？？