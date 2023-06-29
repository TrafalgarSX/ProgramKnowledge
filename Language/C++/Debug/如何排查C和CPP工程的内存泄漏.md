### 内存泄漏检查方法
---
#### Valgrind
此工具是linux系统独有的（macos不确定）

Valgrind 可以用来检测程序是否有非法使用内存的问题，例如访问未初始化的内存、访问数组时越界、忘记释放动态内存等问题。在 Linux 可以使用下面的命令安装 Valgrind：
```bash
yum install valgrind
# 或者 编译安装
wget ftp://sourceware.org/pub/valgrind/valgrind-3.13.0.tar.bz2
bzip2 -d valgrind-3.13.0.tar.bz2
tar -xf valgrind-3.13.0.tar
cd valgrind-3.13.0
./configure && make
sudo make install
```

##### 检测内存泄漏
```bash
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./testLeak
```

为了使 valgrind 发现的错误更精确，如能够定位到源代码行，建议在编译时加上-g 参数，编译优化选项请选择 O0，虽然这会降低程序的执行效率。

上面的命令可以用来检测C/C++程序的内存泄漏，但是C++程序有个不同：
```cpp
#include <string>

int main()

{
    auto ptr = new std::string("Hello, World!");

    delete ptr;

    return 0;
}
```

检测上面C++程序是否内存泄露时会出现：
```bash
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./main_cpp
==31438== Memcheck, a memory error detector
==31438== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==31438== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==31438== Command: ./main_cpp
==31438==
==31438==
==31438== HEAP SUMMARY:
==31438==     in use at exit: 72,704 bytes in 1 blocks
==31438==   total heap usage: 2 allocs, 1 frees, 72,736 bytes allocated
==31438==
==31438== 72,704 bytes in 1 blocks are still reachable in loss record 1 of 1
==31438==    at 0x4C2DBF6: malloc (vg_replace_malloc.c:299)
==31438==    by 0x4EC3EFF: ??? (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.21)
==31438==    by 0x40104E9: call_init.part.0 (dl-init.c:72)
==31438==    by 0x40105FA: call_init (dl-init.c:30)
==31438==    by 0x40105FA: _dl_init (dl-init.c:120)
==31438==    by 0x4000CF9: ??? (in /lib/x86_64-linux-gnu/ld-2.23.so)
==31438==
==31438== LEAK SUMMARY:
==31438==    definitely lost: 0 bytes in 0 blocks
==31438==    indirectly lost: 0 bytes in 0 blocks
==31438==      possibly lost: 0 bytes in 0 blocks
==31438==    still reachable: 72,704 bytes in 1 blocks
==31438==         suppressed: 0 bytes in 0 blocks
==31438==
==31438== For counts of detected and suppressed errors, rerun with: -v
==31438== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

使用 Valgrind 分析 C++ 程序时，有一些问题需要留意。例如，这个程序并没有发生内存泄漏，但是从`HEAP SUMMARY`可以看到，程序分配了 2 次内存，但却只释放了 1 次内存，为什么会这样呢？  

实际上这是由于 C++ 在分配内存时，为了提高效率，使用了它自己的内存池。当程序终止时，内存池的内存才会被操作系统回收，所以 Valgrind 会将这部分内存报告为 **reachable** 的，需要注意，**reachable 的内存不代表内存泄漏**，例如，从上面的输出中可以看到，有 72704 个字节是 reachable 的，但没有报告内存泄漏。

参考网站 [使用 Valgrind 检测 C++ 内存泄漏 | Senlin's Blog](http://senlinzhan.github.io/2017/12/31/valgrind/)

#### AddressSanitizer
linux下的 gcc 和 clang 编译器都带有 AddressSanitizer 功能，可以检测内存泄漏， 指针释放后使用， 堆，栈缓存访问溢出等问题。

Visual Studio 2019 version 16.9之后也有 AddressSanitizer， 但是目前Visual Studio的 AddressSanitizer 并不支持检测内存泄漏。 windows下的 gcc 和 clang 的AddressSanitizer 也不支持检测内存泄漏，仅支持部分功能。

AddressSanitizer 介绍：
[AddressSanitizer · google/sanitizers Wiki · GitHub](https://github.com/google/sanitizers/wiki/AddressSanitizer)

Visual Studio AddressSanitizer 介绍：
[AddressSanitizer | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/sanitizers/asan?view=msvc-170)

AddressSanitizer 和 Valgrind 一样，在Debug下使用， 可以精确指出内存泄漏的函数调用栈与相关的代码。

AddressSanitizer 非常快，只拖慢程序 2 倍速度。Valgrind 其会极大的降低程序运行速度，大约降低10倍。

##### 如何在C/C++工程中使用AddressSanitizer
[[CMake中如何设置sanitize]]

使用 AddressSanitizer后， 正常运行代码，如果出现内存泄漏会出现：
```bash
==905317==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 40 byte(s) in 1 object(s) allocated from:
    #0 0x7fc201cb4867 in __interceptor_malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:145
    #1 0x558a0d8fc2a8 in testMemoryLead /home/eetrust/branch43.220/Debug/memory.c:8
    #2 0x558a0d8fc339 in main /home/eetrust/branch43.220/Debug/memory.c:21
    #3 0x7fc201829d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: 40 byte(s) leaked in 1 allocation(s).

```

在使用 AddressSanitizer 时， clang默认使用静态链接 asan 库， gcc 默认使用动态库，如果编译或使用过程中出现问题，可以根据这个特点排查。

#### AddressSanitizer 和 Valgrind 适用范围
AddressSanitizer 和 Valgrind 都适用于可执行程序， 如何输出的是动态库，可以将测试代码编译成可执行程序，链接相关的动态库和静态库， 测试相关的接口是否有内存泄漏。

**经过测试， 如果输出动态库给 Java 通过 Jni 调用, 这两个工具都不能正常测试动态库相关接口的内存泄漏**， 所以目前可以先通过工具测试C层接口是否有内存泄漏， 然后通过循环跑 Java 调用 Jni 接口的测试程序，查看jni的封装是否出现内存泄漏， 缩小排查范围。

#### Visual Studio排查内存泄漏
##### 方式一 使用CRT检测内存泄漏
```cpp
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <stdio.h>
#include <string.h>
#include <iostream>

void testMemoryLeak() {
	int* p = (int*)malloc(sizeof(int) * 10);
	if (p != NULL) {
		memset(p, 0x00, sizeof(int) * 10);
	}
	else {
		exit(-1);
	}
	for (size_t i = 0; i < 10; i++)
	{
		printf("printf out %d\n", p[i]);
	}

	//free(p);
	//printf("printf out %d\n", p[0]);
}

void testMemoryLeak_std() {
	int* p = new int[10];

	//delete p;
}

int main(void) {
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF); // 使用CRT来检测leak
	_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_FILE); // 将leak输出到文件 
	_CrtSetReportFile(_CRT_WARN, _CRTDBG_FILE_STDERR); // 以stderr做为输出文件

	testMemoryLeak();
	testMemoryLeak_std();

	printf("test Memory Leak end\n");
}
```

首先在测试程序开头定义宏 `_CRTDBG_MAP_ALLOC`, 并引入 `#include<crtdbg.h>`, 然后在测试程序前调用即可检测相关代码的内存泄漏：

```cpp
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF); // 使用CRT来检测leak
	_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_FILE); // 将leak输出到文件 
	_CrtSetReportFile(_CRT_WARN, _CRTDBG_FILE_STDERR); // 以stderr做为输出文件
```

检测到内存泄漏后会出现：
```
Detected memory leaks!
Dumping objects ->
{109} normal block at 0x00000170693101F0, 40 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD
 
D:\workspace\cpp\testMemoryLeak\MemoryLeak.cpp(9) : {107} normal block at 0x0000017069310180, 40 bytes long.
 Data: <                > 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Object dump complete.
```

##### 方式二 使用Visual Studio的内存监测工具
当运行程序时，Visual Studio 2015及之后的版本提供了 `Diagnostic Tools`, 可以用来查看CPU, 内存的占用， 并且可以设置内存快照。 

**设置内存快照后，可以看到与上一次内存快照之间的差别：内存占用是增大还是减小， 分配了哪些内存。** 点击进入后内存快照，还可以查看相对与上一次内存快照相比，增加或减少内存的分配或回收的代码位置。 这样就可以**在调用某一个函数前后分别添加内存快照，然后查看是否有内存泄漏**，如果有，可以进入内存快照，点击相关的内存分配，查看是代码哪里分配了内存。

要使用内存快照，就要在 Debug 运行程序后先暂停， 点击菜单 Debug 项 --》 Windows 项 --》 `show Diagnostic Tools` 项， 即可出现相关的 `Diagnostic Tools` 页面，在页面的下方 `Memory Usage` 框中有 `Take Snapshot`（快照）按钮，点击按钮即可记录一次内存快照。

Visual Studio 还有一个工具就是 Debug 菜单下的 `Performance Profiler`, 这个工具可以选择监测程序运行期间的CPU占用， GPU占用， 内存占用， 文件IO等， 我们可以选择监测内存占用， 然后循环跑相关的测试代码， 看运行过程中， 内存占用的图是否在不停升高，即可直观的看出测试相关的接口是否有内存泄漏。