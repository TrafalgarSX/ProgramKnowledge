### RAII
**RAII**，全称**资源获取即初始化**（英语：**R**esource **A**cquisition **I**s **I**nitialization），它是在一些面向对象语言中的一种惯用法。RAII源于C++,在Java， C#， D，Rust中也有应用。

RAII要求， 资源的有效期与持有资源的对象的声明期严格绑定，即由对象的构造函数完成资源的分配（获取），同时由于析构函数完成资源的释放。在这中要求下，只要对象能够正确的析构，就不会出现资源泄露问题。

RAII的主要作用是在不失代码简洁性的同时，可以很好地保证代码的异常安全性。

下面的C++实例说明了如何用RAII访问文件和互斥量
```C++
#include <string>
#include <mutex>
#include <iostream>
#include <fstream>
#include <stdexcept>
 
void write_to_file(const std::string & message)
{
    // 创建关于文件的互斥锁
    static std::mutex mutex;
 
    // 在访问文件前进行加锁
    std::lock_guard<std::mutex> lock(mutex);
 
    // 尝试打开文件
    std::ofstream file("example.txt");
    if (!file.is_open())
        throw std::runtime_error("unable to open file");
 
    // 输出文件内容
    file << message << std::endl;
 
    // 当离开作用域时，文件句柄会被首先析构 (不管是否抛出了异常)
    // 互斥锁也会被析构 (同样地，不管是否抛出了异常)
}
```

C++保证了所有栈对象在生命周期结束时会被销毁(即调用析构函数），所以该代码是异常安全的。无论在write_to_file函数正常返回时，还是在途中抛出异常时，都会引发write_to_file函数的[堆栈回退](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88%E5%9B%9E%E9%80%80 "堆栈回退")，而此时会自动调用lock和file对象的析构函数。

当一个函数需要通过多个局部变量来管理资源时，RAII就显得非常好用。因为只有被构造成功(构造函数没有抛出异常)的对象才会在返回时调用析构函数，同时析构函数的调用顺序恰好是它们构造顺序的反序，这样既可以保证多个资源(对象)的正确释放，又能满足多个资源之间的依赖关系。

由于RAII可以极大地简化资源管理，并有效地保证程序的正确和代码的简洁，所以通常会强烈建议在C++中使用它。

### 同样原理的技巧但是不用智能指针，可同时释放多个资源，并且没有类型限制，scope guard

将 类析构函数和 lamda 函数表达式 结合起来
```cpp
#include<functional>
#include<iostream>
class ScopeGuard
{
    std::function<void()> mFunc;

public:
    ScopeGuard(std::function<void()> f)
    {
        mFunc = f;
    }
    ~ScopeGuard()
    {
        mFunc();
    }
};

int doSomething(int* p) {
   return -1;
}
void finalize(int* p) {
}
void f() {
   int* p = new int{3};
   ScopeGuard s([&p]() {
        if (p) {
            delete[] p;
        };
        std::cout << "delete point\n";
    });
   int error = doSomething(p);
   if (error) {
       return;
    }  
   finalize(p);
   std::cout<<"Function ends!\n";
}
int main()
{
    f();
}
```

原文出处:
[RAII:如何编写没有内存泄漏的代码 with C++ - 知乎](https://zhuanlan.zhihu.com/p/264855981)