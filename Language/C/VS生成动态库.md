###  __declspec(dllexport)
```c
#ifdef _EXPORTING
#define API_DECLSPEC __declspec(dllexport)
#else
#define API_DECLSPEC __declspec(dllimport)
#endif
​API_DECLSPEC void HelloWorld();
```

__declspec(dllexport)用于导出符号，也就是定义该函数的dll；__declspec(dllimport)用于导入，也就是使用该函数。

因为这个头文件既要被定义该函数的dll包含，也要被使用该函数的程序包含，当被前者包含时我们希望使用 __declspec(dllexport) 定义函数，当被后者包含时我们希望使用 dllimport。

#### dllimport dllexport 的作用
导入全局、静态或者类成员变量需要__declspec(dllimport)。
```c
#define DllImport __declspec(dllimport) 
DllImport int j;
```

不使用 **declspec(dllimport) 也能正确编译代码，但使用** declspec(dllimport) 使编译器可以生成更好的代码。编译器之所以能够生成更好的代码，是因为它可以确定函数是否存在于 DLL 中，这使得编译器可以生成跳过间接寻址级别的代码，而这些代码通常会出现在跨 DLL 边界的函数调用中。但是，必须使用 __declspec(dllimport) 才能导入 DLL 中使用的变量。


__declspec(dllexport)是用于避免需要自己写 def 文件的。

编译器会为被 declspec(dllexport)修饰的函数**自动添加一个导出函数入口。** 如果你在其他模块中包含 declspec(dllexport)的头文件，这些项目的导出表中也会生成一个同名导出函数。

  