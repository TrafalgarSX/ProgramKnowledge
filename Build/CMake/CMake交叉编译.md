### 工具链文件
---
In order to use CMake for cross-compiling, **a CMake file that describes the target platform has to be created, called the “toolchain file,” This file tells CMake everything it needs to know about the target platform.** Here is an example that uses the MinGW cross-compiler for Windows under Linux; the contents will be explained line-by-line afterwards.

```cmake
# the name of the target operating system
set(CMAKE_SYSTEM_NAME Windows)

# which compilers to use for C and C++
set(CMAKE_C_COMPILER   i586-mingw32msvc-gcc)
set(CMAKE_CXX_COMPILER i586-mingw32msvc-g++)

# where is the target environment located
set(CMAKE_FIND_ROOT_PATH  /usr/i586-mingw32msvc
    /home/alex/mingw-install)

# adjust the default behavior of the FIND_XXX() commands:
# search programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# search headers and libraries in the target environment
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

Assuming that this file is saved with the name TC-mingw.cmake in your home directory, you instruct CMake to use this file by setting the [`CMAKE_TOOLCHAIN_FILE`](https://cmake.org/cmake/help/latest/variable/CMAKE_TOOLCHAIN_FILE.html#variable:CMAKE_TOOLCHAIN_FILE "(in CMake v3.26.0)") variable:

```bash
~/src$ cd build
~/src/build$ cmake -DCMAKE_TOOLCHAIN_FILE=~/TC-mingw.cmake ..
...
```

[`CMAKE_TOOLCHAIN_FILE`](https://cmake.org/cmake/help/latest/variable/CMAKE_TOOLCHAIN_FILE.html#variable:CMAKE_TOOLCHAIN_FILE "(in CMake v3.26.0)") has to be specified only on the initial CMake run; after that, the results are reused from the CMake cache. You don’t need to write a separate toolchain file for every piece of software you want to build. The toolchain files are per target platform; i.e. if you are building several software packages for the same target platform, you only have to write one toolchain file that can be used for all packages.

### System Inspection
---
Most portable software projects have a set of system inspection tests for determining the properties of the (target) system. The simplest way to check for a system feature with CMake is by testing variables. For this purpose, CMake provides the variables [`UNIX`](https://cmake.org/cmake/help/latest/variable/UNIX.html#variable:UNIX "(in CMake v3.26.0)"), [`WIN32`](https://cmake.org/cmake/help/latest/variable/WIN32.html#variable:WIN32 "(in CMake v3.26.0)"), and [`APPLE`](https://cmake.org/cmake/help/latest/variable/APPLE.html#variable:APPLE "(in CMake v3.26.0)"). When cross-compiling, these variables apply to the target platform, for testing the build host platform there are corresponding variables [`CMAKE_HOST_UNIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_UNIX.html#variable:CMAKE_HOST_UNIX "(in CMake v3.26.0)"), [`CMAKE_HOST_WIN32`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_WIN32.html#variable:CMAKE_HOST_WIN32 "(in CMake v3.26.0)"), and [`CMAKE_HOST_APPLE`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_APPLE.html#variable:CMAKE_HOST_APPLE "(in CMake v3.26.0)").

If this granularity is too coarse, the variables [`CMAKE_SYSTEM_NAME`](https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM_NAME.html#variable:CMAKE_SYSTEM_NAME "(in CMake v3.26.0)"), [`CMAKE_SYSTEM`](https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM.html#variable:CMAKE_SYSTEM "(in CMake v3.26.0)"), [`CMAKE_SYSTEM_VERSION`](https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM_VERSION.html#variable:CMAKE_SYSTEM_VERSION "(in CMake v3.26.0)"), and [`CMAKE_SYSTEM_PROCESSOR`](https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM_PROCESSOR.html#variable:CMAKE_SYSTEM_PROCESSOR "(in CMake v3.26.0)") can be tested, along with their counterparts [`CMAKE_HOST_SYSTEM_NAME`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_SYSTEM_NAME.html#variable:CMAKE_HOST_SYSTEM_NAME "(in CMake v3.26.0)"), [`CMAKE_HOST_SYSTEM`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_SYSTEM.html#variable:CMAKE_HOST_SYSTEM "(in CMake v3.26.0)"), [`CMAKE_HOST_SYSTEM_VERSION`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_SYSTEM_VERSION.html#variable:CMAKE_HOST_SYSTEM_VERSION "(in CMake v3.26.0)"), and [`CMAKE_HOST_SYSTEM_PROCESSOR`](https://cmake.org/cmake/help/latest/variable/CMAKE_HOST_SYSTEM_PROCESSOR.html#variable:CMAKE_HOST_SYSTEM_PROCESSOR "(in CMake v3.26.0)"), which contain the same information, but for the build host and not for the target system.

```cmake
if(CMAKE_SYSTEM MATCHES Windows)
   message(STATUS "Target system is Windows")
endif()

if(CMAKE_HOST_SYSTEM MATCHES Linux)
   message(STATUS "Build host runs Linux")
endif()
```


### 更多内容
---
[Cross Compiling With CMake — Mastering CMake](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html)