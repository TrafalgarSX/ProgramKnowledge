### C/C++是否支持VLA
VLA： variable-length array 变长数组
```c
void vla_func() 
{
	int n;
	scanf("%d",&n);

	int array[n];
	/* 错误的，使用变长数组不能进行初始化
	 * 因为长度在运行时才能确定
	/* int array[n] = {0}; 
	 * 
	*/ 
	for(int i=0;i<n;i++)
	{
		printf("%d\t", array[i]);	
	}
}
```

在C99之后，上面的写法是允许的，经过测试gcc和clang都支持该标准。

C90标准中并不支持VLA，C99开始支持VLA，很大的一个原因：FORTRAN中支持这种写法。**C99中对对VLA有一些限制，比如变长数组必须是自动存储类型，也就是说，如果我上面两句放在函数外面就就不能通过编译了**，这是因为在函数外面定义的是全局变量，此外，使用VLA不能对数组进行初始化，因为它的长度在运行时才能确定。

此外VLA并不是真正的变长，它实际上只是将数组的长度推迟到运行时确定而已，也就是说C90标准中，数组的长度必须在编译时期就知道，但C99支持VLA后，数组的长度可以推迟到运行时知道，但是，一旦长度确定，数组的长度就不能变了。

不同C++编译器对VLA的支持是不同的， 目前好像只有gcc支持，不过不建议在开发中使用VLA，无论C或者C++。

### C/C++中const关键字的区别

**const关键字最早是C++中的产物，后来才引入C语言中。**

const在C语言中和C++中是相当不一样的。

- 在C语言中const修饰的，被认为是一个**只读的，或者叫不可改变的变量**。  是有限制的变量。
- 在C++中const修饰的才是**真正的常量（编译时值）。**
- c++中的const 变量默认内部链接， C中默认具有外部链接
  >如果需要在其他文件中访问 const 全局变量，可以使用 extern 关键字进行声明，并确保在另一个文件中定义该变量。  当 extern const 一起的时候， extern 的链接性优先。
  > const 局部变量没有链接性，其他函数无法访问该局部变量。它在块的每次执行时都被创建和初始化，并在块执行结束时销毁。

一些程序帮助理解区分：
```c++
const int a = 10;

int array[a];

int main()
{
         return 0;
}

```

该程序在g++中可以编译通过，在gcc下编译不通过。
原因：
- a是全局变量，全局变量位于数据区，空间在编译时期就被分配了。但是编译期间，编译器不能读取数据区的内存（可分配并初始化，但并不能读取）。**所以编译时期，编译器并不知道a的值是什么，因为它不能读取数据区的内存而a的值是在内存中的**
对于数组array 编译器一定要知道数组长度才可以，也就是必须要知道a的值

- C++中可以通过是因为C++真的**把const当成常量看待**
>字面量，无论是在C中或者C++中字面量都是保存在代码段中的，编译期间会将其保存在符号表中。

**C++尽量不对const分配数据区（或者运行时的栈区）的内存空间，只有必须分配时才分配。(所以C++的const可以退化成 C的const)**

`const int a = 10`下面一条语句`int array[a]`，编译器一定要知道a的值的，C语言要想知道a的值，必须读内存，但是C++却不需要，直接读取代码段中的的符号表即可，编译时期访问符号表是没有任何问题的，但是访问数据区的内存是做不到的。所以上面的语句在C++中是没有问题的。

```c++
const int a = 10;

const int *p = &a;

int array[a];

int main()
{
         return 0;
}
```
注意 `const int *p = &a;`这句，对a取地址操作，**我们知道位于代码段的数据是不取地址的**，所以这个时候，只能给a在数据区分配空间了。（但是还是没有出现了退化， 因为当编译器读a的值的时候，不是从数据区的内存中，而是程序段的符号表中读取的那个字面常量10。）

结论：**C++中的const之所以和C语言中的const不一样，C++中的const之所以能够看成常量，就是因为C++在读取const的时候，实际上读取的是代码段的字面常量，而不是数据区（对于全局变量来说是静态区，对于局部变量来说是栈区）的内存中的数值。**


C++中const退化的情况：
```c++
#include <stdio.h>

int main()
{
         int i = 10;

         const int a = i;

         int *pa = (int *)&a;

         printf("pa指向的地址为：%p  a的地址为：%p\n",pa,&a);

         (*pa)++;

         printf("a = %d,*pa = %d\n",a,*pa);

         return 0;
}
```
C++读取const的值，实际上是读取代码段的字面常量，那么，如果我们初始化const的时候，给它的不是字面量（或者是常量表达式），那么他就没有字面量可以读啦！这时候就只能退而求其次，去读数据区内存中的值啦！这个时候，C++中的const和C语言中的const就一样了。(const出现了退化)

**此外在类中使用const修饰类成员变量的时候也要小心，因为也会退化成C语言中的const**

```c++
#include <stdio.h>
class A{
         public:
                  const int a;

                  int array[a];
                  A(int i):a(i)
                  {
                  }

};
int main()
{
         return 0;
}
```

以上编译器会报错。
使用初始化列表对a进行初始化。 由于对象创建之前在会调用构造函数进行对象的初始化，所以**在编译期间我们根本就不知道a的值，所以编译会出错。**

```c++
#include <stdio.h>

class A{
         public:
                  static const int a = 10;
                  int array[a];

                  A()

                  {

                  }
};
int main()
{
         return 0;
}
```

类中定义常量应该用`static const int constant = 10;` 这种写法。

#### 比较特殊的特性区别
[Const correctness in C vs C++ - Stack Overflow](https://stackoverflow.com/questions/8908071/const-correctness-in-c-vs-c)
[c - Why does passing char\*\* as const char\*\* generate a warning? - Stack Overflow](https://stackoverflow.com/questions/14562845/why-does-passing-char-as-const-char-generate-a-warning)
难以理解， 以后再说。  TODO

In addition to the differences you cite, and the library differences that Steve Jessop mentions,

```csharp
char* p1;
char const* const* p2 = &p1;
```

is legal in C++, but not in C. Historically, this is because C originally allowed:

```csharp
char* p1;
char const** p2 = &p1;
```

Shortly before the standard was adopted, someone realized that this punched a hole in const safety (since `*p2` can now be assigned a `char const*`, which results in `p1` being assigned a `char const*`); with no real time to analyse the problem in depth, the C committee banned any additional `const` other than top level const. (I.e. `&p1` can be assigned to a `char **` or a `char **const`, but not to a `char const**` nor a `char const* const*`.) The C++ committee did the further analysis, realized that the problem was only present when a `const` level was followed by a non-`const` level, and worked out the necessary wording. (See §4.4/4 in the standard.)


#### 另一种易错的区别
if a program could assign a pointer of type T** to a pointer of type const T** (that is, if line #1 below were allowed), **a program could inadvertently modify a const object** (as it is done on line #2). For example

```c
int main()
{

  const char c = 'c';
  char* pc;

  //c = 'a';  // not allowed
  const char** pcc = &pc;   //#1 not allowed in c++
  *pcc = &c;
  *pc = 'B';                //#2

  printf("c = %c\n", c);
  
  return 0;
}
```

测试发现这段代码C++无法编译通过， 但是C可以编译通过（有警告），并且成功修改了 变量c。
[Question 11.10](https://c-faq.com/ansi/constmismatch.html)

#### 链接性
　const关键字具有内部链接属性，但是其“优先级”是低于extern的，如果我们以

    extern const int i = 0;

　- 这样的语法来定义一个常量i时，这个i是具有**外部链接**属性的，换而言之在使用时需要注意先用extern声明。

　- 关于const修饰的变量地址：**在通常情况下编译器是不会为const对象分配内存**，只有在几种情况下会分配地址：

　　1.extern和const同时使用，使得变量具有了外部链接属性。

　　2.**对const对象取地址**—-我们知道常量一般是放到**符号表**中的，所以上面的情况即使各个编译单元有自己的额外定义，但是实际上还是存放在同一个地方，并不占额外空间。而当我们的程序对const对象取地址的时候才会强制分配内存地址：不同CPP里面定义的相同常量一般来说对他们取地址得到的结果应该是一样的。(至少MSVC中是这样的结果)

　　3.runtime的const—-编译器是需要为它分配空间的，而且也不在符号表里面记录相关信息。