## 虚拟终端
个人电脑及其用户能够与大型计算机的连接。
例子：
* 类Unix系统中的Xterm\ gnome-terminal等
* Windows中的控制台

## 回顾历史
**Teleprinter**或**teletypewriter** 作为输入输出设备连接到早期的大型计算机上。
TTY代表计算机终端，电传打字机**teletypewriter** 曾经是计算机的终端。
![[Pasted image 20230209212548.png]]
**UART** （Universal Asynchronous Receiver and Transmitter，通用异步接收和发送器
操作系统包含一个 `UART` 驱动程序，管理字节的物理传输，包括奇偶校验和流量控制。

## 终端模拟器（terminal emulator）
现在已经没有真实的终端了（不是一个物理设备了）， teletypewriter已经进入了博物馆，但是Linux/Unix仍然保留了TTY驱动和line discipline的设计和功能。终端成为了内核的一个模块，它可以直接向 TTY 驱动发送字符，并从 TTY 驱动读取响应然后打印到屏幕上。也就是说，**用内核模块模拟物理终端设备**，因此被称为**终端模拟器**(terminal emulator)。
![[Pasted image 20230209213535.png]]

终端模拟器就像过去的物理终端一样，它监听来自键盘的事件将其发送到 TTY 驱动，并从 TTY 驱动读取响应，通过显卡驱动将结果渲染到显示器上。TTY 驱动 和 **line discipline** 的行为与原先一样，但不再有 UART 和 物理终端参与。

## 伪终端(pseudo terminal, PTY)
**终端模拟器(terminal emulator)** 是运行在内核的模块，我们也可以让终端模拟程序运行在用户区。运行在用户区的终端模拟程序，就被称为**伪终端（pseudo terminal, PTY）**。

PTY 运行在用户区，更加安全和灵活，同时仍然保留了 TTY 驱动和 **line discipline** 的功能。常用的伪终端有 xterm，gnome-terminal，以及远程终端 ssh。
![[Pasted image 20230209213935.png]]

gnome-terminal 持有 `PTY master` 的文件描述符 `/dev/ptmx`。gnome-terminal 负责监听键盘事件，通过`PTY master`接收或发送字符到 `PTY slave`，还会在屏幕上绘制来自`PTY master`的字符输出。
  
gnome-terminal 会 fork 一个 shell 子进程，并让 shell 持有 `PTY slave` 的设备文件 `/dev/pts/[n]`，shell 通过 `PTY slave` 接收字符，并输出处理结果。

`PTY master` 和 `PTY slave` 之间是 TTY 驱动，会在 master 和 slave 之间复制数据，并进行会话管理和提供 **line discipline** 功能。

我们以实际的例子，看看在 terminal 执行一个命令的全过程。
- 我们在桌面启动终端程序 `gnome-terminal`，它向操作系统请求一个`PTY master`，并把 GUI 绘制在显示器上
- `gnome-terminal` 启动子进程 `bash`  
- `bash` 的标准输入、标准输出和标准错误都设置为 `PTY slave`  
- `gnome-terminal` 监听键盘事件，并将输入的字符发送到`PTY master`  
- **line discipline** 收到字符，进行缓冲。只有当你按下回车键时，它才会把缓冲的字符复制到`PTY slave`。
- **line discipline** 在接收到字符的同时，也会把字符写回给`PTY master`。`gnome-terminal` 只会在屏幕上显示来自 `PTY master` 的东西。因此，**line discipline** 需要回传字符，以便让你看到你刚刚输入的内容。
- 当你按下回车键时，TTY 驱动负责将缓冲的数据复制到`PTY slave`  
- bash 从标准输入读取输入的字符（例如 `ls -l` ）。注意，bash 在启动时已经将标准输入被设置为了`PTY slave`  
- bash 解释从输入读取的字符，发现需要运行 `ls`  
- bash fork 出 ls 进程。bash fork 出的进程拥有和 bash 相同的标准输入、标准输出和标准错误，也就是`PTY slave`  
- ls 运行，结果打印到标准输出，也就是`PTY slave`  
- TTY 驱动将字符复制到`PTY master`  
- `gnome-terminal` 循环从 `PTY master` 读取字节，绘制到用户界面上。

## Shell
Shell 是用户空间的应用程序，通常由 terminal fork 出来，是 terminal 的子进程。**Shell 不处理键盘事件，也不负责字符的显示，这是 terminal 要为它处理好的。** Shell 用来提示用户输入，解释用户输入的字符，然后处理来自底层操作系统的输出。
例子：Bash、Zsh

## 远程终端
ssh连接到远程主机。
![[Pasted image 20230209215615.png]]

参考：[理解Linux 终端、终端模拟器和伪终端](https://xie.infoq.cn/article/a6153354865c225bdce5bd55e)