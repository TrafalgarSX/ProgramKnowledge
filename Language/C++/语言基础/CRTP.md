
[C++ CRTP的原理与使用](https://zhuanlan.zhihu.com/p/714887437)

CRTP 可用于在父类暴露接口，而子类实现该接口，以此实现“**_编译期多态_**”，或称“**_静态多态_**”。

```cpp
template <class Dervied>
class Base {
public:
    // 公开的接口函数 供外部调用
    void addWater(){
        // 调用子类的实现函数。要求子类必须实现名为 impl() 的函数
        // 这个函数会被调用来执行具体的操作
        static_cast<Dervied*>(this)->impl();
    }
};

class X : public Base<X> {
public:
    // 子类实现了父类接口
    void impl() const{
        std::cout<< "X 设备加了 50 毫升水\n";
    }
};
```

### CRTP 的好处

上一节我们详细的介绍和解释了 CRTP 的编写范式和原理。现在我们来稍微介绍一下 CRTP 的众多好处。

1. **静态多态**

CRTP 实现静态多态，无需使用[虚函数](https://zhida.zhihu.com/search?q=%E8%99%9A%E5%87%BD%E6%95%B0&zhida_source=entity&is_preview=1)，**静态绑定，无运行时开销。**

1. **类型安全**

CRTP 提供了类型安全的多态性。通过模板参数传递具体的子类类型，编译器能够确保类型匹配，**避免了传统向下转换可能引发的类型错误。**

1. **灵活的接口设计**

CRTP 允许父类定义公共接口，并要求子类实现具体的操作。这使得基类能够提供通用的接口，而具体的实现细节留给[派生类](https://zhida.zhihu.com/search?q=%E6%B4%BE%E7%94%9F%E7%B1%BB&zhida_source=entity&is_preview=1)。其实也就是说多态了。