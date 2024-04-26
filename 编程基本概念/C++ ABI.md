
ABI 就是两个二进制程序模块之间的接口。作为一个概括性的描述，已经足够了。但是让人感觉到有些空洞。

[彻底理解C++ABI](https://zhuanlan.zhihu.com/p/692886292)
主要考虑
1. Object File Format（目标文件格式），
2. Data Representation（数据表示）， 
3. Function Calling Convention（函数调用约定）
4. Runtime Library（运行时库）等因素。
### CPU & OS
x64 平台上主要有两套常用的 ABI：

- 用于 64 位 Windows 操作系统上的 [Windows x64 ABI](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/cpp/build/x64-software-conventions%3Fview%3Dmsvc-170)
- 用于 64 位 Linux 以及一众 UNIX-like 的操作系统上的 [x86-64 System V ABI](https://link.zhihu.com/?target=https%3A//gitlab.com/x86-psABIs/x86-64-ABI)

从一个动态库里面调用某个函数可以简单的看成下面这个三个步骤：

- 按照某种格式解析动态库
- 根据符号名从解析结果中查找函数地址
- 函数参数传递，调用函数

#### Object File Format
以何种格式解析动态库？这就是 ABI 中对 Object File Format 的规定起作用的地方了。如果你希望自己写一个链接器，那么最后生成的可执行文件就需要满足对应平台的格式要求。Windows x64 使用的可执行文件格式是 [PE32+](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/windows/win32/debug/pe-format) ，也就是 PE32（Portable Executable 32-bit）格式的`64`位版本。System V ABI 使用的则是 [ELF（Executable Linkable Format）](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Executable_and_Linkable_Format) 格式的可执行文件。

#### Data Representation
在调用之前，首先得传参对吧。那传参的时候就特别要注意 Data Representation（数据表示）表示的一致性
假设我把下面这个文件编译成动态库

```cpp
struct X{
    int a;
    int b;
};

int foo(X x){
    return x.a + x.b;
}
```

结果后续版本升级导致结构体内容发生变动了，用户代码里面看到的结构体定义变成了

```cpp
struct X{
    int a;
    int b;
    int c;
};
```

然后仍然去尝试链接旧版本代码编译出的动态库，并调用里面的函数

```cpp
int main(){
    int n = foo({1, 2, 3});
    printf("%d\n", n);
}
```

上面的情况属于用户主动变更代码导致的 ODR 违反，**那如果我不主动变更代码，能确保结构体布局的稳定性吗？那这就由 ABI 中 Data Representation 来进行相关保证了。** 例如：规定一些基础类型的大小和对齐， Windows x64 规定`long`是`32`位，而 System V 则规定`long`是`64`位。规定`struct`和`union`的大小和对齐等等。

#### Function Calling Convention
接下来就到函数传参这一步了。我们知道，函数不过就是一段二进制数据，执行函数其实就是跳转到函数的入口地址，然后执行那一段代码，最后执行完了再跳转回来就行了。而传参无非就是找一块地方，存放数据，**使得调用前后都能访问到这个地方来取数据**。有哪些位置可以选择呢？主要有下面四个选项：

- global（全局变量）
- heap（堆）
- register（寄存器）
- stack（栈）

具体来说，调用约定规定下面这些内容：

- 函数参数传递顺序，从左到右还是从右到左？
- 函数参数和返回值传递的方式，通过栈还是寄存器？
- 哪些寄存器在调用者调用前后是保持不变的？
- 谁负责清理栈帧，调用者还是被调用者？
- 如何处理 C 语言的 [variadic](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/variadic) 函数？

#### C++ Standard
我们都知道 C++ 标准没有明确规定 ABI，但并不是完全没有规定，它对于编译器的实现是有一些要求的，例如：

- 结构体成员地址按照声明顺序 [递增](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/struct%23Explanation)，这保证了编译器不会对结构体成员进行重新排序
- 满足 [Standard Layout](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/language/data_members%23Standard-layout) 约束的结构体需要与相应的 C 结构体布局兼容
- 满足 [Trivially Copyable](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/types/is_trivially_copyable) 约束的结构体可以使用`memmove`或者`memcpy`进行拷贝得到一个完全相同的全新对象

另外，由于 C++ 一直在推出新的版本。同一份代码，我使用新标准和旧标准分别进行编译，得到的结果相同吗（不考虑使用宏控制 C++ 版本进行条件编译的影响）？这就要看 C++ 标准层面对 ABI 兼容性的保证了，事实上，C++ 标准尽可能的保证**向后兼容性**。也就是说，两段代码，使用旧标准和新标准编译出来的代码是完全一样的。

然而，也有极少数的例外，例如（我只找得到这些，欢迎评论区补充）：

- C++17 把`noexcept`作为函数类型的一部分，这会影响函数最后生成的 mangling name
- C++20 引入的`no_unique_address`，MSVC 目前仍然没直接支持，因为会导致 ABI Broken


成员函数作为回调函数， 标准库中有一些仿函数，里面什么成员都没有，只有一个`operator()`等


### Compiler Specific

C++ 中的一些抽象最终是要落实到实现上的，而标准有没有规定如何实现，那这部分内容就由编译器自由发挥，例如：

- name mangling 的规则（为了实现函数重载和模板函数）
- 复杂类型的布局（例如含有虚继承）
- 虚函数表的布局
- RTTI 的实现
- 异常处理


如果编译器对这些部分的实现不同，那么最后不同编译器编译出的二进制产物自然是互不兼容，不能混用的。

在今天，C++ 编译器的 ABI 已经基本得到统一，主流的 ABI 只有两套：

- Itanium C++ ABI，具有公开透明的 [文档](https://link.zhihu.com/?target=https%3A//itanium-cxx-abi.github.io/cxx-abi/abi.html)
- MSVC C++ ABI，并没有官方的文档，这里有一份非正式的 [版本](https://link.zhihu.com/?target=http%3A//www.openrce.org/articles/files/jangrayhood.pdf)
考虑到 C++ 巨大的 codebase，这两套 C++ ABI 已经基本稳定，不会再改动了，**所以我们现在其实可以说 C++ 编译器具有稳定的 ABI**。

在 Linux 平台上，GCC 和 Clang 都使用 Itanium ABI，所以两个编译器编译出来的代码就具有互操作性，可以链接到一起并运行。而在 Windows 平台上，情况则稍微复杂些，默认的 MSVC 工具链使用自己的 ABI。但是除了 MSVC 工具链以外，还有人把 GCC 移植到 Windows 上了，也就是我们熟知的 [MinGW](https://link.zhihu.com/?target=https%3A//www.mingw-w64.org/) 工具链，它使用的仍然是 Itanium ABI。这两套 ABI 互不兼容，编译出来的代码不能直接链接到一起。而 Windows 平台上的 Clang 可以通过编译选项控制使用这两种 ABI 的其中的一种。

### Runtime & Library

一个 C++ 程序依赖的库的 ABI 稳定性。**理想情况下是，对于一个可执行程序，使用新版本的动态库替换旧版本的动态库，仍然不影响它运行。**

三大 C++ 编译器都有自己的标准库

- MSVC 对应的是 [msvc stl](https://link.zhihu.com/?target=https%3A//github.com/microsoft/STL)
- GCC 对应的是 [libstdc++](https://link.zhihu.com/?target=https%3A//github.com/gcc-mirror/gcc/tree/master/libstdc%252B%252B-v3)
- Clang 对应的是 [libc++](https://link.zhihu.com/?target=https%3A//github.com/llvm/llvm-project/tree/main/libcxx)

用户库也要注意ABI稳定
[[虚函数作为库接口的缺点]]