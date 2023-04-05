### 什么是ABI？
CPU与指令集的每种组合都有专用的应用二进制接口（ABI）。ABI包含以下信息：
- 可使用的CPU指令集（和扩展指令集）
- 运行时内存存储和加载的字节顺序。（大端小端，Android始终是little-endian.)
- 在应用和系统之间传递数据的规范（包括对齐限制），以及系统调用函数时如何使用堆栈和寄存器(参数传递顺序等)。
- 可执行二进制文件（例如程序和共享库）的格式，以及他们支持的内容类型。Android始终使用ELF。请参阅 [ELF System V 应用二进制接口](https://refspecs.linuxfoundation.org/elf/gabi4+/contents.html)
- 如何重整C++名称。如需了解详情，请参阅 [Generic/Itanium C++ ABI](http://itanium-cxx-abi.github.io/cxx-abi/)