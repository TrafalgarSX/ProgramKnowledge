**将 C 运行时 (CRT) 对象（如文件句柄、区域设置和环境变量）传入或传出 DLL 时，代码可能会出现错误**。 如果 DLL 以及任何调入 DLL 的文件使用不同的 CRT 库副本，则跨 DLL 边界的函数调用可能会导致意外行为。

如果你分配内存（使用 new 或 malloc 显式分配，或使用 strdup、strstreambuf::str 等隐式分配），然后跨 DLL 边界传递要释放的指针，则会出现相关问题。 如果 DLL 及其使用者使用不同的 CRT 库副本，则此类指针会导致内存访问冲突或堆损坏。

调试期间，此问题的另一种症状可能是输出窗口中的错误，例如 HEAP[]: Invalid Address specified to RtlValidateHeap(#,#)

### 原因

**CRT 库的每个副本都具有单独且完全不同的状态**，且**按应用或 DLL** 保存于线程本地存储中。

CRT 对象（例如，文件句柄、环境变量和区域设置）仅对在其中分配或设置这些对象的应用或 DLL 中的 CRT 副本有效。 当 DLL 及其客户端使用不同的 CRT 库副本时，你无法期望在跨 DLL 边界传递这些 CRT 对象时正确地使用它们。

这对 Visual Studio 2015 及更高版本中的通用 CRT 之前的 CRT 版本尤其如此。 对于使用 Visual Studio 2013 或更低版本生成的 Visual Studio 的每个版本，均有特定于版本的 CRT 库。 CRT 的内部实现详细信息（如数据结构和命名约定）在每个版本中都不同。 为某个版本的 CRT 所编译的代码一直都无法动态链接到不同版本的 CRT DLL。 尽管它有时能起作用，但多半是运气使然，而非设计如此。

**CRT 库的每个副本都有其自己的堆管理器。 如果分配一个 CRT 库中的内存并跨 DLL 边界传递由 CRT 库的不同副本释放的指针，则会导致堆损坏**。 如果 DLL 跨 DLL 边界传递 CRT 对象或分配在 DLL 的外部释放的内存，则 DLL 的客户端必须使用与 DLL **相同的 CRT 库副本。**

仅在 DLL 及其客户端都在加载时链接到相同版本的 CRT DLL 时，二者才使用相同的 CRT 库副本。 由于 Visual Studio 2015 及更高版本使用的通用 CRT 库的 DLL 版本现在是一个集中部署的 Windows 组件 (`ucrtbase.dll`)，因此它对使用 Visual Studio 2015 及更高版本生成的应用也是一样的。 但是，**即使在 CRT 代码完全相同时，你也不能将分配给一个堆的内存移交给使用其他堆的组件**。

### 示例：跨 DLL 边界传递文件句柄

#### 说明

此示例跨 DLL 边界传递文件句柄。

使用 `/MD` 生成 DLL 和 .exe 文件，以便它们共享单个 CRT 副本。

如果使用 `/MT` 重新生成它们以使其使用单独的 CRT 副本，则运行生成的 `test1Main.exe` 将导致访问冲突。

DLL 源文件 `test1Dll.cpp`：

```cpp
// test1Dll.cpp
// compile with: cl /EHsc /W4 /MD /LD test1Dll.cpp
#include <stdio.h>
__declspec(dllexport) void writeFile(FILE *stream)
{
   char   s[] = "this is a string\n";
   fprintf( stream, "%s", s );
   fclose( stream );
}
```

可执行源文件 test1Main.cpp：
```cpp
// test1Main.cpp
// compile with: cl /EHsc /W4 /MD test1Main.cpp test1Dll.lib
#include <stdio.h>
#include <process.h>
void writeFile(FILE *stream);

int main(void)
{
   FILE  * stream;
   errno_t err = fopen_s( &stream, "fprintf.out", "w" );
   writeFile(stream);
   system( "type fprintf.out" );
}
```

### 示例：跨 DLL 边界传递环境变量

### 说明

此示例跨 DLL 边界传递环境变量。

DLL 源文件 `test2Dll.cpp`：

```cpp
// test2Dll.cpp
// compile with: cl /EHsc /W4 /MT /LD test2Dll.cpp
#include <stdio.h>
#include <stdlib.h>

__declspec(dllexport) void readEnv()
{
   char *libvar;
   size_t libvarsize;

   /* Get the value of the MYLIB environment variable. */
   _dupenv_s( &libvar, &libvarsize, "MYLIB" );

   if( libvar != NULL )
      printf( "New MYLIB variable is: %s\n", libvar);
   else
      printf( "MYLIB has not been set.\n");
   free( libvar );
}
```

可执行源文件 `test2Main.cpp`：

```cpp
// test2Main.cpp
// compile with: cl /EHsc /W4 /MT test2Main.cpp test2dll.lib
#include <stdlib.h>
#include <stdio.h>

void readEnv();

int main( void )
{
   _putenv( "MYLIB=c:\\mylib;c:\\yourlib" );
   readEnv();
}
```

Output

```
MYLIB has not been set.
```

**如果使用 `/MD` 同时生成 DLL 和 EXE 文件以便仅使用一个 CRT 副本**，则此程序将成功运行并产生以下输出：

Output

```
New MYLIB variable is: c:\mylib;c:\yourlib
```

### 参考
[跨 DLL 边界传递 CRT 对象可能的错误 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/potential-errors-passing-crt-objects-across-dll-boundaries?view=msvc-170#example-pass-file-handle-across-dll-boundary)