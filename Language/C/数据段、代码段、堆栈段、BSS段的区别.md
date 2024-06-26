### Linux段管理
---
在Linux下内存分配是以页为单位的，而页是通过段管理，各个段之间是独立的，方便管理。一个简单的程序被编译成目标文件后的结构如下：

![[linux段.png]]
从上图可以看出，
- 已初始化的全局变量和局部静态变量保存在.data段中
- 未初始化的全局变量和未初始化的局部静态变量保存在.bss段中

目标文件各个段在文件中的布局如下：
![[cpp-segment-layout.png]]

下面我们介绍一下各个段：

- **init段**

**程序初始化入口代码**，在main()之前运行。

- **bss段**

BSS段属于静态内存分配。通常是指用来存放程序中**未初始化**的全局变量和未初始化的局部静态变量。未初始化的全局变量和未初始化的局部静态变量**默认值是0**，本来这些变量也可以放到`data`段的，但是因为它们都是0，**所以它们在`data`段分配空间并且存放数据0是没有必要的(💡为什么从.data段独立出来)**。

💡在程序运行时，才会给BSS段里面的变量分配内存空间。

在目标文件(`*.o`)和可执行文件中，BSS段只是为未初始化的全局变量和未初始化的局部静态变量预留位置而已，它并没有内容，所以它不占据空间。

  section table中保存了BSS段（未初始化的全局变量和未初始化的局部静态变量）内存空间大小总和。可以通过`objdump -h *.o`命令查看到。

- **data段**

数据段(data segment)通常是指用来存放程序中**已初始化**的全局变量和已初始化的静态变量的一块内存区域。数据段属于静态内存分配。

- **text段**

**代码段**(code segment / text segment)通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读，某些架构也允许代码段为可写，即允许修改程序。在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等。

- **rodata段**

**存放的是只读数据，比如字符串常量**，全局`const`变量和`#define`定义的常量。本段又称为`常量区`。例如：
```c
char *p = "123456";
```

这里`123456`就存放在rodata段中。

但是注意，**并不是所有的常量都放在`rodata`段的**，其特殊情况如下：

1) 有些立即数与指令编译在一起直接放在代码段；

2） 对于字符串常量，编译器会去掉重复的常量，让程序的每个字符串常量只有一份

3） 有些系统中rodata段是多个进程共享的，目的是为了提高空间的利用率

- **strtab段**

存储的是变量名、函数名等。例如：

```c
char *szPath = "/root";
void func();
```

上述变量名`szPath`和函数名`func`存储在strtab段里。

- **shstrtab段**

bss、text、data等段名也存储在这里。

- **rel.text段**

针对text段的重定位表，还有`rel.data`(针对data段的重定位表）

- **heap堆**

堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。当进程调用malloc()等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；当利用free()等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）。

- **stack栈**

是用户存放程序临时创建的局部变量，也就是说我们函数括弧`{}`中定义的变量（但不包括static声明的变量，static意味着在数据段中存放变量）。除此以外，在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。由于栈的先进后出特点，所以栈特别方便用来保存/恢复调用现场。从这个意义上讲，我们可以把栈看成一个寄存、交换临时数据的内存区。

#### 查看可执行文件的文件布局命令
```bash
objdump -h test
```

#### 数据段 .data
**只读数据段 .rodata**：  
只读数据段是程序使用的一些不会被更改的数据，使用这些数据的方式类似查表式的操作，由于这些变量不需要更改，因此只需要放置在只读存储器中即可。一般是**const修饰的变量以及程序中使用的文字常量**一般会存放在只读数据段中。

**已初始化的读写数据段：**  
已初始化数据是在程序中声明，并且具有初值的变量，这些变量需要占用存储器的空间，在程序执行时它们需要位于可读写的内存区域内，并且有初值，以供程序运行时读写。在程序中一般为**已经初始化的全局变量**，**已经初始化的静态局部变量**(static修饰的已经初始化的变量)

💡**常量区（特殊的常量存储区，属于静态存储区）**  
　　1) 常量占用内存,只读状态,决不可修改  
　　2) 常量字符串就是放在这里的，程序结束后由系统释放

##### 静态存储方式
所谓静态存储方式是指在**程序编译期间分配固定的存储空间的方式**。  
该存储方式通常是在**变量定义时就分定存储单元并一直保持不变**， 直至整个程序结束。**全局变量，静态变量**等就属于此类存储方式。

##### 静态存储区
一定会存在的而且会永恒存在、不会消失，这样的数据包括**常量、常变量（const 变量）、静态变量、全局变量**等。  
静态 、常量、全局变量就是存放在静态存储区，他们在程序编译完成后就已经分配好了，生命周期持续至程序结束。

### VMA和LMA
---
我们先简要介绍一下`VMA`和`LMA`这两个字段：

- `VMA`(virtual memory address): 程序区段在**执行时期的地址**
    
- `LMA`(load memory address): 某程序区**段加载时的地址**。因为我们知道程序运行前要经过：编译、链接、装载、运行等过程。装载到哪里呢？ 没错，就是LMA对应的地址里。
    

一般情况下，`LMA`和`VMA`都是相等的，不等的情况主要发生在一些嵌入式系统上。

### segment布局
---
如下我们简要的画出各区段的一个布局：
![[cpp-lma-vma.png]]

### 汇编文件分析
---
在讲解各变量存放位置之前，我们先简要介绍一下`.comm`和`.lcomm`:

- **.comm**: 声明为未初始化的通用内存区域；
    
- **.lcomm**: 声明为未初始化的本地内存区域；

其实上述两个均是属于bss段，不过作用域范围有些不同。`.lcomm`是为**不会从本地汇编代码之外进行访问的数据**保留的。两者使用的格式为：

```asm
.comm symbol, length
.lcomm symbol, length
```

例如：

```string
.section .bss
.lcomm buffer, 1000
```

👉该语句把1000字节的内存地址**赋予标签buffer**，再声明本地通用内存区域的程序之外的函数是不能访问他们的。

在bss段声明的好处是，**数据不包含在可执行文件中**。在数据段（.data)中定义数据时，它必须被包含在可执行程序中，因为必须使用特定值初始化它。因为不使用数据初始化bss段中声明的数据区域，所以内存区域被保留在运行时使用，并且不必包含在最终的程序中。

### 参考
---
[数据段、代码段、堆栈段、BSS段的区别 | Ivanzz](https://ivanzz1001.github.io/records/post/cplusplus/2018/11/12/cpluscplus-segment#1-linux%E6%AE%B5%E7%AE%A1%E7%90%86)

