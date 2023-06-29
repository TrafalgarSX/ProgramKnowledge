### CMake中如何开启sanitize
---
想在程序中使用sanitize去检测内存泄漏，工程使用的是CMake管理的，怎样在CMake中开启sanitize呢？

在命令行中通过编译器开启sanitize是这样的：
```bash
clang++ -fsanitize-address -fno-omit-frame-pointer -g test.cpp -o test
```
编译器也可以是g++ gcc  clang等

如何让CMake中也起到相同的作用呢？
The idea of this solution is to pass -fsanitize=address to the compiler and linker flags.

##### 方式一
```cmake
list(APPEND EXTRA_COMPILE_FLAGS "-fsanitize=address" "-fno-omit-frame-pointer")
list(APPEND EXTRA_LINK_FLAGS "-fsanitize=address"  "-fno-omit-frame-pointer")

# 链接选项
target_link_options(${PROJECT_NAME} PUBLIC ${EXTRA_LINK_FLAGS})
# 编译选项
target_compile_options(${PROJECT_NAME} PUBLIC ${EXTRA_COMPILE_FLAGS})
```

要注意的是， 如果通过命令行开启sanitize, 只需要带有"-fsanitize=address"等参数即可，但是在CMake中要注意链接选项和编译选项都要带有相关的参数。

如果想同时为所有target启用此功能，可以使用：
```cmake
add_compile_options(-fsanitize=address)
add_link_options(-fsanitize=address)
```

##### 方式二
如果有些CMakeLists.txt中没有使用`target_link_options`这种方式，而是使用 `FLAGS`的方式的话，用下面的脚本：
```cmake
set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
set (CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_LINKER_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
```

或者下面这种方式：
```cmake
get_property(isMultiConfig GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)

if(isMultiConfig)
    if(NOT "Asan" IN_LIST CMAKE_CONFIGURATION_TYPES)
        list(APPEND CMAKE_CONFIGURATION_TYPES Asan)
    endif()
else()
    set(allowedBuildTypes Asan Debug Release RelWithDebInfo MinSizeRel)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${allowedBuildTypes}")

    if(CMAKE_BUILD_TYPE AND NOT CMAKE_BUILD_TYPE IN_LIST allowedBuildTypes)
        message(FATAL_ERROR "Invalid build type: ${CMAKE_BUILD_TYPE}")
    endif()
endif()

set(CMAKE_C_FLAGS_ASAN
    "${CMAKE_C_FLAGS_DEBUG} -fsanitize=address -fno-omit-frame-pointer" CACHE STRING
    "Flags used by the C compiler for Asan build type or configuration." FORCE)

set(CMAKE_CXX_FLAGS_ASAN
    "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address -fno-omit-frame-pointer" CACHE STRING
    "Flags used by the C++ compiler for Asan build type or configuration." FORCE)

set(CMAKE_EXE_LINKER_FLAGS_ASAN
    "${CMAKE_EXE_LINKER_FLAGS_DEBUG} -fsanitize=address" CACHE STRING
    "Linker flags to be used to create executables for Asan build type." FORCE)

set(CMAKE_SHARED_LINKER_FLAGS_ASAN
    "${CMAKE_SHARED_LINKER_FLAGS_DEBUG} -fsanitize=address" CACHE STRING
    "Linker lags to be used to create shared libraries for Asan build type." FORCE)
```