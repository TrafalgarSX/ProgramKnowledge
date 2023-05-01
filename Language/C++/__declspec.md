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

