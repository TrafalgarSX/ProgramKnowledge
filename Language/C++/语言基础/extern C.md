### extern "C"作用
---
extern “C”用于修饰函数的时候，只是告诉编译器不要对这个函数的函数名进行mangle，也就是让编译器直接使用你代码里的名字作为最终[obj文件](https://www.zhihu.com/search?q=obj%E6%96%87%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)当中的**链接符号**（同时也自然失去了了C++[函数重载](https://www.zhihu.com/search?q=%E5%87%BD%E6%95%B0%E9%87%8D%E8%BD%BD&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)的能力，因为C++函数重载其实就是靠在编译的时候对函数名施加魔法实现的），并且把该函数的链接属性设为external，也就是可以和其它obj文件进行链接。

extern "C" 它本身并不能保证被其修饰的对象能够被C语言写的代码正确调用，只是保证被其修饰的函数就是使用代码里的那个函数名，并且可以被别的obj调用（具有外部连接属性），仅此而已。
也就是说extern "C" 影响的是链接期，extern "C"决定了符号（变量和函数）以什么样的命名规则导出，**C的导出符号名字就是符号本身，并不包含类型信息**

所以如果你在参数或者[返回值](https://www.zhihu.com/search?q=%E8%BF%94%E5%9B%9E%E5%80%BC&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1988830501%7D)里面用了C++独有的东西，那么**即便可以正常编译连接，C语言（或者其它高级语言）那边是很可能没有办法正确调用它的。**

但是你如果只是在**函数体内使用C++特性**，那么因为本来就是作为C++代码编译的，自然不会有任何问题。(它是你的内部实现细节；**只要内部细节不暴露，对外随意。因为反正都会编译成二进制的代码**)

- `extern "C"` is ignored for class members

- `extern "C"` forces a function to have external linkage (cannot make it static)' is incorrect. 'static' inside 'extern "C"' is valid; an entity so declared has internal linkage, and so does not have a language linkage.

- Only function names and variable names with external linkage have a language linkage

- In multiple levels of nested extern declarations, the innermost extern specification prevails.

```cpp
// In this example, func has C++ linkage.
extern "C" {
      extern "C++" {
            void func();
      }
}
```


```cpp
#include <iostream>
#include <vector>

// 函数体内使用c++特性，没有问题， 因为最后都编译成汇编了。
extern "C" void function(int a, int b) {
  std::cout << " extern C function with cpp stl vector body" << std::endl;
  std::vector<int> nums = {0, 1, 2, 3, 4, 5};
  std::cout << "a = " << a << std::endl;
  for (auto num : nums) {
    std::cout << num << std::endl;
  }
}

// extern "C" 后就不能函数重载了， 如果下面打开就会出现编译错误。
#if 0
extern "C" void function(char a, int b) {

}
#endif
// 如下面的test1 函数所示， 正常c++是可以重载的
void test1(int a) { std::cout << "test1" << std::endl; }

void test1(char a) { std::cout << "test2" << std::endl; }

// extern "C" 后 函数声明也可以带 c++ 特性，但是没有意义，因为该函数实际上应该被c语言调用并
// 编译， 但是c语言中并没有 std::vector
extern "C" void function2(std::vector<int> nums) {
  std::cout << "extern C function parameter with stl vector" << std::endl;
  for (auto num : nums) {
    std::cout << num << std::endl;
  }
}
// 下面两个函数被调用会出现编译错误，找不到符号， 这时候可以看到编译器提示
// 这两个函数的 symbol 是不一样的， 后者被 name mangling 了， 前者函数的 symbol 就是函数名 name。
// extern "C" void name(std::vector<int> nums);
void name(std::vector<int> nums);

int main() {
  // if cpp compiler
#if __cplusplus
  printf("C++\n");
#else
  printf("C\n");
#endif
  std::vector<int> nums = {0, 1, 2, 3, 4, 5};
  int a;
  scanf("%d", &a);
  if (a == 1) {
    goto end;
  }
  function(1, 2);
  function2(nums);
  // int b = 10;
  int b;
  printf("b = %d\n", b);
  name(nums);

end:
  return 0;
}
```

下图是 gcc 对函数名是否进行mangling的结果
![[gcc_mangling.png]]

下图是visual studio 对函数名进行mangling的结果
![[cl_mangling.png]]

#### 如果函数有多个链接规范
```cpp
// 下面这种声明顺序，编译正常
extern "C" int CFunc1();
...
int CFunc1();            // Redeclaration is benign; C linkage is
                         //  retained.

// 颠倒后，编译不通过
int CFunc2();
...
extern "C" int CFunc2(); // Error: not the first declaration of
                         //  CFunc2;  cannot contain linkage
                         //  specifier.
```

If a function has **more than one linkage specification, they must agree（一致）**. It's an error to declare functions as having both C and C++ linkage. 

Furthermore, if two declarations for a function occur in a program, one with a linkage specification and one without, **the declaration with the linkage specification must be first.** Any redundant declarations of functions that already have linkage specification are given the linkage specified in the first declaration.


### Name mangling
---
**Name mangling** (also called name decoration) is a method used by C++ compilers to add additional information to the names of functions and objects in object files. This information is used by linkers when a function or object defined in one module is referenced from another module. Name mangling serves the following purposes:

1. Make it possible for linkers to **distinguish between different versions of overloaded functions.**
2. Make it possible for linkers to check that objects and functions are declared in exactly the same way in all modules.
3. Make it possible for linkers to give complete information about the type of unresolved references in error messages.

Name mangling was invented to fulfill purpose 1. The other purposes are secondary benefits not fully supported by all compilers.

The minimum information that **must be supplied** for a function is the name of the function and the types of all its parameters as well as any class or namespace qualifiers. 

**Possible additional information** includes the return type, calling convention, etc. All this information is coded into a single ASCII text string which looks cryptic to the human observer. The linker does not have to know what this code means in order to fulfill purpose 1 and 2. It only needs to check if strings are identical.
