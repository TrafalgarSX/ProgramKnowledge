---
id: MSVC静态库链接分析
aliases: 
tags:
  - static-library
  - MSVC
---
### 研究这个问题的动机
最近有个任务是要提供抗量子算法库给第三方使用，对方要求兼容 visual studio 2008，但是抗量子算法库依赖的 liboqs 纯 C 语言实现的工程在 windows 下编译最低要求是 visual studio 2019，所以我做了个测试发现 visual studio 2019 编译的 liboqs 其实可以被 visual studio 2008 链接（需要额外的 stdbool.h stdint.h 头文件），我疑惑为什么可以，这两个编译器依赖的 CRT 是不同的，所以我研究了下MSVC 静态库链接的原理（通过分析 .lib 文件）。 

note: 不过最后实际上是给了一个导出 C 接口的 DLL，使用 visual studio 2019 编译（/MT 必须设计为 /MT 选项），因为我封装 liboqs 的代码是 cpp11 的， visual studio 2008 支持不了，后续也会涉及到 cpp 编译结果的链接性。这也让我了解到，对于基础类型的接口（int char 指针等）C语言接口的二进制兼容性十分强大，后续对于这种无理的编译器兼容应该都使用这种方式提供。

### 我的跨编译器静态库链接测试
liboqs 默认使用 /MT 选项的 visual studio 2019 编译静态库，然后使用 cmakelist.txt 生成 visual studio 2008 的测试工程，在 vs2008 中编译并运行 liboqs 的测试代码，我本以为链接或者运行会有问题， 但是发现并没有出现任何问题， 所以研究了下：

首先 /MT 让我很困惑，按我之前学习的内容，静态库编译是不会链接的，事实上也如此，编译最后的链接静态库的 DLL 可执行程序的时候才会进行链接，而且编译 DLL 和可执行程序的时候可以设置自己的 /MT /MD 编译选项。

这让我产生了一系列疑问：

- 如果我的静态库和 DLL 或者可执行程序选择了不同的编译选项，那静态库的编译选项有什么意义？

- 链接发生在生成 DLL 和可执行程序的过程中，那最后不应该是 DLL 和可执行程序的 /MT /MD 选项覆盖掉静态库的选项吗？ 

- 由于生成静态库的时候不会发生链接，那这个 /MT 选项的作用是什么？ 

- 最后生成 DLL 和可执行程序的时候，静态库链接的 CRT 到底是由自己的 /MT 编译选项决定还是 DLL 和可执行程序的？ 引用的符号是哪个 CRT 的？

这个问题通过我编译了几个不同 /MT /MD 编译选项编译的静态库对比了下， 对比的内容是 xxx.lib 的  dumpin /ALL xxx.lib 的输出。

```cpp
int add(int a, int b)
{
    const char *str = "Hello, World!";

    char buffer[50];
    memset(buffer, 0, sizeof(buffer));
    strncpy(buffer, str, sizeof(buffer) - 1);
    return a + b;
}
```

经过对比发现，同样是上面的代码，使用 /MT /MD 不同选项编译出来的静态库主要有两处不同：

1. PE 文件中 .drectve 段的不同： 
使用 /MD 编译的 .drectve 段的部分内容是：
```cpp
   Linker Directives
   -----------------
   /DEFAULTLIB:MSVCRT
   /DEFAULTLIB:OLDNAMES
```

使用 /MT 编译的 .drectve 段的部分内容是：
```cpp
   Linker Directives
   -----------------
   /DEFAULTLIB:LIBCMT
   /DEFAULTLIB:OLDNAMES
```

可以看到前一个是 CRT 的动态导入库，后一个 libcmt 是 CRT 的静态库，是将本机 CRT 启动静态链接到你的代码，所以实际上 /MT /MD 编译选项将静态库的链接信息存储到了 .obj 文件的 .drectve 段中

**note: 
      .drectve 段的作用是存储链接指示信息，它的内容是编译器传递给链接器的指令（Directive），即编译器希望告诉链接器应该怎样链接这个目标文件。**

2. PE 文件中引用的外部符号不同：
使用 /MD 编译的符号表内容是：
```

COFF SYMBOL TABLE
...
00C 00000000 UNDEF  notype       External     | __imp_strncpy
...

```

使用 /MT 编译的符号表内容是：
```

COFF SYMBOL TABLE
...
00C 00000000 UNDEF  notype       External     | strncpy
...

```

可以看到动态链接 CRT 的引用的符号实际上是 `__imp_strncpy` 导入库版本的符号，与 /MT 编译引用的外部符号不一样，这同样带来了一个有点：
如果同一个 DLL 或者可执行程序链接使用 /MT /MD 编译的两个静态库，不会出现符号冲突的问题，使用 /MT 的静态链接 libcmt 并使用其中的符号， 使用 /MD 使用 msvcrt 中的符号。

当然这里说的静态库有个前置要求：必须是纯 C 编写的库，如果是 CPP 的话，情况大不一样，可能编译通过，也可能报`# LNK4098 defaultlib '_library_' conflicts with use of other libs`警告， 甚至出现编译错误，符号冲突等 error。 后续我展示下 cpp 文件编译结果 .obj 文件的 .drectve 段内容，就可以看出，cpp 链接指示信息更多，更详细。

#### `.drectve` 段中链接指示信息可以自己添加吗？
上面的对比结果又让我有了新的疑问：

/MT /MD 可以在 `.drectve` 中添加链接静态库时的链接指示信息， 那我有什么其他手段自己添加吗？  比如自己添加 Linker Directives 中的 /DEFAULTLIB:xxx 这样我就可以提供一个静态库，当链接这个静态库的时候就不用把该静态库依赖的库信息在链接选项中写一份了。

答案是： 有的。
方式一：通过 `#pragma comment(lib, "bcclib.lib")`
```cpp
#pragma comment(lib, "bcclib.lib")
int bcc();

int add(int a, int b)
{
    bcc();
    const char *str = "Hello, World!";

    char buffer[50];
    memset(buffer, 0, sizeof(buffer));
    strncpy(buffer, str, sizeof(buffer) - 1);
    return a + b;
}
```

这中方式会将  /DEFAULTLIB:bcclib 添加到 Linker Directives 中。

方式二： 通过链接器选项 /DEFAULTLIB:library 添加。

注意这里 /DEFAULTLIB 后面库的搜索路径和你直接链接库的搜索路径应该是一样的，我没有找到对应的文档，但是测试了下，起码是包含用户自己设置的库搜索路径和系统路径的。

#### 为什么 vs2008 使用 vs2019 的纯 C 静态库可以通过编译、链接和测试

**note: 
    有个前提知识， 在 Visual Studio 2015 中，CRT 已重构为新的二进制文件。 通用 CRT (UCRT) 包含通过标准 C99 CRT 库导出的函数和全局函数。 UCRT 现为 Windows 组件，作为 Windows 10 及更高版本的一部分提供。可以在 Visual Studio 2015 及更高版本支持的任何 Windows 版本上使用 UCRT。
	也就是说 vs2015 之后（含）的 C 运行时都是 libucrt.lib 或者 ucrt.lib(ucrtbase.dll 的导入库)
    vs2008 的 C 运行时是 libcmt 和 msvcrt.lib(导入库)

对于我的 liboqs 静态库跨编译器测试，还有几个问题是 
- vs2019 编译的纯 C 静态库是使用 /MT 编译的，那它应该使用的是 libucrt.lib， 为什么在 /DEFAULTLIB 后写的是 libcmt ？ 
- visual studio 2008 对应的部分也是 libcmt ，但是应该使用的 vs2008 版本的 libcmt， 为什么会可以被 vs2019 的工程链接，并正常使用？

**第一个问题的解答是**：
的确 vs2015 以后使用的 C 运行时都是 libucrt.lib 或者 ucrt.lib，但是仍然保留了 libcmt.lib 和 msvcrt.lib， 后者这两个库的作用不再是运行时库了，而是**实现 CRT 初始化和终止的库**。

而且后面两个库中又链接了 libucrt.lib 和 ucrt。这可以通过查看 libcmt.lib 的 .drectve 段看出：
```
   Linker Directives
   -----------------
   /merge:.CRT=.rdata
   /alternatename:__acrt_initialize=__scrt_stub_for_acrt_initialize
   /alternatename:__acrt_uninitialize=__scrt_stub_for_acrt_uninitialize
   /alternatename:__acrt_uninitialize_critical=__scrt_stub_for_acrt_uninitialize_critical
   /alternatename:__acrt_thread_attach=__scrt_stub_for_acrt_thread_attach
   /alternatename:__acrt_thread_detach=__scrt_stub_for_acrt_thread_detach
   /alternatename:_is_c_termination_complete=__scrt_stub_for_is_c_termination_complete
   /alternatename:__vcrt_initialize=__scrt_stub_for_acrt_initialize
   /alternatename:__vcrt_uninitialize=__scrt_stub_for_acrt_uninitialize
   /alternatename:__vcrt_uninitialize_critical=__scrt_stub_for_acrt_uninitialize_critical
   /alternatename:__vcrt_thread_attach=__scrt_stub_for_acrt_thread_attach
   /alternatename:__vcrt_thread_detach=__scrt_stub_for_acrt_thread_detach
   /defaultlib:kernel32.lib  *** 这里指示要链接 kernel32.lib
   /disallowlib:msvcrt.lib  *** 这里指示不要链接 msvcrt.lib
   /disallowlib:msvcrtd.lib
   /defaultlib:libcmt.lib     ***
   /disallowlib:libcmtd.lib
   /disallowlib:vcruntime.lib
   /disallowlib:vcruntimed.lib
   /defaultlib:libvcruntime.lib
   /disallowlib:libvcruntimed.lib
   /disallowlib:ucrtd.lib
   /defaultlib:libucrt.lib  *** 这里指示要链接 libucrt.lib
   /disallowlib:libucrtd.lib
```

**第二个问题的解答是：**
由于 vs2019 编译的 liboqs 静态库的 /DEFAULTLIB 仍然链接的是 libcmt， 虽然原本的意思是链接 vs2019 的 libcmt, 也就是**实现 CRT 初始化和终止的库**， 但是由于和 vs2008 要链接的 libcmt C运行时库名字一样，所以最后 vs2008 编译测试程序的时候将 vs2019 编译的 liboqs 和 vs2008 的 libcmt  C 运行时库链接到了一起。

为什么可以这样呢？  很明显是由于 C ABI 接口的稳定性和 liboqs 项目虽然要求用支持 c11 编译标准的编译器，但实际上 liboqs 的实现并没有使用 vs2008 的 C 运行时不支持的接口，考虑到 vs2008 完整支持的 C 标准是 C89 ，可见 liboqs 的代码兼容性还是可以的。当然我看了 liboqs 的代码实现，标准至少是 C99 的，而 VS2019 好像才完整支持 C99的特性。

所以这次静态库跨编译器链接并成功运行的测试实际上也只是凑巧，如果某个库使用了 vs2008 C 运行时不直接的接口，那就不能成功运行了。

#### CPP 编译结果的 .drectve 
```cpp
     Linker Directives
   -----------------
   /FAILIFMISMATCH:_MSC_VER=1900      ** vs2015 之后编译器编译的 .obj 文件这部分都是 1900
   /FAILIFMISMATCH:_ITERATOR_DEBUG_LEVEL=0  ** 调试等级
   **手动注释 /MT 静态 release  /MD 是 MT_StaticDebug
   /FAILIFMISMATCH:RuntimeLibrary=MT_StaticRelease   
   /DEFAULTLIB:libcpmt
   /FAILIFMISMATCH:_CRT_STDIO_ISO_WIDE_SPECIFIERS=0
   /alternatename:_Avx2WmemEnabled=_Avx2WmemEnabledWeakValue
   /FAILIFMISMATCH:annotate_string=0
   /FAILIFMISMATCH:annotate_vector=0
   /DEFAULTLIB:LIBCMT
   /DEFAULTLIB:OLDNAMES
```

上面是 cpp 文件编译结果 .obj 的  .drectve 段的部分内容，可以看到这里的链接指示信息指出了要链接 cpp 的 .obj 文件， 所有 cpp 的 .obj 文件的 /FAILIFMISMATCH 都得相同，否则链接就会失败。

当然这是由 C 文件编译结果 .obj 文件没有这些字段，C 和 CPP 又可以混编推断出来的只有有 /FAILIFMISMATCH 字段的会进行比较，也就是所有 cpp 的 .obj 文件会进行比较。

这就导致 CPP 的编译结果 .obj 也好， .obj 的集合：静态库也好， 都是不可能实现 vs2008 与 vs2019 的跨编译器链接的。

**事实上微软也明确了不同版本 visual studio 之间的 C++ 二进制兼容性**：
  Visual Studio 2013 及更早版本中的 Microsoft C++ (MSVC) 编译器工具集不保证主版本间的二进制兼容性。 **无法链接**由这些工具集的不同版本生成的**对象文件、静态库、动态库和可执行文件**。 **ABI、对象格式和运行时库不兼容。**

  我们在 Visual Studio 2015 及更高版本中改变了此行为。 由其中任一版本的编译器编译的运行时库和应用具有二进制兼容性。 这反映在 C++ 工具集主版本号中，对于自 Visual Studio 2015 以来的所有版本，该版本号都以 14 开头。 （对于 Visual Studio 2015、2017、2019 和 2022，工具集版本分别为 v140、v141、v142 和 v143）。 假设你具有 Visual Studio 2015 生成的第三方库。 你仍可在 Visual Studio 2017、2019 或 2022 生成的应用程序中使用它们。 无需使用匹配工具集重新编译。 最新版本的 Microsoft Visual C++ 可再发行程序包（可再发行程序包）适用于所有版本。

我之前一直忽略了这里强调的是 c++ 的二进制兼容性。实际上 C 的二进制兼容性一直都挺好。
#### 测试总结
所以 C 静态库的跨编译器链接还是不可靠的，最佳的方式还是通过 导出 C 函数的 DLL （/MT 编译）方式来提供库，这样既可以利用 C ABI 的稳定性提供跨编译器使用第三方库的便利性，又可以利用新的、统一的编译器去编译需要的库，这样就可以使用新的语言特性（尤其是 cpp 的新标准）或者满足库的最低编译器版本要求（liboqs 要求最低是 vs2019 编译）同时保证 ABI、运行时库的兼容。

使用 /MT 编译选项去编译 DLL 是因为在编译 DLL 的时候就完成运行时库的链接，如果使用 /MD 的话，无论链接 DLL 的可执行程序使用 /MT 还是 /MD， DLL 都会使用可执行程序编译器的运行时，同时我们知道 **Visual Studio 2013 及更早版本中的 Microsoft C++ (MSVC) 编译器工具集不保证运行时库的兼容性**，那我们使用 vs2015 以后的编译器编译的动态库就不能被 vs2013 之前的编译器编译的可执行程序链接了。

**TODO 这里说法对吗？** 我应该测试下 vs2008 编译器编译的可执行程序（/MT 编译）链接 vs2019 /MD 编译的 DLL 的情况。

1. 显然旧编译器编译的可执行程序使用 /MD 编译，再链接链接 vs2019 /MD 编译的 DLL 的情况是不可行的，因为这样会使用同样的运行时，但是两个需要的运行时是不兼容的。

2. 那如果 显然旧编译器编译的可执行程序使用 /MT 编译，再链接链接 vs2019 /MD 编译的 DLL 的情况呢？
   这种情况好像没有什么问题？因为两个使用的是不同的运行时， DLL 会加载自己需要的运行时（ DLL 文件里有记录）。


**注意： 如果 DLL 之间或者 DLL 与可执行程序之间使用不同的运行时， 你不能够在这些DLL之间或 DLL 与可执行程序之间相互传递使用一些资源。**

### c/c++ 运行时库知识

#### C 运行时 `.lib` 文件
在 Visual Studio 2015 中，CRT 已重构为新的二进制文件。 通用 CRT (UCRT) 包含通过标准 C99 CRT 库导出的函数和全局函数。

下表列出了实现 UCRT 的库。

| 库                | 关联的 DLL           | 特征                         | 选项         | 预处理器指令 |
| ---------------- | ----------------- | -------------------------- | ---------- | ------ |
| _`libucrt.lib`_  | 无                 | 将 UCRT 静态链接到你的代码。          | **`/MT`**  | `_MT`  |
| _`libucrtd.lib`_ | 无                 | 用于静态链接的 UCRT 调试版本。 不可再发行。  | **`/MTd`** | %>     |
| _`ucrt.lib`_     | _`ucrtbase.dll`_  | UCRT 的 DLL 导入库。            | **`/MD`**  | %>     |
| _`ucrtd.lib`_    | _`ucrtbased.dll`_ | UCRT 调试版本的 DLL 导入库。 不可再发行。 | **`/MDd`** | .- .   |
**vcruntime 库包含 Visual C++ CRT 实现特定的代码**：异常处理和调试支持、运行时检查和类型信息、实现的详细信息和某些扩展的库函数。 vcruntime 库版本需要与所使用的编译器版本匹配。

此表列出了实现 vcruntime 库的库。

|Library  图书馆|关联的 DLL|特征|选项|预处理器指令|
|---|---|---|---|---|
|_`libvcruntime.lib`_|无|静态链接到你的代码。|**`/MT`**|`_MT`|
|_`libvcruntimed.lib`_|无|用于静态链接的调试版本。 不可再发行。|**`/MTd`**|%>|
|_`vcruntime.lib`_|_`vcruntime<version>.dll`_|vcruntime 的 DLL 导入库。|**`/MD`**|%>|
|_`vcruntimed.lib`_|_`vcruntime<version>d.dll`_|调试 vcruntime 的 DLL 导入库。 不可再发行。|**`/MDd`**|.- .|

此代码**处理 CRT 启动、内部每线程数据初始化和终止**。它**特定于所使用的编译器版本**。此库**始终是静态链接的**，即使在使用动态链接的 UCRT 时也是如此。

此表列出了**实现 CRT 初始化和终止的库。**

| 库                | 特征                                                         | 选项         | 预处理器指令 |
| ---------------- | ---------------------------------------------------------- | ---------- | ------ |
| _`libcmt.lib`_   | 将本机 CRT 启动静态链接到你的代码。                                       | **`/MT`**  | `_MT`  |
| _`libcmtd.lib`_  | 静态链接本机 CRT 启动的调试版本。 不可再发行。                                 | **`/MTd`** | %>     |
| _`msvcrt.lib`_   | 与 DLL UCRT 和 vcruntime 一起使用的本机 CRT 启动的**静态库**。             | **`/MD`**  | %>     |
| _`msvcrtd.lib`_  | 与 DLL UCRT 和 vcruntime 一起使用的本机 CRT 启动调试版本的静态库。 不可再发行。      | **`/MDd`** | .- .   |
| _`msvcmrt.lib`_  | 与 DLL UCRT 和 vcruntime 一起使用的本机和托管混合 CRT 启动的静态库。            | **`/clr`** |        |
| _`msvcmrtd.lib`_ | 与 DLL UCRT 和 vcruntime 一起使用的本机和托管混合 CRT 启动调试版本的静态库。 不可再发行。 | **`/clr`** |        |
#### C++ 标准库 (STL) `.lib` 文件

| C++ 标准库          | 特征                                    | 选项         | 预处理器指令 |
| ---------------- | ------------------------------------- | ---------- | ------ |
| _`libcpmt.lib`_  | 多线程, 静态链接                             | **`/MT`**  | `_MT`  |
| _`msvcprt.lib`_  | 多线程动态链接（_`msvcp<version>.dll`_ 的导入库）  | **`/MD`**  | %>     |
| _`libcpmtd.lib`_ | 多线程, 静态链接                             | **`/MTd`** | %>     |
| _`msvcprtd.lib`_ | 多线程动态链接（_`msvcp<version>d.dll`_ 的导入库） | **`/MDd`** | .- .   |

### ref
1. [C++ 二进制兼容性 2015-2022 \| Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/porting/binary-compat-2015-2017?view=msvc-170&viewFallbackFrom=vs-2019)

2. [跨 DLL 边界传递 CRT 对象可能的错误 \| Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/potential-errors-passing-crt-objects-across-dll-boundaries?view=msvc-170)

3. [C 运行时 (CRT) 和 C++ 标准库 (STL) .lib 文件 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/crt-library-features?view=msvc-170)

