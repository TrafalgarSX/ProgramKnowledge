### goto语句对变量声明的影响
---
#### C
- C语言中：goto 语句导致无条件跳转到前附具名 标号 （必须与 goto 语句出现于同一函数中）的语句，**除非此跳转会进入非常量长度数组或另一可变修改类型的作用域**

  标号 是一个后随冒号（ : ）和一条语句的标识符。标号是仅有的拥有**函数作用域**的标识符：能在其所出现于的函数中的任何位置使用它们（在 goto 语句中）。任何语句前可以有多个标号。 

```c
goto lab1; // OK ：进入常规变量的作用域
    int n = 5;
lab1:; // 注意未初始化 n ，如同以 int n; 声明
 
//   goto lab2;   // 错误：进入二个 VM 类型的作用域
     double a[n]; // VLA
     int (*p)[n]; // VM 指针
lab2:
```

若 `goto` 离开 VLA 的作用域，则 VLA 会被解分配（而且可能会被再分配，若再度执行其初始化）：
```c
{
   int n = 1;
label:;
   int a[n]; // 重分配 10 次，每次拥有不同的大小
   if (n++ < 10) goto label; // 离开 VM 的作用域
}
```

👉 上面的代码中 label冒号后有分号 `;`
这是因为：

因为声明不是语句，在声明前的标号必须使用空语句（紧随冒号后的分号）。同样的规则适用于块结尾前的标号。

C++ 在 `goto` 语句上加上了另外的限制，不过允许标号在声明（它们在 C++ 中是语句）前（i.e. c++ 在声明前的label 不用加分号 `;`）。

##### 编译器测试
经过测试，使用C编译器的情况下，linux下的 gcc clang, windows下的gcc clang visual studio 都可以正常编译goto语句后有变量声明定义的， 没有报错，也没有warning。

#### C++
goto 语句将控制转移到标号所指定的位置。goto 语句必须与它所用的 标号 处于相同的函数中，它在标号的前后都可以出现。 

如果控制的转移退出了任何**自动变量**的作用域（例如通过回跳到这种变量声明之前的位置，或向前跳出作为变量作用域的复合语句），那么为所有**退出作用域的变量**以其构造顺序的逆序调用析构函数。 

💡这个和C语言的 goto 退出 可变修改类型的作用域 的情况有点类似， 都是会重新执行自动变量的构造函数， 不过C++的会先逆序调用析构函数。

`goto` 不能将控制转移到 [try 块](https://zh.cppreference.com/w/cpp/language/try_catch "cpp/language/try catch")或 catch 子句之内，但能将控制转移离开 try 块或 catch 子句（遵循上述有关作用域中的自动变量的规则）（i.e. 能跳出来，不能跳进去)

C++中goto语句相对于C的额外限制：

如果控制被转移进入了任何自动变量的作用域（例如通过**向前跳过声明语句**），那么程序非良构（不能编译），除非进入作用域的所有变量拥有

1) 标量类型，且声明**不带初始化器**  
标量类型: [[POD类]]
2) 拥有平凡默认构造函数和析构函数的类类型，且声明不带初始化器

3) 上述之一的 cv 限定版本

4) 上述之一的数组

（注意：相同规则适用于控制转移的所有形式）

📒 在 C 编程语言中，`goto` 语句的限制较少，并且能进入除变长数组或可变修改指针之外的任何变量的作用域。
📒 平凡的构造与析构函数就是默认的构造和析构函数
```cpp
#include <iostream>
 
struct Object {
    // 非平凡析构函数
    ~Object() { std::cout << "d"; }
};
 
struct Trivial {
    double d1;
    double d2;
}; // 平凡构造函数与析构函数
 
int main()
{
    int a = 10;
 
    // 使用 goto 循环
label:
    Object obj;
    std::cout << a << " ";
    a = a - 2;
 
    if (a != 0) {
        goto label;  // 跳出 obj 的作用域，调用 obj 析构函数
    }
    std::cout << '\n';
 
    // goto 可用于简单地离开多层循环
    for (int x = 0; x < 3; x++) {
        for (int y = 0; y < 3; y++) {
            std::cout << "(" << x << ";" << y << ") " << '\n';
            if (x + y >= 3) {
                goto endloop;
            }
        }
    }
endloop:
    std::cout << '\n';
 
    goto label2; // 跳入 n 和 t 的作用域
    int n; // 没有初始化器  OK 可以通过编译
    Trivial t; // 平凡构造函数/析构函数，没有初始化器  OK 可以通过编译
//  int x = 1; // 错误：有初始化器    Wrong 不能通过编译
//  Object obj2; // 错误：没有平凡析构函数  Wrong 不能通过编译
label2:
 
    {
        Object obj3;
        goto label3; // 向前跳，离开 obj3 的作用域
    }
label3: ;
 
}
```

下面的代码按照CPP标准应该时不能通过编译的：
```cpp
#include <iostream>
using namespace std;

int main()
{
        goto Exit;
        int a = 0;
Exit:
        return 0;
}
```

有几个编译OK的写法， 可以学习，但不推荐在实际编码中使用，因为移植性不好：
- 写法一:
改变域，变成局部变量：

```cpp
int main()
{
    goto Exit;
    {
        int a = 0;
    }
Exit:
        return 0;
}
```

- 写法二
变量的声明与定义分开：

```cpp
int main()
{
    goto Exit;
    int a;
    a = 1;
Exit:
    cout << "a = " << a << endl;
        return 0;
}
```



##### 编译器测试
经过测试，使用CPP编译器的情况下，linux下的 gcc clang, windows下的gcc 在编译goto语句后有变量声明定义的会报错。

windows下的clang visual studio 都可以正常编译goto语句后有变量声明定义的， 没有报错，但是clang会有warning。(所以 visual studio实际上并不遵循CPP标准)

visual stuio 也可以不允许 goto 语句跳过 变量的声明和定义。

```txt
When compiled by using /Za, a jump to the label prevents the identifier from being initialized.

You can only jump past a declaration with an initializer if the declaration is enclosed in a block that isn't entered, or if the variable has already been initialized.
```

💡The **/Za** compiler option disables and emits errors for Microsoft extensions to C that aren't compatible with ANSI C89/ISO C90. The deprecated **/Ze** compiler option enables Microsoft extensions. Microsoft extensions are enabled by default.


📒所以：如果想要代码有好的可移植性， 最好不要在 goto语句后有声明变量并定义的。


