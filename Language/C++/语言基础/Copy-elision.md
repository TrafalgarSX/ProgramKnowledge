省略复制及移动 (C++11 起)构造函数，导致零复制的按值传递语义。 

```
`-fno-elide-constructors`  gcc 下可以使用该参数关闭 copy-elison 
```
### copy-elision的各种情况与原理

Copy elision是指编译器为了优化，将不需要的copy/move 操作（含析构函数，为了行文简洁，本文忽略析构函数的省略）直接去掉了。
##### RVO
```cpp
// RVO
Verbose createWithPrvalue()
{
  return Verbose();
}
Verbose v = createWithPrvalue();
```

在没有优化发生的时候，编译器在返回createWithPrvalue()的时候，做了下面三件事

1. 创建临时的Verbose对象，
2. 将临时的Verbose对象复制到函数的返回值，
3. 将返回值复制到变量v。


所以在没有优化的情况下（c++11)  除了 调用函数的地方会有一个对象外， 调用函数会创建两个额外的对象， 函数内的 纯右值 的临时对象（ c++11标准，纯右值会有临时对象， c++17标准没有，所以C++17 RVO是语言上保证了的，不是编译器优化了） 与 函数返回值的纯右值对象， 所以没有优化的话： 上面的调用过程中会是这样:
```bash
// c++11
constructor
move constructor
destructor
move constructor
destructor
destructor

// c++17
```

如果开启了  copy-elison 的话 或者是 c++17标准的话，调用的结果是
```
// c++17
constructor
deconstructor
```

c++17 纯右值 的定义发生了变化后， **纯右值在初始化对象的时候，也就是 Version()初始化 函数返回值对象 的时候 与 函数返回值对象初始化 v 的时候** 都不再有 临时对象的实质化（C++17中， 纯右值的实质化后就变成了  xvalue , 不再是 prvalue), 这样就可以保证 RVO 是复制消除。


##### NRVO
```cpp
#include <iostream>
class Verbose {
    int i;
public:
    Verbose() { std::cout << "Constructed\n"; }
    Verbose(const Verbose&) { std::cout << "Copied\n"; }
    Verbose(Verbose&&) { 
        std::cout << "Moved\n"; }
};

// NRVO
Verbose create() {
    Verbose tv;
    return tv;
}

int main() {
    Verbose v = create();
    return 0;
}
```

没有开启 copy-elision输出结果
```cpp
// c++11
constructor
move constructor
destructor
move constructor
destructor
destructor

// c++17 
constructor
move constructor
destructor
destructor
```

为什么会有这个区别呢？
c++11 的执行流程:
1. 未调用函数`create()`前，选择一个空间v-space用来存储v。
2. 准备进入函数`create()`时，选择一个临时的空间temp-space来存储函数的返回值。
3. 进入函数后，选择一个地址创建tv。
4. 函数返回时，将tv 移动到temp-space。
5. 赋值的时候，从temp-space 将tv 移动到v-space。

因为 c++17 函数返回的对象由 函数内的 tv 初始化的时候是**不实质化的** 也就是第四步被消除了。

总结就是: 所以C++17以后就不用再提RVO了，因为已经是语言标准，而不是优化了。

#### 强制的复制/移动操作消除

以下情况要求编译器省略类对象的复制和移动构造，**即使复制/移动构造函数和析构函数拥有可观察的副作用**。这些对象将直接构造到它们本来要复制/移动到的存储中。**复制/移动构造函数不必存在或可访问**：

- 在 return 语句中，当操作数是一个与函数返回类型相同的类类型的纯右值（忽略 cv 限定）时： 
```cpp
T f()
{
    return T();
}
 
f(); // 只调用一次 T 的默认构造函数
// 不会调用复制构造函数
```

❗返回类型的析构函数必须**在 return 语句位置可访问且未被删除**，即使没有 T 对象要被销毁。 

- 在对象的初始化中，当初始化器表达式是一个与变量类型相同的类类型的纯右值（忽略 cv 限定）时： 
```cpp
T x = T(T(f())); // 只调用一次 T 的默认构造函数以初始化 x      c++17开始
```

只能在已知**要初始化的对象不是可能重叠的子对象**时应用此规则：
```cpp
struct C { /* ... */ };
C f();
 
struct D;
D g();
 
struct D : C
{
    D() : C(f()) {}    // 初始化基类子对象时无消除
    D(int) : D(g()) {} // 无消除，因为正在初始化的 D 对象可能是某个其他类的基类子对象
};
```

注意：上述规则指定的**不是优化**：针对[纯右值](https://zh.cppreference.com/w/cpp/language/value_category "cpp/language/value category")和[临时量](https://zh.cppreference.com/w/cpp/language/implicit_conversion#.E4.B8.B4.E6.97.B6.E9.87.8F.E5.AE.9E.E8.B4.A8.E5.8C.96 "cpp/language/implicit conversion")的 C++17 核心语言规定在**本质上不同于之前的 C++ 版本**：不再有临时量用于复制/移动。描述 C++17 语义的另一种方式是“传递未实质化的值”：**返回并使用纯右值时不实质化临时量。**

#### 参考
[Copy/move elision: C++ 17 vs C++ 11](https://zhuanlan.zhihu.com/p/379566824)
[复制消除 - cppreference.com](https://zh.cppreference.com/w/cpp/language/copy_elision)