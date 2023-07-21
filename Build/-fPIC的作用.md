### 静态库可以使用—fPIC编译选项吗？
---
是的，静态库可以使用 `-fPIC` 选项。

`-fPIC`（Position Independent Code）是一种编译选项，用于生成与**位置无关的代码**。它通常用于编译共享对象（动态库），以**允许动态库在内存中的任何位置加载和重定位。**

尽管 `-fPIC` 主要用于编译共享对象，**但在某些情况下**，也可以在编译静态库时使用 `-fPIC`。这种情况可能包括：

1. **静态库将在共享对象中使用**：如果你计划将**静态库与共享对象链接，并且静态库中的某些函数将在共享对象中使用**，那么使用 `-fPIC` 编译静态库是有意义的。这样可以确保静态库中的代码是与位置无关的，以便与共享对象正确链接和重定位。

2. **平台或编译器要求**：某些平台或编译器可能要求在编译静态库时使用 `-fPIC`，即使静态库不会直接用于共享对象。这可能是为了确保生成的静态库在某些情况下仍然可以与位置无关的代码一起使用。

需要注意的是，使用 `-fPIC` 编译静态库**可能会增加代码的大小和执行开销**。因此，在编译静态库时，应根据具体情况和需求权衡利弊，并根据目标平台和编译器的要求进行选择。

### ./configure如何设置-fPIC选项
---
要在 `./configure` 脚本中设置 `-fPIC` 选项，可以使用环境变量 `CFLAGS` 或 `CXXFLAGS` 来传递编译选项。
在运行 `./configure` 之前，先设置 `CFLAGS` 或 `CXXFLAGS` 环境变量，如下所示：
```bash
export CFLAGS="-fPIC"
./configure
```

或者，如果你使用的是 C++ 编译器，可以设置 `CXXFLAGS` 环境变量：
```bash
export CXXFLAGS="-fPIC"
./configure
```

### cmake中设置-fPIC选项
---
```cmake
target_compile_options(-fPIC)

或者
set（CMAKE_C_FLAGS“$ {CMAKE_C_FLAGS} -fPIC”）
set（CMAKE_CXX_FLAGS“$ {CMAKE_CXX_FLAGS} -fPIC”）

或者
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
```

也可以从命令行设置
```bash
cmake .. -DCMAKE_POSITION_INDEPENDENT_CODE=ON
```
