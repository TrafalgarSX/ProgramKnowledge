### What is the effect of extern "c" in C++?

```cpp
extern "C"{
	void foo();
}
or
#ifdef __cplusplus
extern "C" {
#endif

// all of your legacy C code here

#ifdef __cplusplus
}
#endif
```
在c/c++程序里， non-static functions 在一个程序的二进制文件里是用一个独一无二特殊字符串（symbols）表示的。
在C里， 这个特殊字符串和函数名一样（只是前面多了个下划线）；在C++里，由于函数可以重载，C++使用 name mangling 把函数名转换成了 weird-looking string, c的编译器不能找到的symbols

So if you specify a function to be extern C, the compiler doesn't performs name mangling with it and it can be directly accessed using its symbol name as the function name.

`extern "C"` 的几点特殊总结（不完全）：
* `extern "C"` is a linkage-specification
* A linkage specification shall occur only in **namespace scope**
* `extern "C"` is **ignored for class members**
* static inside extern "C" is valid; an **entity** so declared has **internal linkage**, and so does not have a **language linkage**

#### 释义
TODO  这些都是什么意思
编译单元  name/identifier(标识符)     绑定 实体(entity)
* **translation unit**: This is the **source file** from your implementation plus all the headers you `#included` in it.
* **entity**: 一个定义提供一个实体（类型、实例、函数）在一个作用域的唯一描述
* **internal linkage**:  everything only **in scope of a translation unit.**
* **External linkage**: refers to things that exist beyond a particular translation unit. In other words, **accessible through the whole program, which is the combination of all translation units (or object files).**
* **language linkage**: 提供不同编程语言编写的模块之间的联系

You can explicitly control the linkage of a symbol by using the **extern** and **static** keywords. If the linkage is not specified then the default linkage is extern (external linkage) for non-const symbols and static (internal linkage) for const symbols.

```c++
// In namespace scope or global scope.
int i; // extern by default
const int ci; // static by default  internal
extern const int eci; // explicitly extern
static int si; // explicitly static

// The same goes for functions (but there are no const functions).
int f(); // extern by default
static int sf(); // explicitly static 
```

#### 内外部链接的例子
以下情况是内部连接：**具有内部链接性的名字，在编译后不会产生链接符号，因此不与链接器打交道**
* 所有的声明
* 命名空间（包括全局空间）中的静态自由函数、静态友元函数、静态变量的定义，const常量定义
* enum定义
* inline函数定义（包括自由函数和非自由函数）
* 类的定义
* union的定义

以下情况是外部连接： **可以在不同编译单元声明并绑定到同一实体**
* 非inline函数。包括命名空间中非静态函数、类成员函数和类静态成员函数
* 类静态成员变量总有外部链接。
* 命名空间(不包括无名命名空间)中非静态变量

参考网站：
[C++声明与定义、内部链接与外部链接的意义\_Hansry的博客-CSDN博客](https://blog.csdn.net/Hansry/article/details/80207143)
[c++声明与定义/内部链接与外部链接 - Mr\_chuan - 博客园](https://www.cnblogs.com/shaochuanhe/articles/14238153.html)

### what is the difference between `#include <filename>` and `#include "filename"`?
不同之处就是预处理器寻找文件的路径不一样。
* `#include<filename>` 通常用于标准库和与目标平台相关的系统头文件。预处理器通常在编译器/IDE预先指定的目录中寻找（编译器设置的include路径内搜索）, 如果没有找到再到当前目录下找。
   e.g. 标准库  系统库  第三方库
* `#include "filename"` 预处理器会在相同目录下寻找文件, 如果找不到用`#include<filename>`方式去找。
注意其实#include后接<>或""包含的文件都是以**实现定义**（或者说implementation-defined）的方式去搜索的。 上面的情况是比较主流的实现。

### Undefined, unspecified and implementation-defined behavior
* undefined beheavior:是指执行某种计算机代码所产生的结果，**这种代码在当前程序状态下的行为在其所使用的语言标准中没有规定。** 
```txt
1.  第一，所谓未定义行为，是指对于程序员写的某些在常理上或逻辑上存在错误，并且很容易在执行时出错（如崩溃、结果不符合预期、或是引发其它更加严重、出乎意料的结果）的代码，标准并未做任何规定说该怎样处理，因为这些代码本来就是没有任何意义的。
2.  第二，既然标准未做任何规定，也就是说没有对我们编译器做任何约束，那我们编译器就可以自（wei）由（suo）发（yu）挥（wei）了。对于未定义行为，我们可以给出自己的定义，如报错、警告、或是其它特定的处理方式；也可以置之不理，完全无视；当然，理论上也可以做其它任何事情，如关机、格盘、删库等等。
3.  第三，对于未定义行为引发的任何后果，编译器概不负责。因为未定义行为本身就是不受法律（即标准）保护的。编译器的职责，只是为符合标准的“已定义行为”生成高效的代码（汇编及可执行文件）
4.  第四，编译器之所以对某些未定义行为不做检测，主要有以下原因。首先，检测需要做额外的工作，某些情况下可能会明显降低编译乃至执行效率，因此，出于效率考虑，没有做检测。其次，确实无法检测，如数组元素索引来自运行时的用户输入或是传感器的实时输入等。最后，在某些情况下， 编译器不检测和处理未定义行为（即默认所有行为都是合理合法、有明确定义的），可以极大地优化程序，生成高效的可执行代码。
5.  第五，未定义行为可以带来好处，主要是未定义行为允许编译器可以不检测和处理某些“错误”，减轻了编译负担，提升了编译效率，进一步地，为编译器优化代码提供了更大的自由和更多可能性。
6.  第六，“未定义”并不完全等于“非确定性”，更不等于“随机”。事实上，许多编译器对许多特定的未定义行为都有自己固定的处理方式（注意，不作任何处理也是一种处理），只不过，由于标准未做任何规定和约束，因此，对于同一未定义行为，不同的编译器可能作出不同的处理，或者同一编译器在不同的系统（环境）下可能作出不同的处理。
```
* unspecified beheavior: **use of an unspecified value, or other behavior where this International Standard provides two or more possibilities and imposes no further requirements on which is chosen in any instance** 
* implementation-defined beheavior: unspecified behavior where **each implementation documents how the choice is made** 
