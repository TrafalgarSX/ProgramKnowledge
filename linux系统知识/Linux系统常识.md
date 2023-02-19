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