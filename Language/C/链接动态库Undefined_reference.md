### undefined reference to "function name"
编译使用第三方动态库的Qt程序过程中，出现了`undefined reference`，这个**错误是链接的时候找不到对应函数的实现（函数的定义）时就会出现。**
编译展示二维码的Qt程序过程中，用到了二维码库、EtCommon库、curl等静态库一起编译成的动态库，出现了无法找到curl中的函数和EtCommon库中Linux相关的进程间通讯相关函数。
#### 问题原因
EtCommon库中无法找到函数是因为宏定义把相关的代码给关闭了。

~~curl中函数找不到是因为curl静态库链接到了EtCommon的动态库中，并没有链接到EtCommon的静态库中。（应该是EtCommon动态库依赖EtCommon静态库，EtCommon静态库依赖curl静态库，但是实际上并没有做到）~~

💡如何使用CMake将一个静态库链接到另一个静态库中？
静态库不会链接静态库， 只有动态库和可执行程序有链接的过程， 静态库只是 .o文件的集合。

