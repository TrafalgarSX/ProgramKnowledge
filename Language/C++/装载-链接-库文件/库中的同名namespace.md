
### 静态库
#### windows
静态库会与program 中的同名namespace 合并， 如果声明同名变量之类的会冲突。
#### linux
静态库会与program 中的同名namespace 合并， 如果声明同名变量之类的会冲突。

这个不同平台表现一致，符合 namespace 的要求： 不同编译单元的会合并， 静态库会和 program 一起编译，都属于一起编译的编译单元。

### 动态库
#### windows
DLL不会合并 program 中的同名namespace，并且找不到DLL中定义的符号(DLL独有的变量会找不到)，如果定义了同名变量，也不会编译错误。如果多个DLL 有同样的namespace ,应该只会使用一个（谁先加载用谁）
#### linux

动态库 中同名的namespace 不会和 program 中同名namespace 的同名变量等冲突，而是会被 program 中的覆盖， 会使用 program 中的同名变量； 但是如果 program 中同名的namespace 没有的变量，则 program 可以找到动态库中的 不同名变量。

**所以 不同编译模块中定义同名的namespace 属于 未定义行为，尽量避免这样的操作。**