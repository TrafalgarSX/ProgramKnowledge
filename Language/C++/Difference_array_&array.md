### What's the difference between array and &array?
```c
int array[16];
void *p = array;
void *q = &array;
printf("%p\n", p);
printf("%p\n", q);
```

这个时候输出的地址值是一样的。那 array 和 &array 本质上是一样的吗？  它们的类型是相同的吗？
结论是：  并不是。
- The type of `&array` is `int (*)[16]` (**a pointer to an array of 16 integers**). 
  >&array是指向数组的指针， 当（&array) + 1时， 移动的是一个数组的大小。
  
- The type of `array`, when left to decay, is `int*` (**a pointer to an integer**). 

They both point to the **same location**, **but** have a different meaning to the compiler.

If you do `(&array)[0]`, the value you end up with is the original array of 16 integers, that you can subscript again, like `(&array)[0][0]`. `(&array)[1]` would be the next array of 16 integers, if there was one.

If you do `array[0]`, the value you end up with is an integer, which you can't subscript again. `array[1]` is just the next integer. (This is true regardless of if `array` is a `int[16]` or a `int*`.)

💡Obviously, if you then turn your pointers into `void` pointers, you lose any semantic difference there could have been.

解释：
Basically, “_array_” is a “_**pointer to the first element of array**_” but “_&array_” is a “_**pointer to whole array of 5 int**_”. 

Since “_array_” is pointer to _int_, addition of 1 resulted in an address with increment of 4 (assuming _int_ size in your machine is 4 bytes). 

Since “_&array_” is pointer to array of 5 ints, addition of 1 resulted in an address with increment of 4 x 5 = 20 = 0x14. 

Now you see why these two seemingly similar pointers are different at core level. 

This logic can be extended to multidimensional arrays as well. Suppose `double twoDarray[5][4]` is a 2D array. Here, “twoDarray” is a pointer to array of 4 int but “&twoDarray” is pointer to array of 5 rows arrays of 4 int”. If this sounds cryptic, you can always have a small program to print these after adding 1. We hope that we could clarify that any _array name itself_ is a _pointer to the first element_ but _& (i.e. address-of) for the array name_ is a _pointer to the whole array_ itself.