
[关于std::weak_ptr使用的一些理解](https://zhuanlan.zhihu.com/p/579934418)


### 核心内容

说了这么多，**那么`std::weak_ptr`除了解决相互引用的问题，还能做什么？**

答案是：**一切应该不具有对象所有权，又想安全访问对象的情况**。 还是以互相引用的情况为例，通常的场景是：一个公司类可以拥有员工，那么这些员工就使用`std::shared_ptr`维护。另外有时候我们希望员工也能找到他的公司，所以也是用`std::shared_ptr`维护，这个时候问题就出来了。但是实际情况是，**员工并不拥有公司，所以应该用`std::weak_ptr`来维护对公司的指针**


最后再来聊一个新手使用`std::weak_ptr`容易被坑的地方：对象资源竞争。以下代码在[多线程](https://zhida.zhihu.com/search?q=%E5%A4%9A%E7%BA%BF%E7%A8%8B&zhida_source=entity&is_preview=1)程序中是存在很大风险的，因为`wp.expired()`和`wp.lock()`运行的期间对象可能被释放：

```cpp
// std::weak_ptr<SomeClass> wp{ sp };

if (!wp.expired()) {
    wp.lock()->DoSomething();
}
```

正确的做法是：

```cpp
auto sp = wp.lock();
if (sp) {
    sp->DoSomething();
}
```

`std::weak_ptr`的`lock`函数是一个[原子操作](https://zhida.zhihu.com/search?q=%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C&zhida_source=entity&is_preview=1)。有趣的是，最开始的C++11标准是没有提到原子操作的，C++14标准才对这一点进行了补充，详细过程可以参考提案文档：[LWG2316](https://link.zhihu.com/?target=https%3A//cplusplus.github.io/LWG/issue2316)。