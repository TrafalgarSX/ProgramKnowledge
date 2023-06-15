dll中有两种导出函数声明的方式：①__declspec(dllexport)；②模块定义文件（.def）
### def
---
.def文件为链接器提供有关被链接程序的导出、属性及其他方面的信息。

**模块定义 (.def) 文件**为链接器提供有关导出、特性的信息以及有关要链接的程序的其他信息。 .def 文件在构建 DLL 时最有用。 因为可以使用 MSVC 链接器选项来代替模块定义语句，所以通常不需要 .def 文件。 你还可以使用 `__declspec(dllexport)` 作为指定导出函数的一种方式。

可以在链接器阶段使用 /DEF（指定模块定义文件）链接器选项调用 .def 文件。

如果你正在构建一个没有导出的 .exe 文件，使用 .def 文件将使你的输出文件增大且加载速度变慢。

相关def文件规则和导出符号内容
[模块定义 (.Def) 文件 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/build/reference/module-definition-dot-def-files?view=msvc-170)

### What is `__declspec` and when do i need to use it?
---
You can export data, functions, calsses, or class meember functions from a DLL using the `__declspec(dllexport)` keyword. `__declspec(dllexport)` adds the export directive to the object file **so you do not need to use a .def file.**

This convenience is most apparent when trying to export decorated C++ function names, Because there is no standard specification for name decoration, the name of an exported funciton might change between compiler versions. if you use `__declspec(dllexport)`, recompiling the DLL and dependent .exe files is necessary only to account for any naming convention changes.

To export functions, the **`__declspec(dllexport)`** keyword must appear to the left of the calling-convention keyword, if a keyword is specified. For example:
```C++
__declspec(dllexport) void __cdecl Function1(void);
```

To export all of the public data members and member functions in a class, the key
```C++
class __declspec(dllexport) CExampleExport : public CObject
{ ... class definition ... };
```


### C++ visibility
[Visibility - GCC Wiki](https://gcc.gnu.org/wiki/Visibility)
[Explicitly Linking to Classes in DLL's | CodeGuru](https://www.codeguru.com/windows/explicitly-linking-to-classes-in-dlls/)