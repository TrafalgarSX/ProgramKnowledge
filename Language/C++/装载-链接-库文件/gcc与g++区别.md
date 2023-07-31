GCC:GNU Compiler Collection(GUN 编译器集合)，它可以编译C、C++、JAV、Fortran、Pascal、Object-C等语言。

gcc是GCC中的GUN C Compiler（C 编译器）

g++是GCC中的GUN C++ Compiler（C++编译器）

gcc和g++的主要区别

1. 对于 *.c和*.cpp文件，gcc分别当做c和cpp文件编译（c和cpp的语法强度是不一样的）
	- 对于以 `.c` 为扩展名的文件，`GCC` 会自动将其视为 `C` 源代码文件
	- 对于以 `.cpp` 为扩展名的文件，`GCC` 会自动将其视为 `C++` 源代码文件

  i.e. gcc不仅只会调用C compiler， 它会检测文件后缀名， 使用对应的编译器， 如果你想忽略文件后缀名的影响，可以使用 `-x` 参数。
  
  ```
  gcc -xc++ test.c -o test.exe
  上面的命令 gcc 会将test.c当作cpp文件编译
```

2. 对于 *.c和*.cpp文件，g++则统一当做cpp文件编译
  g++不检测文件后缀名，同一当作 cpp 文件编译

3. 使用g++编译文件时，**g++会自动链接标准库STL，而gcc不会自动链接STL**
  由于这个特性，当gcc编译cpp文件的时候，如果文件中调用了 STL库，e.g. iostream string, 会报编译错误。

4. gcc在编译C文件时，可使用的预定义宏是比较少的

5. gcc在编译cpp文件时/g++在编译c文件和cpp文件时（这时候gcc和g++调用的都是cpp文件的编译器），会加入一些额外的宏。

6.在用gcc编译c++文件时，为了能够使用STL，需要加参数 –lstdc++ ，但这并不代表 gcc –lstdc++ 和 g++等价，它们的区别不仅仅是这个。


如果一个工程中有 .c .cpp的文件需要混合C和CPP进行编译， 像cmake make这样的工具会自动使用对应的编译器进行编译， .c会用 gcc .cpp会用 g++

visual studio 也是，对.c文件用C编译器， .cpp文件用 cpp编译器


![[20201205101836382.png]]