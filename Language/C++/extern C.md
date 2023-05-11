### extern "C"作用
---
extern “C”用于修饰函数的时候，只是告诉编译器不要对这个函数的函数名进行mangle，也就是让编译器直接使用你代码里的名字作为最终[obj文件](https://www.zhihu.com/search?q=obj%E6%96%87%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)当中的**链接符号**（同时也自然失去了了C++[函数重载](https://www.zhihu.com/search?q=%E5%87%BD%E6%95%B0%E9%87%8D%E8%BD%BD&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)的能力，因为C++函数重载其实就是靠在编译的时候对函数名施加魔法实现的），并且把该函数的链接属性设为external，也就是可以和其它obj文件进行链接。

extern "C" 它本身并不能保证被其修饰的对象能够被C语言写的代码正确调用，只是保证被其修饰的函数就是使用代码里的那个函数名，并且可以被别的obj调用（具有外部连接属性），仅此而已。
也就是说extern "C" 影响的是链接期，extern "C"决定了符号（变量和函数）以什么样的命名规则导出，C的导出符号名字就是符号本身，并不包含类型信息

所以如果你在参数或者[返回值](https://www.zhihu.com/search?q=%E8%BF%94%E5%9B%9E%E5%80%BC&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)里面用了C++独有的东西，那么即便可以正常编译连接，C语言（或者其它高级语言）那边是很可能没有办法正确调用它的。

但是你如果只是在函数体内使用C++特性，那么因为本来就是作为C++代码编译的，自然不会有任何问题。(它是你的内部实现细节；只要内部细节不暴露，对外随意。因为反正都会编译成二进制的代码)