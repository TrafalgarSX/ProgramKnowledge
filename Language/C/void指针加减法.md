### 大坑， 最好不要用`void*`做加减法
clang 和 gcc 可以编译通过， 但是vs编译不通过， 所以不推荐使用，平台移植性很差

[Pointer (computer programming)](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Pointer_%28computer_programming%29%23C_and_C.2B.2B)  

Pointer arithmetic cannot be performed on void pointers because the

[void type](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Void_type)

has no size, and thus the pointed address can not be added to, although **[gcc](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/GNU_Compiler_Collection) and other compilers will perform byte arithmetic on void* as a non-standard extension, treating it as if it were char ***.

[Pointer Arith](https://link.zhihu.com/?target=http%3A//gcc.gnu.org/onlinedocs/gcc-4.8.0/gcc/Pointer-Arith.html)  

In GNU C, addition and subtraction operations are supported on pointers to void and on pointers to functions. This is done by **treating the size of a** **void** **or of a function as 1**.


### stackoverflow answer
[c - Warning: pointer of type 'void \*' used in subtraction - Stack Overflow](https://stackoverflow.com/questions/2710923/warning-pointer-of-type-void-used-in-subtraction)
Additions and subtractions with pointers work with the size of the pointed type:

```c
int* foo = 0x1000;
foo++;
// foo is now 0x1004 because sizeof(int) is 4
```

Semantically speaking, the size of `void` should be zero, since it doesn't represent anything. For this reason, pointer arithmetic on `void` pointers **should be illegal.**

However, for several reasons, `sizeof(void)` returns 1, and arithmetic works as if it was a `char` pointer. **Since it's semantically incorrect, you do, however, get a warning.**

To suppress the warning, use `char` pointers.


### another so answer
[Pointer arithmetic for void pointer in C - Stack Overflow](https://stackoverflow.com/questions/3523145/pointer-arithmetic-for-void-pointer-in-c?rq=3)
Final conclusion: arithmetic on a `void*` is **illegal** in both C and C++.

**GCC allows it as an extension**, see [Arithmetic on `void`- and Function-Pointers](https://gcc.gnu.org/onlinedocs/gcc/Pointer-Arith.html) (note that this section is part of the "C Extensions" chapter of the manual). 

Clang and ICC likely allow `void*` arithmetic for the purposes of compatibility with GCC. 

Other compilers (such as MSVC) disallow arithmetic on `void*`, and GCC disallows it if the `-pedantic-errors` flag is specified, or if the `-Werror=pointer-arith` flag is specified (this flag is useful if your code base must also compile with MSVC).


[void指针及其应用，C语言void指针及使用注意事项详解](http://c.biancheng.net/view/365.html)
### The C Standard Speaks

>6.5.6-2: For addition, either **both operands shall have arithmetic type, or one operand shall be a pointer to an object type and the other shall have integer type.**

