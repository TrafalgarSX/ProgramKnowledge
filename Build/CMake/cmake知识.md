### 零碎分散的cmake小知识
---
##### （不推荐）检测编译器并设置
```cmake
# 检测 Clang 编译器是否存在
find_program(CLANG_COMPILER clang)

# 如果 Clang 编译器存在，则使用 Clang，否则使用 GCC
if(CLANG_COMPILER)
    set(CMAKE_C_COMPILER ${CLANG_COMPILER})
    set(CMAKE_CXX_COMPILER ${CLANG_COMPILER}++)
else()
    message("Clang compiler not found, using GCC")
    set(CMAKE_C_COMPILER gcc)
    set(CMAKE_CXX_COMPILER g++)
endif()

```

##### cmake中的list可以添加在前面而不是append吗?
```cmake
# 在列表的开头插入值
list(INSERT my_list 0 "item2")
```

##### list在不同子目录中是共享的吗？
在不同的子目录中，列表（list）是不共享的。**每个子目录都有自己独立的作用域，包括变量和列表。**

当你在一个子目录中定义一个列表，它**只在该子目录的作用域内可见**。其他子目录无法直接访问该列表。如果你想在不同的子目录之间共享列表，你需要将列表传递给这些子目录。

##### 所以subdirectory之间target是共享的， 但是变量不是共享的， 对吗？
  是的，对于子目录之间的变量和目标的共享情况，可以总结如下：

1. 目标（Target）：**子目录之间的目标是共享的**。也就是说，一个子目录中定义的目标可以在另一个子目录中使用。你可以使用 `target_link_libraries()` 命令将一个子目录中的目标链接到另一个子目录中的目标上。

2. 变量（Variable）：**子目录之间的变量是不共享的**。每个子目录都有自己独立的作用域，变量在一个子目录中定义，不会自动传递给其他子目录。如果你希望在子目录之间共享变量，可以使用 CMake 的函数或宏来传递变量值。

3. 需要注意的是，CMake 中的变量作用域是在定义它们的 CMakeLists.txt 文件中。**如果你在主 CMakeLists.txt 中定义了一个变量，它只在主 CMakeLists.txt 和其包含的子目录中可见**。如果你在子目录中定义了一个变量，它只在该子目录及其包含的子目录中可见。因此，确保在正确的作用域内定义和操作变量是很重要的。

需要注意的是，子目录之间的使用顺序也很重要。

##### CMAKE_RUNTIME_OUTPUT_DIRECTORY 这个变量在父目录和子目录中被定义成不同的值，是以最后一次定义为准吗？
是的，对于 CMAKE_RUNTIME_OUTPUT_DIRECTORY 变量，在父目录和子目录中被定义成不同的值时，最后一次定义的值会起作用。

CMAKE_RUNTIME_OUTPUT_DIRECTORY 是一个 CMake 内置的变量，用于指定可执行文件的输出路径。当你在 CMakeLists.txt 文件中设置 CMAKE_RUNTIME_OUTPUT_DIRECTORY 变量时，CMake 将会使用该路径来存储生成的可执行文件。

如果在父目录和子目录中都定义了 CMAKE_RUNTIME_OUTPUT_DIRECTORY 变量，并且值不同，那么最后一次定义的值将会覆盖之前的定义。所以，最终生成的可执行文件将会按照最后一次定义的路径进行输出。

需要注意的是，在子目录中定义的 CMAKE_RUNTIME_OUTPUT_DIRECTORY 变量只会影响该子目录及其子目录中的可执行文件的输出路径，不会影响父目录或其他子目录中的可执行文件的输出路径。如果你希望在整个项目中统一设置可执行文件的输出路径，建议在主 CMakeLists.txt 中定义该变量。

##### cmake字符串拼接有哪些方式?
在 CMake 中，有几种方式可以进行字符串拼接：

1. 使用 `${var}` 语法：可以通过将变量用花括号括起来来进行字符串插值。例如，`${var1}${var2}` 将会将 `var1` 和 `var2` 的值拼接在一起。

2. 使用 `string(APPEND var string)` 命令：可以使用 `string(APPEND var string)` 命令将字符串连接到变量 `var` 的末尾。例如，`string(APPEND myVar "Hello, ")` 将会将 "Hello, " 追加到 `myVar` 的末尾。

3. 使用 `string(CONCAT var string1 string2 ...)` 命令：可以使用 `string(CONCAT var string1 string2 ...)` 命令将多个字符串连接到变量 `var` 中。例如，`string(CONCAT myVar "Hello, " "world!")` 将会将 "Hello, " 和 "world!" 连接在一起赋值给 `myVar`。

4. 使用 `set(var "${var1}${var2}")` 命令：可以使用 `set(var "${var1}${var2}")` 命令将两个变量的值拼接在一起，并将结果赋值给新的变量 `var`。例如，`set(myVar "${var1}${var2}")` 将会将 `var1` 和 `var2` 的值拼接在一起，并将结果赋值给 `myVar`。

这些是在 CMake 中进行字符串拼接的常用方式。使用适合你的场景的方法来进行字符串拼接。