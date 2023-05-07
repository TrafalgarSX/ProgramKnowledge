### CMake中的一些关键概念
---
This chapter provides an introduction to CMake’s key concepts. As you start working with CMake, you will run into a variety of concepts such as **targets, generators, and commands.** 

Understanding these concepts will provide you with the working knowledge you need to create effective CMakeLists files. Many CMake objects such as targets, directories and source files have properties associated with them. 

A property is a key-value pair attached to a specific object. The most generic way to access properties is through the `set_property` and `get_property` commands. These commands allow you to set or get a property from any object in CMake that has properties. See the cmake-properties manual for a list of supported properties. From the command line a full list of properties supported in CMake can be obtained by running cmake with the --help-property-list option.

#### Targets
Probably the most important item is targets. **Targets represent executables, libraries, and utilities built by CMake**. Every [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "(in CMake v3.26.0)"), [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "(in CMake v3.26.0)"), and [`add_custom_target`](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target "(in CMake v3.26.0)") command creates a target. For example, the following command will **create a target named “foo” that is a static library**, with `foo1.c` and `foo2.c` as source files.

```cmake
add_library(foo STATIC foo1.c  foo2.c)
```

Likewise, executables have some options. By default, an executable will be a traditional console application that has a main entry point. **One may specify a `WIN32` option to request a WinMain entry point on Windows systems, while retaining main on non-Windows systems.**

Targets also store the link directories to use when linking, and custom commands to execute after building.

#### Usage Requirements
CMake will also **propagate “usage requirements” from linked library targets**. Usage requirements affect compilation of sources in the `<target>`. They are specified by properties defined on linked targets.

#### Specifying Optimized or Debug Libraries with a Target
On Windows platforms, users are often required to link debug libraries with debug libraries, and optimized libraries with optimized libraries. CMake helps satisfy this requirement with the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "(in CMake v3.26.0)") command, which accepts an optional flag labeled as `debug` or `optimized`. If a library is preceded with either `debug` or `optimized`, then that library will only be linked in with the appropriate configuration type. For example
```
add_executable(foo foo.c)
target_link_libraries(foo debug libdebug optimized libopt)
```
In this case, foo will be linked against libdebug if a debug build was selected, or against libopt if an optimized build was selected.

#### Object Libraries
Large projects often organize their source files into groups, perhaps in separate subdirectories, that each need different include directories and preprocessor definitions. For this use case CMake has developed the concept of Object Libraries.

An Object Library is a collection of source files compiled into an object file which is not linked into a library file or made into an archive. Instead other targets created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "(in CMake v3.26.0)") or [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "(in CMake v3.26.0)") may reference the objects using an expression of the form `$<TARGET_OBJECTS:name>` as a source, where “name” is the target created by the [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "(in CMake v3.26.0)") call. For example:
```
add_library(A OBJECT a.cpp)
add_library(B OBJECT b.cpp)
add_library(Combined $<TARGET_OBJECTS:A> $<TARGET_OBJECTS:B>)
```
will include A and B object files in a library called Combined. Object libraries may contain only sources (and headers) that compile to object files.