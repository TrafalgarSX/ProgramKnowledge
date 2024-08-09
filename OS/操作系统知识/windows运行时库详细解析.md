### windows 运行时详细解析

#### 背景知识

要弄懂这个问题，首先要先了解 Microsoft Visual C++ 运行时库 (CRT) 。
Microsoft Visual C++ 运行时库（CRT）是一套由 Microsoft 提供的标准库，包含了一系列用于执行常见操作的函数和数据结构，如输入输出处理、字符串操作、内存分配和数学计算等。这些库提供了基础的程序运行环境，使用Visual C++开发的程序都需要他们。

如果程序所需的运行时库在用户电脑上找不到，而且分发软件时没有打包相应的运行时库，那么用户可能就会弹出前面的报错提示。

#### VCRuntime

在Visual Studio 2015 之前，CRT 主要指VCRuntime，VCRuntime可以分为几个不同的部分：

    C 运行时库: 这是 CRT 的核心部分，提供了 C 语言标准库所定义的功能，如 printf()、malloc()、fopen() 等。

    C++ 标准库: 包含 C++ 标准库的实现，涵盖了如 STL 容器（如 vector、map 等）、算法、字符串类（如 std::string）以及其他标准功能。

    C++/CLI 运行时库: 用于支持 C++/CLI 语言的特定功能，这是一个为 .NET 框架设计的 C++ 语言变种。

    并发运行时: 提供了并发和并行编程模型的支持，如任务并行库（TPL）。

    安全增强的 CRT: 为了防止缓冲区溢出等安全问题，Microsoft 提供了一些增加了安全检查的函数版本，如 strcpy_s() 代替 strcpy()。

    CRT 的调试版本: 包含了额外的调试信息和检查功能，帮助开发者在开发阶段发现和修复错误。

Visual C++ 运行时库可以以两种方式与应用程序链接：

    静态链接（/MT 或 /MTd 编译器选项）: 运行时库的代码会被编译到应用程序的二进制文件中。这样的应用程序不需要额外的 DLL 文件就可以运行，但会增大最终的可执行文件大小，并且如果运行时库更新，程序也需要重新编译分发。

    动态链接（/MD 或 /MDd 编译器选项）: 应用程序在运行时依赖外部的 DLL 文件，如 msvcrXXX.dll、msvcpXXX.dll、vcruntimeXXX.dll 等。动态链接的好处是减小了可执行文件的大小，并且当运行时库更新时，通常不需要重新编译程序。


#### Universal CRT (UCRT) 的出现

从 Visual Studio 2015 开始，CRT 升级成了 Universal CRT (UCRT) 。UCRT 是操作系统级别的组件，作为 Windows 的一部分，提供跨版本的 C 运行时支持。

    MSVCRT（Microsoft Visual C++ 运行时）被淘汰的原因主要有：

        向后兼容性问题，它未能跟上时代，不兼容C99标准，并且缺少一些特性。例如，它不兼容C99标准中的printf()函数族
        它不支持UTF-8本地化设置，因此跨平台性很差
        与MSVCRT链接的二进制文件不应该与UCRT链接的二进制文件混合使用，因为它们内部结构和数据类型有所不同。对于MSVC编译的二进制文件也适用相同规则，因为MSVC默认使用UCRT（除非进行更改）。

据说UCRT是微软用C++重新实现了之前用C写的vcruntime了，微软还顺便吐槽了一下C语言，同时表扬C++的安全性和便利性。这主要是因为 C++ 提供了更好的类型安全、抽象和资源管理（例如 RAII，即资源获取即初始化）等特性，相对于 C，它可以帮助开发者更容易地编写出更安全和更易于维护的代码。

另外，微软的目标是希望 UCRT 成为所有 Windows 应用程序的共同基础，UCRT 作为 Windows 的一部分，通过 Windows Update 分发，确保了所有系统上的运行时行为的一致性和最新性。
更多信息，可以参考：upgrade-your-code-to-the-universal-crt

api-ms-win-crt-runtime-XXX.dll就是Universal CRT (UCRT) 的组件之一。

由于Visual C++ 2015(版本14.0)之后开始，旧淘汰了老的一套msvcpXXX.dll的二进制模式了，因此，在电脑中是找不到vcruntime140.dll版本以后的运行时了，例如msvcp141.dll是不存在的：

#### 展开说说VCRuntime 各个文件的作用：

以 Microsoft Visual Studio 2015（代号为 Visual Studio 14.0）的 C++ 运行时库组件文件为例，每个文件的功能如下：
concrt140.dll

    名称: Concurrency Runtime
    功能: 提供支持并发编程模型的功能。这包括任务并行库、并行算法、同步数据结构等。它是用于开发利用多核处理器性能的应用程序的重要组件。

msvcp140.dll

    名称: Microsoft C++ Runtime Library
    功能: 包含 C++ 标准库的函数和类，例如标准输入输出流、字符串操作、数学函数等。这是编写 C++ 程序时常用到的运行时库，用于动态链接。

vccorlib140.dll

    名称: Visual C++ C++/CX Runtime Library
    功能: 为 C++/CX (Component Extensions) 提供支持，这是 Microsoft 为简化 Windows 运行时组件编程而创建的一套语言扩展。这个库主要用于开发 Windows Store 应用程序。

vcruntime140.dll

    名称: Visual C++ Runtime Library
    功能: 包含 C 运行时库的核心组件，如内存分配、字符串操作和数学运算等。它也包含了基本的运行时错误处理和其他关键的函数。这个 DLL 专注于 C 语言层面的运行时支持。

#### 展开说说Universal CRT (UCRT)各个文件的作用

从VCRuntime发展到UCRT，二进制按功能划分得更细了，完整的19041 SDK的UCRT有43个文件，因此，我们根据文件体积，选取Top20的UCRT文件来说明各个模块的功能：

        ucrtbase.dll
            提供了 C 运行时库的基础部分，包括标准的输入/输出操作、字符串处理、数学计算等。

        api-ms-win-crt-private-l1-1-0.dll
            提供了一些内部使用的或者不公开的 C 运行时库功能。

        api-ms-win-crt-math-l1-1-0.dll
            包含了 C 运行时库中的数学函数，如 sin, cos, sqrt 等。

        api-ms-win-crt-multibyte-l1-1-0.dll
            提供了用于多字节字符集（MBCS）和宽字符集（Unicode）之间转换的函数。

        api-ms-win-crt-string-l1-1-0.dll
            包含了 C 运行时库中处理字符串的函数，如 strcpy, strlen 等。

        api-ms-win-crt-stdio-l1-1-0.dll
            提供了标准输入输出相关的函数，如 printf, scanf, fopen 等。

        api-ms-win-crt-runtime-l1-1-0.dll
            包含了 C 运行时库中与程序运行时环境相关的函数，如 abort, exit, 环境变量处理等。

        api-ms-win-crt-convert-l1-1-0.dll
            提供了类型转换相关的函数，如数字到字符串的转换等。

        api-ms-win-core-file-l1-1-0.dll
            提供了文件操作相关的函数，如 CreateFile, ReadFile 等。

        api-ms-win-crt-time-l1-1-0.dll
            包含了 C 运行时库中处理日期和时间的函数。

        api-ms-win-core-processthreads-l1-1-0.dll
            提供了进程和线程管理相关的函数。

        api-ms-win-core-localization-l1-2-0.dll
            提供了本地化和国际化支持相关的函数。

        api-ms-win-crt-filesystem-l1-1-0.dll
            包含了 C 运行时库中处理文件系统相关操作的函数，如路径操作等。

        api-ms-win-core-synch-l1-1-0.dll
            提供了同步对象（如互斥锁、事件等）的操作函数。

        api-ms-win-crt-process-l1-1-0.dll
            包含了与进程操作相关的函数。

        api-ms-win-crt-heap-l1-1-0.dll
            提供了堆内存管理相关的函数，如 malloc, free 等。

        api-ms-win-crt-conio-l1-1-0.dll
            包含了控制台输入输出相关的函数，如 _getch, _cprintf 等。

        api-ms-win-core-sysinfo-l1-1-0.dll
            提供了系统信息查询相关的函数。

        api-ms-win-core-processenvironment-l1-1-0.dll
            包含了进程环境相关的函数，如环境变量的获取和设置。

        api-ms-win-core-libraryloader-l1-1-0.dll
            提供了 DLL 加载和卸载相关的函数。

### 参考

[深入解析VC Runtime：什么是vcruntimeXXX.dll和api-ms-win-crt-runtime-X-X-X.dll？\_vcruntimes-CSDN博客](https://blog.csdn.net/hebhljdx/article/details/139925402)