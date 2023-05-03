目前都是chatGPT给出的答案，以后发现错误再修正

有点理解无能， 这个用的好像很少，以后用到再深入理解吧
### INTERFACE的作用
---
[[add_library#Interface Libraries]]
在CMake中，INTERFACE的作用是定义一个接口库或目标。接口库是一种特殊的库，其目的是为其他目标（如可执行文件或库）**提供编译器和链接器选项，而不是提供实际的代码库。**

具体来说，INTERFACE库可以定义以下内容：

1. 头文件路径和包含选项，使得其他目标可以使用它们的头文件。
2. 编译器选项和宏定义，使得其他目标可以使用相同的编译器选项和宏定义。
3. 链接器选项和依赖库，使得其他目标可以使用相同的链接器选项和依赖库。

通过使用INTERFACE库，可以将这些选项和依赖关系从一个目标传递到另一个目标，从而简化项目的构建和维护。

例如，如果一个库需要使用某个外部库，可以将这个外部库定义为一个INTERFACE库，并将其依赖关系添加到需要使用它的库的目标中。这样，如果外部库的路径或名称发生变化，只需要在一个地方修改INTERFACE库的定义，而不是在所有使用它的目标中修改依赖关系。

总之，INTERFACE库提供了一种灵活且强大的方式来管理编译器和链接器选项以及依赖关系，从而简化了项目的构建和维护。

### Example
---
当使用CMake构建一个复杂的项目时，**使用INTERFACE库可以帮助我们减少代码的重复和提高项目的可维护性。** 下面是几个使用INTERFACE库的例子： 
1. 将头文件路径传递给其他目标
```cmake
add_library(mylib INTERFACE)
target_include_directories(mylib INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

在这个例子中，我们创建了一个名为mylib的INTERFACE库，并将其头文件路径添加到CMAKE_CURRENT_SOURCE_DIR/include。通过target_include_directories命令，**其他目标可以包含mylib的头文件路径。**

2. 将编译器选项传递给其他目标
```cmake
add_library(mylib INTERFACE)
target_compile_options(mylib INTERFACE -Wall -Wextra)
```

在这个例子中，我们创建了一个名为mylib的INTERFACE库，并将其编译器选项设置为-Wall和-Wextra。**其他目标可以通过target_compile_options命令使用相同的编译器选项。**

3. 将依赖关系传递给其他目标
```cmake
add_library(mylib INTERFACE)
target_link_libraries(mylib INTERFACE otherlib)
```

在这个例子中，我们创建了一个名为mylib的INTERFACE库，并将其依赖关系设置为otherlib。**其他目标可以通过target_link_libraries命令使用相同的依赖关系。** 

4. 在不同的目标之间共享INTERFACE库
```cmake
add_library(mylib INTERFACE)
target_include_directories(mylib INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_executable(myexe main.cpp)
target_link_libraries(myexe PRIVATE mylib)
```

在这个例子中，我们创建了一个名为mylib的INTERFACE库，并将其头文件路径添加到CMAKE_CURRENT_SOURCE_DIR/include。然后，我们创建了一个可执行文件myexe，并将其链接到mylib。这样，myexe就可以使用mylib的头文件路径了。 

总之，INTERFACE库提供了一种灵活且强大的方式来管理编译器和链接器选项以及依赖关系。在大型项目中，使用INTERFACE库可以帮助我们减少代码的重复和提高项目的可维护性。

cmake25_tutorial step5中的例子：
add_library()命令创建了一个名为tutorial_compiler_flags的INTERFACE库，**这意味着该库不包含任何源文件或链接库，只是用来传递编译器标志和其他编译配置属性**。这些编译器标志和配置属性可以通过target_compile_options()、target_compile_definitions()、target_include_directories()和target_compile_features()等命令**传递给其他目标库或可执行文件。**

当其他库或可执行文件链接了tutorial_compiler_flags库时，该库中定义的编译器标志和配置属性会被传递给链接的目标库或可执行文件。这意味着其他目标库或可执行文件可以使用tutorial_compiler_flags库中定义的编译器标志和配置属性，以确保它们在编译时具有相同的设置。这样**可以确保不同的目标库或可执行文件使用相同的编译器标志和配置属性**，从而提高代码的可移植性和一致性。