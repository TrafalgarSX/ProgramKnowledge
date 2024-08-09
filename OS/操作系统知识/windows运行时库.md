### C 运行时 `.lib` 文件

在 Visual Studio 2015 中，CRT 已重构为新的二进制文件。 **通用 CRT (UCRT) 包含通过标准 C99 CRT 库导出的函数和全局函数**。 

UCRT 现为 Windows 组件，作为 Windows 10 及更高版本的一部分提供。 静态库、DLL 导入库和 UCRT 的头文件现在 Windows SDK 中提供。

| 库                | 关联的 DLL           | 特征                         | 选项         | 预处理器指令                  |
| ---------------- | ----------------- | -------------------------- | ---------- | ----------------------- |
| _`libucrt.lib`_  | 无                 | 将 UCRT 静态链接到你的代码。          | **`/MT`**  | `_MT`                   |
| _`libucrtd.lib`_ | 无                 | 用于静态链接的 UCRT 调试版本。 不可再发行。  | **`/MTd`** | `_DEBUG`，`_MT`          |
| _`ucrt.lib`_     | _`ucrtbase.dll`_  | UCRT 的 DLL 导入库。            | **`/MD`**  | `_MT`，`_DLL`            |
| _`ucrtd.lib`_    | _`ucrtbased.dll`_ | UCRT 调试版本的 DLL 导入库。 不可再发行。 | **`/MDd`** | `_DEBUG`, `_MT`, `_DLL` |
#### cpp windows 基础库
vcruntime 库包含 **Visual C++ CRT 实现特定的代码：异常处理和调试支持、运行时检查和类型信息、实现的详细信息和某些扩展的库函数**。 vcruntime 库版本需要与所使用的编译器版本匹配。

| Library               | 关联的 DLL                     | 特征                             | 选项         | 预处理器指令                  |
| --------------------- | --------------------------- | ------------------------------ | ---------- | ----------------------- |
| _`libvcruntime.lib`_  | 无                           | 静态链接到你的代码。                     | **`/MT`**  | `_MT`                   |
| _`libvcruntimed.lib`_ | 无                           | 用于静态链接的调试版本。 不可再发行。            | **`/MTd`** | `_MT`，`_DEBUG`          |
| _`vcruntime.lib`_     | _`vcruntime<version>.dll`_  | vcruntime 的 DLL 导入库。           | **`/MD`**  | `_MT`，`_DLL`            |
| _`vcruntimed.lib`_    | _`vcruntime<version>d.dll`_ | 调试 vcruntime 的 DLL 导入库。 不可再发行。 | **`/MDd`** | `_DEBUG`, `_MT`, `_DLL` |

以上都是操作系统自带的库

**初始化 CRT 的代码是几个库中的一个**，根据 CRT 库是采用静态或动态链接还是本机、托管或混合代码而定。 此代码处理 CRT 启动、内部逐线程数据初始化和终止。 它特定于所用编译器的版本。

**note：**
`mainCRTStartup, WinMainCRTStartup, _DllMainCRTStartup`  这些接口都在 libcmt.lib 或 msvcrt.lib 中， 每次调用 main wmain dllMain 这些函数前会调用相应的代码初始化 CRT， 如果不调用，后续可能会出错。

即使使用动态链接的 UCRT，**该库也始终是静态链接的。**

| 库                    | 特征                                                         | 选项              | 预处理器指令                  |
| -------------------- | ---------------------------------------------------------- | --------------- | ----------------------- |
| _`libcmt.lib`_       | 将本机 CRT 启动静态链接到你的代码。                                       | **`/MT`**       | `_MT`                   |
| _`libcmtd.lib`_      | 静态链接本机 CRT 启动的调试版本。 不可再发行。                                 | **`/MTd`**      | `_DEBUG`，`_MT`          |
| _`msvcrt.lib`_       | 与 DLL UCRT 和 vcruntime 一起使用的本机 CRT 启动的静态库。                 | **`/MD`**       | `_MT`，`_DLL`            |
| _`msvcrtd.lib`_      | 与 DLL UCRT 和 vcruntime 一起使用的本机 CRT 启动调试版本的静态库。 不可再发行。      | **`/MDd`**      | `_DEBUG`, `_MT`, `_DLL` |
| _`msvcmrt.lib`_      | 与 DLL UCRT 和 vcruntime 一起使用的本机和托管混合 CRT 启动的静态库。            | **`/clr`**      |                         |
| _`msvcmrtd.lib`_     | 与 DLL UCRT 和 vcruntime 一起使用的本机和托管混合 CRT 启动调试版本的静态库。 不可再发行。 | **`/clr`**      |                         |
| _`msvcurt.lib`_      | 纯托管 CRT 的**已弃用**静态库。                                       | **`/clr:pure`** |                         |
| _`msvcurtd.lib`_<br> | 纯托管 CRT 调试版本的**已弃用**静态库。 不可再发行。                            | **`/clr:pure`** |                         |

如果您从命令行链接程序而没有指定 C 运行时库的编译器选项，则链接器将使用静态链接的 CRT 库：**libcmt.lib 、 libvcruntime.lib 和 libucrt.lib 。**

libcmt.lib libvcruntime.lib libucrt.lib  /MT
msvcrt.lib vcruntime.lib ucrt.lib  /MD

使用静态链接的 CRT 意味着由 C 运行时库保存的任何状态信息对于 CRT 的该实例而言是本地的。 例如，如果在使用静态链接的 CRT 时使用 [`strtok`](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strtok-strtok-l-wcstok-wcstok-l-mbstok-mbstok-l?view=msvc-170)，则 `strtok` 分析器的位置将与链接到另一个静态 CRT 实例的同一进程（但在不同的 DLL 或 EXE 中）的代码中使用的 `strtok` 状态无关。 相反，动态链接的 CRT 可共享动态链接到 CRT 的进程中的所有代码的状态。 如果使用这些函数的更安全的新版本，则这一问题不适用；例如，`strtok_s` 不存在此问题。

#### 旧C运行时库
msvcrt.dll 是另一个 C运行时库， ~~是旧版本的~~
- C:/Windows/SysWOW64/msvcrt.dll
- C:/Windows/System32/msvcrt.dll

msvcrt.dll 应该是系统自带的库， 系统有些库依赖它。  在vs2015前， 每个编译器自带的应该是 _`msvcrt71.dll`_  这些库， 这才是旧的 C 运行时库， 这些库是有导入库的，可以链接， 系统自带的 msvcrt.dll 没有导入库，下面的 github 链接讲如何制作其导入库

[GitHub - neosmart/msvcrt.lib: msvcrt.lib for linking against msvcrt.dll on all versions of Windows](https://github.com/neosmart/msvcrt.lib)

### C++ 标准库 (STL) `.lib` 文件
| C++ 标准库          | 特征                                    | 选项         | 预处理器指令                  |
| ---------------- | ------------------------------------- | ---------- | ----------------------- |
| _`libcpmt.lib`_  | 多线程, 静态链接                             | **`/MT`**  | `_MT`                   |
| _`msvcprt.lib`_  | 多线程动态链接（_`msvcp<version>.dll`_ 的导入库）  | **`/MD`**  | `_MT`，`_DLL`            |
| _`libcpmtd.lib`_ | 多线程, 静态链接                             | **`/MTd`** | `_DEBUG`，`_MT`          |
| _`msvcprtd.lib`_ | 多线程动态链接（_`msvcp<version>d.dll`_ 的导入库） | **`/MDd`** | `_DEBUG`, `_MT`, `_DLL` |

每个编译器版本有自己的 vcruntime140.dll  msvcp140.dll 

- vcruntime140.dll 是Visual C++ CRT 实现特定的代码：异常处理和调试支持、运行时检查和类型信息、实现的详细信息和某些扩展的库函数
- msvcp140.dll   提供 c++ 标准库的功能

ucrtbase.dll 每个 windows kit 的版本下面有自己的，  在操作系统路径下也有

这两个库 在操作系统路径下也有  
C:/Windows/System32/msvcp140.dll  
C:/Windows/SysWOW64/msvcp140.dll

### 如果应用程序使用多个 CRT 版本，将存在什么问题？

每个可执行映像（EXE 或 DLL）可以具有其自己静态链接的 CRT，或可以动态链接到 CRT。 静态包括在某个映像中或某个映像动态加载的 CRT 版本取决于构建 该 CRT 时采用的工具和库。 
**单个进程可能会加载多个 EXE 和 DLL 映像，每个都有其自己的 CRT。 每个 CRT 可能使用不同的分配器，可能具有不同的内部结构布局，可能使用不同的存储排列方式。**  这意味着，分配的内存、CRT 资源或跨 DLL 边界传递的类可能会导致内存管理、内部静态使用情况或布局解释方面的问题。 例如，如果在一个 DLL 中分配类，但将其传递给另一个 DLL 或由另一个 DLL 删除，那么使用了哪个 CRT 释放器？ 导致的错误程度可以从微小到立即致命，因此建议不要直接传输此类资源。

CRT 库的每个副本都具有**单独且完全不同的状态**，且按应用或 DLL 保存于线程本地存储中。

CRT 对象（例如，文件句柄、环境变量和区域设置）仅对在其中分配或设置这些对象的应用或 DLL 中的 CRT 副本有效。 当 DLL 及其客户端使用不同的 CRT 库副本时，你无法期望在跨 DLL 边界传递这些 CRT 对象时正确地使用它们。

这对 Visual Studio 2015 及更高版本中的通用 CRT **之前**的 CRT 版本尤其如此。 对于使用 Visual Studio 2013 或更低版本生成的 Visual Studio 的每个版本，均有特定于版本的 CRT 库。 **CRT 的内部实现详细信息（如数据结构和命名约定）在每个版本中都不同。** 为某个版本的 CRT 所编译的代码一直都无法动态链接到不同版本的 CRT DLL。 尽管它有时能起作用，但多半是运气使然，而非设计如此。

**CRT 库的每个副本都有其自己的堆管理器**。 如果分配一个 CRT 库中的内存并跨 DLL 边界传递由 CRT 库的不同副本释放的指针，则会导致堆损坏。 如果 DLL 跨 DLL 边界传递 CRT 对象或分配在 DLL 的外部释放的内存，则 DLL 的客户端必须使用与 DLL 相同的 CRT 库副本。

### crt
C runtime 和 C++ runtime 是两个概念。Windows 上 C runtime 早就是 UCRT（Universal C Runtime）了。Windows 是自带 UCRT 的。

MSVC 的 C++ runtime（Visual C++ runtime，主体部分是 `vcruntime<version>.dll` 的那个，其他部分分散另外几个 dll）也要引用 UCRT 的。Windows 没有自带的 C++ runtime。

### 汇编如何链接并调用 CRT
[windows - Can't link ml64 compiled assembly file using link.exe - Stack Overflow](https://stackoverflow.com/questions/76179330/cant-link-ml64-compiled-assembly-file-using-link-exe/76191000#76191000)

这里面我注意到， 即使在汇编代码中设置 `/entry:AnyName`， 修改入口函数， AnyName 中调用了 `_initterm , .CRT$XIA, .CRT$XIZ`, 这里其实就是手动初始化了 CRT

手动初始化CRT原理：
[CRT Initialization | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/c-runtime-library/crt-initialization?view=msvc-170)
### 参考
[Upgrade your code to the Universal CRT | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/porting/upgrade-your-code-to-the-universal-crt?view=msvc-170)

[深入解析VC Runtime：什么是vcruntimeXXX.dll和api-ms-win-crt-runtime-X-X-X.dll？\_vcruntimes-CSDN博客](https://blog.csdn.net/hebhljdx/article/details/139925402)