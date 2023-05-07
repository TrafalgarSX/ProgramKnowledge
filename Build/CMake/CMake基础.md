### CMake基础变量和命令
```cmake
# 设置项目的最低的cmake版本要求
cmake_minimum_required（VERSION <min>)

# prject命令设置项目的名称，并将其保存在变量 PROJECT_NAME中，如果是顶层的 CMakeLists.txt,还将项目名称存储在变量 CMAKE_PROJECT_NAME 中
project(<PROJECT_NAME> [...])

# 查找当前目录下的所有源文件，并将名称保存到 VARNAME变量
aux_source_directory(. VARNAME)

# 指明本项目包含一个子目录， 子目录下的CMakeListst.txt文件和源代码也会被处理
add_subdirectory(dir)

# 指定可执行文件名称和使用的源文件, windows平台会补全 .exe 还有其他用法，这个最常用
add_executable(<name> [source1] [source2] ...)

# 生成动态库
add_library({PROJECT_NAME} SHARED [source1] [source2])
# 静态库
add_library({PROJECT_NAME} STATIC [source1] [source2])
# 链接静态库或动态库 目标可以是 库 或者 可执行文件
target_link_libraries({PROJECT_NAME}  libname libname)

# 版本控制动态库， 生成带有版本号的软链接指向 动态库
set_target_properties({PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION})

# 安装库文件和头文件，编译时将编译出的库文件和头文件拷贝到系统对应的环境变量路径下
install(TARGETS libname libnamea
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
install(FILES header.h DESTINATION include/header/)

编译时执行： cmake -DCMAKE_INSTALL_PREFIX=/usr ..


# 找到某个目录下所有的相关文件
file(GLOB var_name "${var_name}/*.so") 

# 复制文件到另一个目录
file(COPY ${var_name} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

# 如果第三方库里有.cmake文件可以直接调用find_package找路径
find_package(OpenCV REQUIRED)

# 头文件目录
include_directories(${var_name_header_dir})
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

# 库目录
lib_directories(${var_name_lib_dir})

# 循环
foreach(var IN LISTS vars)
	get_filename_component(var_name ${var} NAME_WE)
endforeach()
```

```cmake
变量
CMAKE_MAKE_PROGRAM:
Tool that can launch the native build system. The value may be the full path to an executable or just the tool name if it is expected to be in the PATH.
e.g. make nmake gmake ninja xcodebuild MSBuild.exe
MSBuild.exe 相关两个变量
CMAKE_VS_MSBUILD_COMMAND  CMAKE_VS_DEVENV_COMMAND.


```

#### 自定义编译选项
CMake 允许为项目增加编译选项，从而可以根据用户的环境和需求选择最合适的编译方案。
```CMake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo4)

# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  )

# 是否使用自己的 MathFunctions 库
option (USE_MYMATH
       "Use provided math implementation" ON)

# 是否加入 MathFunctions 库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/math")
  add_subdirectory (math)  
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries (Demo  ${EXTRA_LIBS})
```

1. 7行的configure_file命令用于加入一个配置头文件config.h, 这个文件有CMake从config.h.in生成，通过这样的机制，**将可以通过预定义一些参数和变量来控制代码的生成。**
2. option命令添加了一个USE_MYMATH选项，并且默认值位ON 

config.h被c/c++源文件引用，但我们并不直接编写这个文件，我们直接编写config.h.in:
```cmake
#cmakedefine USE_MYMATH
```
config.h被自动生成

`ccmake`命令或 `cmake -i`命令可以提供一个会话式的交互式配置界面

#### 安装和测试
CMake指定安装规则和添加测试。这两个功能分别可以通过在产生 Makefile 后使用 `make install` 和 `make test` 来执行。

##### 定制安装规则
```cmake
# 指定 MathFunctions 库的安装路径  
install (TARGETS MathFunctions DESTINATION bin)  
install (FILES MathFunctions.h DESTINATION include)

# 指定安装路径
install (TARGETS Demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h"
         DESTINATION include)
```
生成的 Demo 文件和 MathFunctions 函数库 libMathFunctions.o 文件将会被复制到 /usr/local/bin 中，而 MathFunctions.h 和生成的 config.h 文件则会被复制到 /usr/local/include 中。

/usr/local是默认安装到的根目录，可以通过修改`CMAKE_INSTALL_PREFIX`变量的值来指定这些文件应该拷贝到哪个根目录。

Then, run the install step by using the [`--install`](https://cmake.org/cmake/help/v3.25/manual/cmake.1.html#cmdoption-cmake-install) option of the [`cmake`](https://cmake.org/cmake/help/v3.25/manual/cmake.1.html#manual:cmake(1) "cmake(1)") command (introduced in 3.15, older versions of CMake must use `make install`) from the command line. This step will install the appropriate header files, libraries, and executables. For example:
```
cmake --install .

cmake --install . --prefix "path/you/want/to/install"
```

The [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX "(in CMake v3.26.0)") variable can be set in the CMake cache to specify where to install the provided software. If the provided software has install rules, specified using the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install "(in CMake v3.26.0)") command, they will install artifacts into that prefix. On Windows, the default installation location corresponds to the `ProgramFiles` system directory which may be architecture specific. On Unix hosts, `/usr/local` is the default installation location.

##### 添加测试
CMake 提供了一个称为 CTest 的测试工具。我们要做的只是在项目根目录的 CMakeLists 文件中调用一系列的 `add_test` 命令。
```cmake
# 启用测试
enable_testing()

# 测试程序是否成功运行
add_test (test_run Demo 5 2)

# 测试帮助信息是否可以正常提示
add_test (test_usage Demo)
set_tests_properties (test_usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")

# 测试 5 的平方
add_test (test_5_2 Demo 5 2)

set_tests_properties (test_5_2
 PROPERTIES PASS_REGULAR_EXPRESSION "is 25")

# 测试 10 的 5 次方
add_test (test_10_5 Demo 10 5)

set_tests_properties (test_10_5
 PROPERTIES PASS_REGULAR_EXPRESSION "is 100000")

# 测试 2 的 10 次方
add_test (test_2_10 Demo 2 10)

set_tests_properties (test_2_10
 PROPERTIES PASS_REGULAR_EXPRESSION "is 1024")
```

##### Adding Support for a Testing Dashboard
Adding support for submitting our test results to a dashboard is simple. We already defined a number of tests for our project in [Testing Support](https://cmake.org/cmake/help/v3.25/guide/tutorial/Installing%20and%20Testing.html#tutorial-testing-support). Now we just have to run those tests and submit them to a dashboard.

Replace:
```
enable_testing()
```
With
```
include(CTest)
```

The [`CTest`](https://cmake.org/cmake/help/v3.25/module/CTest.html#module:CTest "CTest") module will automatically call `enable_testing()`, so we can remove it from our CMake files.

We will also need to acquire a `CTestConfig.cmake` file to be placed in the top-level directory where we can specify information to CTest about the project. It contains:

-   The project name
-   The project "Nightly" start time
    -   The time when a 24 hour "day" starts for this project.
-   The URL of the CDash instance where the submission's generated documents will be sent

```
ctest [-VV] -D Experimental
```

##### 支持调试
```cmake
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

💡if CMAKE_BUILD_TYPE is empty then no flags are added to the build.

##### 添加版本号
首先修改顶层 CMakeLists 文件，在 `project` 命令之后加入如下两行：

```scss
set (Demo_VERSION_MAJOR 1)
set (Demo_VERSION_MINOR 0)
```

分别指定当前的项目的主版本号和副版本号。

##### 生成安装包
本节将学习如何配置生成各种平台上的安装包，包括**二进制安装包和源码安装包**。为了完成这个任务，我们需要用到 **CPack** ，它同样也是由 CMake 提供的一个工具，专门用于打包。

在顶层CMakeLists.txt文件尾部添加：
```
# 构建一个 CPack 安装包
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
  "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Demo_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Demo_VERSION_MINOR}")
include (CPack)
```
上面的代码做了以下几个工作：

1. 导入 InstallRequiredSystemLibraries 模块，以便之后导入 CPack 模块；
2. 设置一些 CPack 相关变量，包括版权信息和版本信息，其中版本信息用了上一节定义的版本号；
导入 CPack 模块。
3. 接下来的工作是像往常一样构建工程，并执行 cpack 命令。

生成二进制安装包：
`cpack -C CPackConfig.cmake`
生成源码安装包
`cpack -C CPackSourceConfig.cmake`
