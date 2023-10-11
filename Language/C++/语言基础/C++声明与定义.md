### 变量
---
**声明**：是指出存储类型，并给存储单元指定名称。

**定义**：是分配内存空间，还可为变量指定初始值。

**extern关键字**：通过使用extern关键字声明变量名，而不是定义它。

注：
1. 声明不一定是定义：**extern声明没有分配内存空间，所以不是定义**；extern告诉编译器变量在其他地方定义了。

例如：extern int val; // 只是声明了变量val，并没有分配内存空间，所以不是定义。

2. **定义也是声明**：定义变量的时候，同时给变量指出了存储类型，并给变量的存储单元指定了名称，所以也是声明。

例如：int val; // 声明了变量val，也会分配内存空间，所以也是定义。

3. extern关键字标识的才是声明，其余都是定义

例如：extern int val; // 声明

int val; // 定义

4. 声明有指定初始值：如果指定了初始值，即使前面加了extern关键字，也是定义。

例如：extern int val = 1; // 定义

5. 一个变量的定义永远只能有一个，但是可以有多个声明。

### 函数
---

1. **函数原型（又称函数声明）**：函数原型之于函数，相当于变量声明之于变量，只有函数头的就是函数原型。

例如：int function();

extern int function();

2. **函数定义**：带有函数体的就是定义，

例如：int function() { return 0; }

注：

1. 函数原型的**返回值类型**与函数定义必须相同。

2. 函数原型的形参表的类型与顺序必须与函数定义相同，**但是函数原型可以不写形参名称，即便写了形参名称也可以和原函数不一样。**

#### 空形参
C语言中：
int func(); // 表示可以有很多个参数

int func(void); // 表示没有参数

C++中，上述两行代码等价，且都表示没有参数。

### 声明规范
---
说明符与限定符：任意顺序的下列内容的空白符分隔列表
- 类型说明符
  - void
  - 算术类型名
  - 原子类型名
  - 先前由 typedef 声明引入的名称
  - struct、union 或 enum 说明符
 - 零或一**存储类型说明符** ： typedef auto register static extern thread_local
 - 零或多个**类型限定符**：const volatile restrict atomic
 - (只在声明函数时)零或多个函数说明符： inline _Noreturn_
 - 零或多个对齐说明符: `_Alignas`
 
```c
int a, *b=NULL; // “ int ”是类型说明符，
                // “ a ”是声明符
                // “ *b ”是声明符而 NULL 是初始化器
const int *f(void); // “ int ”是类型说明符
                    // “ const ”是类型限定符
                    // “ *f(void) ”是声明符
enum COLOR {RED, GREEN, BLUE} c; // “ enum COLOR {RED, GREEN, BLUE} ”是类型说明符
                                 // “ c ”是声明符
```

### 重声明

若另一个同一标识符的声明在同一[作用域](https://zh.cppreference.com/w/c/language/scope "c/language/scope")的较早部分出现，则声明不可再引入同一标识符，除非

- [有链接](https://zh.cppreference.com/w/c/language/storage_duration "c/language/storage duration")对象（外部或内部）的声明可以重复：
```c
extern int x;
int x = 10; // OK
extern int x; // OK
 
static int n;
static int n = 10; // OK
static int n; // OK
```

- 非 VLA [typedef](https://zh.cppreference.com/w/c/language/typedef "c/language/typedef") 可以任意重复，只要它指名同一类型：

```c
typedef int int_t; 
typedef int int_t; // OK
```

- [struct](https://zh.cppreference.com/w/c/language/struct "c/language/struct") 和 [union](https://zh.cppreference.com/w/c/language/union "c/language/union") 声明可以重复：

```c
struct X;
struct X { int n; };
struct X;
```

这些规则会简化头文件的使用。

⚠️若声明符的任一部分是[非常量长度数组](https://zh.cppreference.com/w/c/language/array "c/language/array")（ VLA ）声明符，则整个声明符的类型被称作“可变修改类型”。根据可变修改类型定义的类型同样是可变修改的（ VM ）。

任何可变修改类型声明只允许出现在[块作用域](https://zh.cppreference.com/w/c/language/scope "c/language/scope")或函数原型作用域，而且不能是任何结构体或联合体的成员。尽管 VLA 只能拥有自动或分配[存储期](https://zh.cppreference.com/w/c/language/storage_duration "c/language/storage duration")，一个 VM 类型，例如指向 VLA 的指针也可以有静态存储。关于 VM 类型有其他使用限制，见 [goto](https://zh.cppreference.com/w/c/language/goto "c/language/goto")、 [switch](https://zh.cppreference.com/w/c/language/switch "c/language/switch")、 [longjmp](https://zh.cppreference.com/w/c/program/longjmp "c/program/longjmp") 。

### extern
---
如果想要声明一个全局变量而非定义它， 就是用关键字extern,并且不要显式地初始化变量：
```cpp
extern int i;      // 声明i而非定义i
extern int i = 1;  // 定义i, 这样做抵消了extern的作用 
```

💡对于全局变量和静态变量即使没有显式赋初值，也会被默认初始化，但是对于局部变量，如果没有显式赋初值，它们的值是未定义的。

### static
---
当我们在C/C++用static修饰变量或函数时，主要有三种用途：

- 局部静态变量
- 外部静态变量/函数
- 类内静态数据成员/成员函数

#### 静态局部变量
在局部变量前面加上static说明符就构成静态局部变量

- 静态局部变量在**函数内定义**，但不像自动变量那样当函数被调用时就存在，调用结束就消失，**静态变量的生存期为整个源程序**
- 静态变量的生存期虽然为整个源程序，但是**作用域与自动变量相同**，即只能在定义该变量的函数内使用该变量，退出函数后虽然变量还存在，但不能够使用它
- 对基本类型的静态局部变量**如果在声明时未赋初始值，则系统自动赋0值**；而对普通局部变量不赋初始值，那么它的值是不确定的

据静态局部变量的特点，它的生存期为整个源程序，在离开定义它的函数（作用域）但再次调用定义它的函数时，它又可继续使用，而且保存了前次被调用后留下的值。因此，当多次调用一个函数且要求在调用之间保留某些变量的值时，可考虑采用静态局部变量，虽然用全局变量也可以达到上述目的，但全局变量有时会造成意外的副作用，因此最好采用局部静态变量。

#### 静态全局变量（C++中推荐用匿名命名空间替代）

📒Tips：对于全局变量，不管是否被static修饰，它的存储区域都是在静态存储区，生存期为整个源程序。只不过加上static后限制这个全局变量的作用域只能在定义该变量的源文件内。

当一个源程序由多个源程序组成时，非静态的全局变量在各个源文件中都是有效的，而静态全局变量则限制了其作用域，即只在定义该变量的源文件内有效，在同一源程序的其他源文件中不能使用它。

这种在文件中进行静态声明的做法是从C语言继承而来的，**在C语言中声明为static的全局变量在其所在的文件外不可见。这种做法已经被C++标准取消了，现在的替代做法是使用匿名命名空间。**

> 匿名命名空间：指关键字namespace后紧跟花括号括起来的一系列声明语句，具有如下特点：
> 
> - 在匿名命名空间内定义的变量具有**静态生命周期**
> - 匿名空间在某个给定的文件内可以不连续，但是不能跨越多个文件
> - **每个文件定义自己的匿名命名空间**，不同文件匿名命名空间中定义的名字对应不同实体
> - 如果在一个头文件中定义了匿名命名空间，则该命名空间内定义的名字在每个包含该头文件的文件中对应不同实体   

💡上面最后一条很有意思。  所以可以在头文件中定义匿名命名空间，并在其中定义变量。
```cpp
namespace {
    int i;  // 匿名命名空间内定义的变量具有静态生命周期, 作用域仅限于当前文件
}
```

#### 类静态成员变量
在类声明中声明数据成员时，**`static`** 关键字将指定类的所有实例共享该成员的一个副本。 必须在文件范围内定义 `static` 数据成员。 声明为 **`const static`** 的整型数据成员可以有初始化表达式。

在类声明中声明成员函数时，static 关键字将指定类的所有实例共享该函数。 由于函数没有隐式 this 指针，因此 static 成员函数不能访问实例成员。 若要访问实例成员，请使用作为实例指针或引用的参数来声明函数。

**不能将 union 的成员声明为 static**。 但是，全局声明的匿名 union 必须是显式声明的 static。

#### 一些结论总结
static这个说明符在不同地方所起的作用域是不同的，比如
- 把局部变量改变为静态变量后是改变了它的存储方式即改变了它的生存期
- 把全局变量改变为静态变量后是改变了它的作用域，限制了它的使用范围
- static不能和extern混用

在C++中，匿名命名空间和静态全局变量都可以用于实现类似的功能，即限制变量的作用域只在当前文件中可见。然而，匿名命名空间相对于静态全局变量具有一些优势，因此在许多情况下更推荐使用匿名命名空间中的变量。

以下是匿名命名空间的一些优势：

1. **避免命名冲突**：使用匿名命名空间可以避免全局命名冲突。变量和函数在匿名命名空间中定义后，只在当前文件中可见，不会污染全局命名空间。

2. **更好的封装性**：使用匿名命名空间可以将变量和函数封装在一个独立的作用域中，不会被其他文件访问或修改。这有助于提高代码的模块化和可维护性。

3. **隐式的静态特性**：匿名命名空间中的变量和函数具有隐式的静态特性，即它们只在当前文件中可见，不会被其他文件引用。这种隐式性更符合封装和信息隐藏的原则。

### auto
---
C++98 `auto`用于声明变量为自动变量（拥有自动的生命周期），`C++11`已经删除了该用法，取而代之的是“**变量的自动类型推断方法**”。

```cpp
// c++ 98:
int a = 10;         // 拥有自动生命期
auto int b = 20;    // 拥有自动生命期(C++11编译不过)
static int c = 30;  // 延长了生命期
```

C++11新标准引入了auto类型说明符，让编译器通过初始值来自动推断变量类型（这意味着通过**auto定义的变量必须有初始值**）。

#### auto会去除变量的引用语义
当引用对象作为初始值时，真正参与初始化的是引用对象的值，此时编译器会以引用对象的类型作为auto推算的类型：

```cpp
int main(void) {
    int i = 10;
    int &ri = i;
    auto auto_i = ri;  // 去除引用语义, 自动推断为int
}
```

如果希望推断出来的auto类型包含引用语义，我们需要用&明确指出：

```cpp
int main(void) {
    int i = 10;
    auto &auto_i = i;  // 加上引用语义, 自动推断为int&
}
```

#### auto忽略顶层const
auto一般会忽略掉顶层const，同时底层const会被保留下来：
```cpp
int main(void) {
    const int ci = 10;    // 常量int
    auto auto_ci = ci;    // auto_ci被推断为int类型
    auto_ci = 20;         // 正确: auto_ci非常量

    const int &cr = ci;   // cr是指向常量int的常量引用
    auto auto_cr = cr;    // auto_cr被推断为int类型: 去除了引用语义 + 去除了顶层const
    auto_cr = 20;         // 正确: auto_cr非常量

    const int *cp = &ci;  // cp是指向常量int(底层)的常量指针(顶层)
    auto auto_cp = cp;    // auto_cp被推断为const int*类型(指向常量int的指针): 去除了顶层const + 保留底层const
    // *auto_cp = 10;     // 错误: 不能修改auto_cp指向的常量
}
```

如果希望推断出来的auto类型是一个顶层const，我们需要通过const关键字明确指出：
```cpp
int main(void) {
    const int ci = 10;          // 常量int
    const auto auto_ci = ci;    // auto_ci被推断为const int类型
    // auto_ci = 20;            // 错误: auto_ci是一个常量, 禁止修改
}
```

### const
---
有时我们希望定义一个不能被改变值的变量，可以使用关键字const对变量类型加以限定。

#### const声明如何理解
rule:
1. 永远从右往左读一个变量声明。

2. const永远修饰const出现地点左边的声明类型。

3. 如果左边没有声明类型，就修饰右边。

所以：
```cpp
const int * const * const p1; //两者是一致的

int const * const * const p2;
```

解释为：p2是一个const的指针（指针本身不能变），指向const的指针（指向的指针也不能变），指向const int，最终指向的内存区域是一个int变量，也不能变。

所以用标准写法，也就是第二种写法在理解上会方便一些。但是第一种写法过于普遍了，几乎所有代码都这么写。下次读的时候从右往左然后自动脑补最左边的const是修饰右边的类型就好。

#### const对象必须初始化
因为const对象一经创建后其值就不能再改变，所以const对象必须初始化，但是初始值可以是任意复杂的表达式：
```cpp
const int i = get_size();  // 正确: 运行时初始化
const int j = 42;          // 正确: 编译时初始化
const int k;               // 错误: k是一个未经初始化的常量
```

#### 默认情况下const仅在文件内有效
举个例子，我们在编译时初始化一个const对象：
```cpp
const int i = 10;
```

编译器会在**编译过程把用到该变量的地方都替换为对应的值**。为了执行这个替换，编译器必须知道变量的初始值，如果程序包含多个文件，那么每个用了这个const对象的文件都必须得能访问到它的初始值才行（即每个文件都要定义const对象）。为了避免对同一变量的重复定义，当多个文件中出现同名的const对象时，其实等同于在不同文件中分别定义了独立的变量。

```cpp
/*
 * 下面是合法的, 不存在变量i重复定义问题
 */

// foo.cpp
const int i = 10;

// bar.cpp
const int i = 5;
```

如果想在多个文件之间共享const对象，那么必须在变量的定义之前添加extern关键字：

```cpp
/*
 * 下面是合法的, main.cpp和foo.cpp中的const int对象是同一个
 */

// foo.cpp
extern const int i = 10;

// main.cpp
#include <iostream>

int main(void) {
    extern int i;
    std::cout << "i:" << i << std::endl;
}
```

##### 上面两条的总结
全局变量如果遇到const,就会退化为静态变量的作用域，只能在本文件中作用。

如果想要在文件外使用const 全局变量，就需要在const外面加上个extern。
[[存储类说明符的链接性]]
#### 允许常量引用绑定非常量对象、字面值甚至一般表达式 (重点理解)
一般而言，引用的类型必须与其所引用对象的类型一致，但是有两个例外：

- **初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能转换成引用类型即可**，允许为一个常量引用绑定非常量的对象、字面值甚至是一个一般表达式（如下）
- 可以将基类的指针或引用绑定到派生类对象上（后续面向对象章节再探讨）

```cpp
int i = 10;

const int &ri1 = i;      // 合法: 绑定到非常量对象 先记忆吧， 为什么呢？
const int &ri2 = 100;    // 合法: 绑定到字面值  OK 这个可以理解
const int &ri3 = i * 2;  // 合法: 绑定到一般表达式  OK 这个也可以理解
```

讲解例子：
```cpp
double dval = 3.14;
// int &a = dval;    // 编译错误，因为普通引用的类型要与对象类型一致
const int &b = dval; // 编译正确  WHY? 因为中间多了一步转换，有临时量， 常量引用可以绑定这个临时量
```

##### 如何理解：
c++的自动类型转换机制中，当用一个double去初始化int时，会舍弃掉小数转换为int。在上面的例子中，编译后的代码实际是这样的：

```cpp
double dval = 3.14;
const int temp = dval; // 由double生成了一个临时的整形常量
const int &b = temp;   // 让b绑定这个临时量
```

在这种情况下，引用绑定的是一个临时量对象而不是dval本身。临时量对象就是：当编译器需要一个空间来暂存表达式的求值结果时，临时创建的一个未命名的对象。  

显然，c++认为，**常量引用可以绑定这个临时量，而普通引用就不能绑定这个临时量。**

因为c++认为，使用普通引用绑定一个对象，就是为了能通过引用对这个对象做改变。如果普通引用绑定的是一个临时量而不是对象本身，**那么改变的是临时量而不是希望改变的那个对象，这种改变是无意义的**。所以规定普通引用不能绑定到临时量上。

那么为什么常量引用就可以呢，**因为常量是不能改变的。也就是说，不能通过常量引用去改变对象，那么绑定的是临时量还是对象都无所谓了**，反正都不能做改变也就不存在改变无意义的情况。

所以常量引用可以绑定临时量，也就可以绑定非常量的对象、字面值，甚至是一般表达式，并且不用必须类型一致。

#### 顶层const与底层const (重点理解) 

⚠️底层翻译的不好， 英文是  low-level const
实际上你可以这样定义，这样就可以看到， 底层的翻译并不正确。
```cpp
int const * const * const p2;
```

指针本身是一个对象，因此**指针本身是不是常量**与**指针所指对象是不是常量**是两个独立的问题，前者被称为顶层const，后者被称为底层const。

> Tips：指针类型既可以是顶层const也可以是底层const，其他类型要么是顶层常量要么是底层常量。

💡指针常量 和 常量指针的主要区别在于 const 在修饰谁。 是在修饰指针(指针常量)，还是在修饰指针指向的变量（常量指）。理解声明的时候从右往左， 读的时候从左往右。

- 顶层const用于表示任意的对象是常量，包括算数类型、类和指针等， 比如顶层表示指针本身是个常量，指针不能改变指向(指针的值不能改变)。

- 底层const用于表示引用和指针等**复合类型的基本类型部分是否是常量**, 比如底层表示指针所指向的对象是个常量。

👉比较特殊的是，指针类型**既可以是顶层** const **也可以是底层** const 或者**二者兼备**。

顶层const用于表示任意的对象是常量，包括算数类型、类和指针等，底层const用于表示引用和指针等复合类型的基本类型部分是否是常量。
```cpp
int i = 10;

int *const p1 = &i;        // 顶层const: 不能改变p1的值
const int *p2 = &i;        // 底层const: 不能通过p2改变i的值
const int *const p3 = &i;  // 底层const + 顶层const

const int &r1 = i;         // 底层const: 不能通过r1改变i的值
```

##### 顶层const和底层const的区别
```cpp
int i = 0;
int *const p1 = &i;       // 不能改变p1的值，这是一个顶层const
const int ci = 42;        // 不能改变ci的值，这是一个顶层const
const int *p2 = &ci;      // 允许改变p2的值，这是一个底层const
const int *const p3 = p2; // 靠右的const是顶层const，靠左的const是底层const
const int &r = ci;        // 用于声明引用的const都是底层const
```

当执行拷贝操作时，常量是顶层 `const` 还是底层 `const` 的区别就非常明显了：

- 顶层 `const` 没有影响。拷贝操作不会改变被拷贝对象的值，因此拷入和拷出的对象是否是常量无关紧要。

```c
i = ci;     			// ok: 拷贝ci的值，ci是一个顶层const，对此操作无影响
p2 = p3;    			// ok: p2和p3指向的对象类型相同，p3顶层const的部分不影响
```

- 底层 `const` 的限制不能忽视。拷入和拷出的对象必须具有相同的底层 `const` 资格，或者两个对象的数据类型可以相互转换（一般来说，**非常量可以转换成常量**，反之则不行）。

```c
int *p = p3;    		// error: p3包含底层const的定义，而p没有
p2 = p3;        		// ok: p2和p3都是底层const
p2 = &i;        		// ok: int*能转换成const int*
int &r = ci;    		// error: 普通的int&不能绑定到int常量上
const int &r2 = i;  	// ok: const int&可以绑定到一个普通int上
```

### constexpr
---
C++11引入了常量表达式constexpr的概念，指的是值不会改变并且在**编译期间**就能得到计算结果的表达式。

```cpp
const int i = 10;          // 常量表达式
const int j = i + 1;       // 常量表达式
const int k = size();      // 仅当size()是一个constexpr函数时才是常量表达式, 运行时才能获得具体值就不是
```

在一个复杂系统中，我们很难分辨一个初始值是否是常量表达式，通过constexpr关键字声明一个变量，我们可以让编译器来验证变量的值是否是一个常量表达式。

#### 字面值是常量表达式

算术类型、引用和指针都属于字面值类型，自定义类则不属于字面值类型，因此也无法被定义为constexpr。

> Tips：尽管指针和引用都能被定义成constexpr，但它们的初始值却受到严格限制。一个constexpr指针的初始值必须是nullptr、0或者是存储于某个固定地址中的对象。

#### constexpr是对指针的限制

在constexpr声明中定义了一个指针，**限定符constexpr仅对指针有效**，与指针所指对象无关：

```cpp
const int *pi1 = nullptr;      // 底层const: pi1是指向整型常量的普通指针
constexpr int *pi2 = nullptr;  // 顶层const: pi2是指向整型的常量指针
```

我们也可以让constexpr指针指向常量：

```cpp
constexpr int i = 10;
constexpr const int *pi = &i;  // 顶层const + 底层const
```

### thread_local(C++11)
---
使用 **`thread_local`** 说明符声明的变量仅可在它在其上创建的线程上访问。 变量在创建线程时创建，并在销毁线程时销毁。 每个线程都有其自己的变量副本。 在 Windows 上，**`thread_local`** 在功能上等效于特定于 Microsoft 的 [`__declspec( thread )`](https://learn.microsoft.com/zh-cn/cpp/cpp/thread?view=msvc-170) 属性。

```cpp
thread_local float f = 42.0; // Global namespace. Not implicitly static.

struct S // cannot be applied to type definition
{
    thread_local int i; // Illegal. The member must be static. NOT OK
    thread_local static char buf[10]; // OK
};

void DoSomething()
{
    // Apply thread_local to a local variable.
    // Implicitly "thread_local static S my_struct".
    thread_local S my_struct;
}
```

- DLL 中动态初始化的线程局部变量可能无法在所有调用线程上正确初始化。 有关详细信息，请参阅 [`thread`](https://learn.microsoft.com/zh-cn/cpp/cpp/thread?view=msvc-170)。  （不是很重要）

- **`thread_local`** 说明符可以与 **`static`** 或 **`extern`** 合并。
    
- 可以将 **`thread_local`** 仅应用于数据声明和定义；**`thread_local`** 不能用于函数声明或定义。
    
- 只能在具有静态存储持续时间的数据项上指定 **`thread_local`**，其中包括全局数据对象（**`static`** 和 **`extern`**）、局部静态对象和类的静态数据成员。 如果未提供其他存储类，则声明 **`thread_local`** 的任何局部变量都为隐式静态；换句话说，在块范围内，**`thread_local`** 等效于 **`thread_local static`**。（重要， 上面的代码示例就是这一条）

- 不建议将 **`thread_local`** 变量与 `std::launch::async` 一起使用。 有关详细信息，请参阅 [`<future>` 函数](https://learn.microsoft.com/zh-cn/cpp/standard-library/future-functions?view=msvc-170)。



### goto 对变量定义的影响
![[goto对变量定义的影响]]
