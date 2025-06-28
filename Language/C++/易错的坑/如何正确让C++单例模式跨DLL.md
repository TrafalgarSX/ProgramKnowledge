
```cpp
#ifdef IN_HEADER
class single{
public:
    static single &get_instance()
    {
        static single instance;
        return instance;
    }
private:
    single() = default;
    single(const single &) = delete;
    single(single &&) = delete;
    single &operator=(const single &) = delete;
    single &operator=(single &&) = delete;
};

#else  

class single{
public:
    static single &get_instance();

private:
    single() = default;
    single(const single &) = delete;
    single(single &&) = delete;
    single &operator=(const single &) = delete;
    single &operator=(single &&) = delete;
};

#endif
```

可参考的回答：

```markdown
嘿嘿嘿，这个坑我在工作中发现很多同事都踩进去过，而且半天出不来，怎么调怎么都不对（看戏（我没有））

先回答“C++单例模式跨DLL是不是就是会出问题”这个问题，后面再解释。以后同事再问我也可以拿这个回答直接给他看，省得费口舌了。

C++单例模式跨DLL是不是就是会出问题——如果没有使用对就会有问题，或者说你只是看网上的各种初学者博客上一顿互抄的文章，简单地在类里使用了一下static变量作为单例，并期望这个单例能跨动态库，那就出问题了。比如一堆博客中都会给出的这段代码就是有问题的：

```cpp
template<typename T>
class Singleton {
public:
    static T& GetReference()
    {
        static T instance;
        return instance;
    }
};
```
由于是模板， 只能在头文件中， 所以如果每个 dll 或者 exe 包含了这个模板头文件， 就会导致每个 dll 或者 exe 就会有一个实例。(每个静态存储区域的变量都会在 dll 或者 exe 的数据段， 加载 dll 或者 exe 的时候会初始化)

```markdown
**static变量是单个编译单元的静态变量，内存将被分配于当前的编译单元中**，此时如果你编可执行文件的代码和编动态库的代码都包含了上述的单例头文件，那么这个“单例”将会出现两个实例，一个实例的内存在可执行文件中，一个实例的内存在动态链接库中，在运行存在于可执行文件中的代码段时，使用的是可执行文件中的那一份“单例”，而在运行存在于动态链接库中的代码段时，使用的则是动态链接库中的那一份“单例”，这其实本质就是所谓的动态链接库间的内存隔离导致的问题。

note:  标记的这段话说的不准确， cpp 中的 static 是单个编译单元的静态变量，内存将被分配于当前的编译单元中

在内联函数中，(class 中定义的函数默认内联  inline)
- 所有函数定义中的函数局部静态对象在所有翻译单元间共享（它们都指代相同的在某一个翻译单元中定义的对象）。
- 所有函数定义中所定义的类型同样在所有翻译单元中相同。

由于关键词 inline 对于函数的含义从 C++98 起已经变为“容许多次定义”而不是“优先内联”，因此这个含义也扩展到了变量。 


解决这个问题有两条路：

1.设计代码时选好单例的内存该放在哪，然后通过导入导出解决（MSVC上导入导出使用__declspec(dllimport)和__declspec(dllexport)，类比extern变量）。比如单例打算放在动态链接库A中，那就在生成动态库A的地方的这个static变量导出，其他动态链接库或可执行文件使用的地方导入。

2.搞个统一的地方大家的单例共同注册到一起，用的时候拿出来。

我之前写过一个全局的单例模式支持这种跨编译单元的单例，可以参考：
[GitHub - KondeU/GlobalSingleton: 一个进程空间内全局单例的框架（注册到独立外置的一个dll中），支持热加载/热卸载动态链接库。](https://github.com/KondeU/GlobalSingleton)

我采用的解决方式是第2种，大家共同把单例注册到一个独立的动态库singleton_manager中，使用时只需要继承GlobalSingleton<T>即可（需要注意我实现的这个单例的singleton_manager只记录了指针，如果有热卸载动态库的需求，那么不适用）。
```


这个回答是我觉得是完全正确且十分清晰的：

这个问题得分开来讨论，首先就是问题提到的“内存分配问题”

> **o、dll、exe 等可执行模块一般都有独立的堆，跨模块的分配与回收往往会造成严重错误。[见](https://link.zhihu.com/?target=https%3A//github.com/Qihoo360/safe-rules/blob/main/c-cpp-rules.md%23user-content-incompletenewdeletepair)**。

假设不涉及提到的这个问题（大概率我觉得没问题）。那我们来聊单例，我们来用一个经典的单例来描述这个问题：

```cpp
struct X{
    static X& f();
};

inline X& X::f(){
    static X x;
    return x;
}
```

这是一个很常规的单例，C++11 保证了静态局部对象初始化的线程安全，所以使用的很多。我们之所以将它写在类外，是为了表示这个“**inline**”。

我看到这个问题下面有一些人批评，很瞧不上这种做法。又或者认为单例不应该写在头文件中，其实我认为这反而是普遍的需求，跨 dll 才是特殊的。

> **在内联函数中，所有函数定义中的函数局部静态对象在所有翻译单元间共享（它们都指代相同的在某一个翻译单元中定义的对象）。[见](https://link.zhihu.com/?target=https%3A//zh.cppreference.com/w/cpp/language/inline%23%3A~%3Atext%3D%25E6%2589%2580%25E6%259C%2589%25E5%2587%25BD%25E6%2595%25B0%25E5%25AE%259A%25E4%25B9%2589%25E4%25B8%25AD%25E7%259A%2584%25E5%2587%25BD%25E6%2595%25B0%25E5%25B1%2580%25E9%2583%25A8%25E9%259D%2599%25E6%2580%2581%25E5%25AF%25B9%25E8%25B1%25A1%25E5%259C%25A8%25E6%2589%2580%25E6%259C%2589%25E7%25BF%25BB%25E8%25AF%2591%25E5%258D%2595%25E5%2585%2583%25E9%2597%25B4%25E5%2585%25B1%25E4%25BA%25AB%25EF%25BC%2588%25E5%25AE%2583%25E4%25BB%25AC%25E9%2583%25BD%25E6%258C%2587%25E4%25BB%25A3%25E7%259B%25B8%25E5%2590%258C%25E7%259A%2584%25E5%259C%25A8%25E6%259F%2590%25E4%25B8%2580%25E4%25B8%25AA%25E7%25BF%25BB%25E8%25AF%2591%25E5%258D%2595%25E5%2585%2583%25E4%25B8%25AD%25E5%25AE%259A%25E4%25B9%2589%25E7%259A%2584%25E5%25AF%25B9%25E8%25B1%25A1%25EF%25BC%2589%25E3%2580%2582)。**

单例写在头文件中，被项目的其他 cpp 文件引入，并不构成问题，C++保证指代的是同**一个单例对象**。

[Compiler Explorer - C++ (x86-64 gcc 14.1)](https://link.zhihu.com/?target=https%3A//godbolt.org/z/P9MxbGWeh)

如果当前程序要编译为dll，要将单例导出，我觉得也不成问题，会确保只导出一个单例对象。我认为最终出现问题的是别的程序引用了这个有单例定义的头文件，并使用这个 dll。**C++ 标准似乎无法管理这种情况**，dll 不在管理的范围内，程序中此时可能出现多个单例。


### 规范的理解  inline ,  static,  static inline 

1. **函数**：[`inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 函数可以在多个编译单元中定义，而不会导致重复定义错误。编译器会将这些定义视为同一个函数，并在链接时合并它们。这使得 [`inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 函数非常适合在头文件中定义。
    
2. **变量**：[`inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 变量（C++17 引入）也可以在多个编译单元中定义，而不会导致重复定义错误。编译器会确保这些定义在链接时合并为一个单一的定义。

在头文件中声明 [`static inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 函数有以下几个作用：

1. **内联优化**：[`inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 关键字提示编译器将函数的调用替换为函数体，从而减少函数调用的开销。这对于小型、频繁调用的函数尤其有用。
    
2. **内部链接**：`static` 关键字使函数具有内部链接（internal linkage），这意味着每个包含该**头文件的编译单元都会有该函数的一个独立实例**，而不会与其他编译单元中的同名函数冲突。
    
3. **避免重复定义**：由于 [`inline`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.h") 函数可以在多个编译单元中定义而不会导致重复定义错误，`static inline` 函数可以安全地在头文件中定义，并且**每个包含该头文件的编译单元都会有自己的函数实例。**


# 类的成员函数以及成员变量

- inline修饰的函数或变量（c++17开始可以修饰变量）在**全局保留一份**；
    
- static修饰的函数或者变量会在各自的编译单元**都保留一份**；
    
- static函数的局部static变量**也会有多份**，inline函数的static变量**只有一份**;
    
- static inline 修饰的函数或者变量与static单独修饰的效果一致；(**不要把静态成员函数当成 static inline 修饰的函数， 两者虽然看起来 都是用 static inline 修饰， 但实际上语义并不相同, 类中的所有函数在全局都只有一份**)
    
- inline 不能修饰局部变量；
    
# 有关类的成员函数以及成员变量

- 类的非const静态成员变量初始化，**C++ 17可以通过static inline 在类内直接初始化**，C++17之前必须在类外初始化（const static 修饰的变量也可以类内初始化，这个在之前也是可以的）；
    
- C++17之后，类的静态成员变量在类内通过static声明，在类外（但是在头文件中）初始化不加inline的话可能会导致重定义从而出现链接错误，而加了inline 就不会出错，类似有无inline修饰的全局函数；
    
- C++ 17之前必须在.cpp中初始化静态成员才不会出现重定义的错误，在.h中初始化还是会导致重定义错误，因为C++17之前的标准不支持inline修饰类的静态成员变量；
    
- 类中的**函数**其实可以认为是都隐式加了inline的，因为**类中的所有函数在全局都只有一份，而有无static修饰只是限制该函数对类数据成员的使用**（类的static函数只能使用static成员变量，类的普通成员函数可以使用所有的成员变量 ）；
    

# **C++ 17之前为什么类的非const静态成员必须类外初始化？**

- 因为**类的静态成员是类所有，不属于任何对象，如果放在类内初始化，那么每个对象构造的时候都会对该静态成员进行初始化**，这样会导致其他对象在使用该静态成员时出现意想不到的错误，所以禁止类内初始化非const静态变量；
- 而const修饰的静态成员变量是在类中初始化，对象构造的时候不会再进行初始化操作，对象构造的时候默认该变量已经初始化了，反正也不能修改它；
- C++17之前的类内静态变量必须在类外初始化，如果有头文件和实现文件，最好在类的实现文件中对静态变量做初始化，如果在头文件中做初始化，也会引起重定义的问题；
- **C++17 之后可以在类内通过static inline 直接进行声明与初始化，也可以在类内部进行声明**，在类外（但是在头文件中）通过inline 进行初始化。其实就是**新标准通过inline能够保证类内静态变量只初始化一次，全局共享一份数据**，而之前的标准是不允许inline修饰类的静态成员变量的；

### 参考
[C++单例模式跨DLL是不是就是会出问题?](https://www.zhihu.com/question/425920019)

[07 | C++中的static和inline - 独立树 - 博客园](https://www.cnblogs.com/mmxingye/p/16200592.html)

[static 成员 - cppreference.com](https://zh.cppreference.com/w/cpp/language/static)