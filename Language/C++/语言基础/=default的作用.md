参考：
[c++ - How is "=default" different from "{}" for default constructor and destructor? - Stack Overflow](https://stackoverflow.com/questions/13576055/how-is-default-different-from-for-default-constructor-and-destructor)

对于C++ 11标准中支持的default函数，编译器会为其自动生成默认的函数定义体，从而获得更高的代码执行效率，也可免除程序员手动定义该函数的工作量。

C++的类有四类特殊成员函数，它们分别是：

- 默认构造函数
- 析构函数
- 拷贝构造函数
- 拷贝赋值运算符


Using `= default` syntax for special member functions (default constructor, copy/move constructors/assignment, destructors etc) means something very different from simply doing `{}`. With the latter, **the function becomes "user-provided". And that changes everything.**

The `= default` syntax is mainly there for doing things like copy constructors/assignment, when you add member functions that prevent the creation of such functions. **But it also triggers special behavior from the compiler, so it's useful in default constructors/destructors too.**

```c
class X {
private:
	int x;
};

class Y: public X {
private:
	int y;
};

int main(){
	X* x = new Y;
	delete x;   // 有内存泄露
}
```

```c
class X {
public:
	virtual ~X(){}; // 手动定义虚析构函数
private:
	int x;
};

class Y: public X {
private:
	int y;
};

int main(){
	X* x = new Y;
	delete x;
}
```

```c
class X {
public:
	virtual ~X()= default; // 编译器自动生成 default 函数定义体
private:
	int x;
};

class Y: public X {
private:
	int y;
};

int main(){
	X* x = new Y;
	delete x;
}
```
### =default 的错误用法
**=default 函数特性仅适用于类的特殊成员函数，且该特殊成员函数没有默认参数**。
```cpp
class X {
public:
	int f() = default; // 错误 , 函数 f() 非类 X 的特殊成员函数
	X(int) = default; // 错误 , 构造函数 X(int, int) 非 X 的特殊成员函数
	X(int = 1) = default; // 错误 , 默认构造函数 X(int=1) 含有默认参数
};

```


### =default 与 {}的区别
```cpp
// non-trivial
class B {
    public:
    B(){}
    int i;
    int j;
};
```

```cpp
// trivial
class B {
    public:
    B() = default;
    int i;
    int j;
};
```
#### 重要区别一：
这两个的重要区别就是：
default constructor defined with `B() = default;` is considered **not-user defined**. This means that in case of _value-initialization_ as in A special kind of initialization that doesn't use a constructor at all will take place and for built-in types this will result in _zero-initialization_. In case of `B(){}` this won't take place.

```cpp
#include <iostream>

class A {
    public:
    A(){}
    int i;
    int j;
};

class B {
    public:
    B() = default;
    int i;
    int j;
};

struct C {
    int i;
    int j;
};

int main()
{
    for( int i = 0; i < 10; ++i) {
        A* pa = new A();
        B* pb = new B();
        C* pc = new C();
        std::cout << pa->i << "," << pa->j << std::endl;
        std::cout << pb->i << "," << pb->j << std::endl;
        std::cout << pc->i << "," << pc->j << std::endl;
        delete pa;
        delete pb;
        delete pc;
    }
  return 0;
}
```

possible result:
```cpp
-842150451,-842150451
0,0
0,0
-842150451,-842150451
0,0
0,0
-842150451,-842150451
0,0
0,0
-842150451,-842150451
0,0
0,0
-842150451,-842150451
0,0
0,0
-842150451,-842150451
0,0
0,0
//...
```

#### 重要区别二：
{} 是用户提供的, =default 是和编译器自动提供的默认构造函数等是一样的。
前者不可以作为聚合体， 不能进行聚合体初始化， 后者可以。  （主要用于POD类型？)
```cpp
    A a = {.i = 1, .j = 2};  // error 编译错误 no matching constructor

    B b = {.i = 1, .j = 2};  // ok

    C c = {.i = 1, .j = 2};  // ok
```

![[聚合初始化#聚合体]]