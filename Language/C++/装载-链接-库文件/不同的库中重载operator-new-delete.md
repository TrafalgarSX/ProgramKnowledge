
首先声明，不要这样做， 我只是为了观察现象。
### DLL 中定义 operator new/delete
#### windows
如果在DLL中定义了 operator new/delete， DLL 的编译模块会使用这个定义。如果 program 也定义了 operator new/delete, program 会使用自己定义的， 如果 Program 没有自己定义，它会使用系统库的，反正不会使用 DLL 中定义的。

按规定如果要重定义 operator new/delete, 整个程序中只应该定义一个。

而且windows 下 DLL重定义了 operator new/delte 是不能导出的，我在vs2022下编译会出现错误。

不过可以声明在头文件中，并加 inline, 这样 Program 引入头文件就会使用 DLL 的 operator new/delete， 但是这样声明会有警告，同样不推荐。

下面这个回答解释的很详细
[c++ - new and delete operator overloading for dll - Stack Overflow](https://stackoverflow.com/questions/11846511/new-and-delete-operator-overloading-for-dll)

```markdown
On one hand, you cannot have the definitions of your new/delete operators compiled inside the DLL because the overloaded new/delete cannot be linked dynamically (this is because new/delete might be needed during static initialization, prior to loading the DLL, so you would have inconsistent new/delete operators before and after the loading of the DLL, and that's undefined behavior).

On the other hand, you cannot just put your new/delete operator definitions in your DLL header files, because they would need to be marked `inline` in order to satisfy the One Definition Rule (ODR), which, in turn, doesn't satisfy the above clause. The requirement for them to not be marked `inline` is probably there because a function definition marked `inline` has "no linkage", resulting in each translation unit using its own compiled version of it (or as inline expansions), which, normally is OK, but not for dynamic memory allocation.
```

[c++ - Does DLL new/delete override the user code new/delete? - Stack Overflow](https://stackoverflow.com/questions/5802005/does-dll-new-delete-override-the-user-code-new-delete)
#### linux
如果动态库和program 都定义了 operator new/delete, 最后会使用 program 定义的，

如果动态库定义了 operator new/delete, program 没有定义，最后会使用动态库定义的，program 并不会使用默认全局的

gcc or clang 表现一致

这两条都与 windows 不一样

**所以这也属于未定义的行为, 尽量避免**

### program 中重定义 operator new/delete
#### windows
动态库不会使用program 中重定义的 operator new/delete, 但是静态库会。

#### linux

动态库定义的 operator new/delete 不会被使用， 动态库和静态库都会使用 program 定义的。

**所以这也属于未定义的行为, 尽量避免**


