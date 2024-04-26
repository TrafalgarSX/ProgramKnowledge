
```c
  void myprint(char* x) {
      printf("%s\n", x); 
  }

  int main() {
     char* s = "hello";
     void (*test)(char*);
     void (*test2)(char*);

     test = myprint;
     test2 = &myprint;

     test(s);
     (*test)(s);
     test2(s);
     (*test2)(s);

  }
```

This is just a quirk of C. **There's no other reason but the C standard just says that dereferencing or taking the address of a function just evaluates to a pointer to that function, and dereferencing a function pointer just evaluates back to the function pointer**.

[c - Function Pointer - Automatic Dereferencing - Stack Overflow](https://stackoverflow.com/questions/7518815/function-pointer-automatic-dereferencing?noredirect=1&lq=1)

Functions and references to functions decay to a function pointer whenever necessary. There is no dereference operator defined for functions but there is one for function pointer: the function or reference to function happily decays to a pointer just to become derefernced again.

[c++ - Why is it possible to use the dereferencing operator multiple times when assigning a function to a function pointer? - Stack Overflow](https://stackoverflow.com/questions/41097868/why-is-it-possible-to-use-the-dereferencing-operator-multiple-times-when-assigni?noredirect=1&lq=1)

#### 从标准解释
On the one hand, according to the C Standard (6.3.2.1 Lvalues, arrays, and function designators)

> 4 A function designator is an expression that has function type. Except when it is the operand of the sizeof operator65) or the unary & operator, **a function designator with type ‘‘function returning type’’ is converted to an expression that has type ‘‘pointer to function returning type**’’.

and according to the C++ Standard (4.3 Function-to-pointer conversion)

> 1 **An lvalue of function type T can be converted to a prvalue of type “pointer to T.” The result is a pointer to the function**

On the other hand according to the C Standard(6.5.3.2 Address and indirection operators)

> 4 The unary * operator denotes indirection. **If the operand points to a function, the result is a function designator**;

and the C++ Standard (5.3.1 Unary operators)

> 1 The unary * operator performs indirection: the expression to which it is applied shall be a pointer to an object type, or a pointer to a function type and **the result is an lvalue referring to the object or function to which the expression points**. If the type of the expression is “pointer to T,” the type of the result is “T.”

Thus there are recursive conversions.at first from the function designator to pointer and then from the pointer to the function designator (or as written in the C++ Standard to lvalue referring the function) and again from the function designator or lvalue to function pointer and so on.


按照&运算符本来的意义，它要求其操作数是一个对象，但函数名不是对象（函数是一个对象），本来&test是非法的，但很久以前有些编译器已经允许这样做，   
c/c++标准的制定者出于对象的概念已经有所发展的缘故，也承认了&test的合法性。   
  
**因此，对于test和&test你应该这样理解，test是函数的首地址，它的类型是void ()，&test表示一个指向函数test这个对象的地址，**   
**它的类型是void (*)()，因此test和&test所代表的地址值是一样的，但类型不一样。test是一个函数，&test表达式的值是一个指针！**