[add\_library — CMake 3.25.3 Documentation](https://cmake.org/cmake/help/v3.25/command/add_library.html)
### add_library
---
Add a library to the project using the specified source files.

#### Normal Libraries
```scss
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```
Adds a library target called `<name>` **to be built from the source files listed in the command invocation.** The `<name>` corresponds to the logical target name and must be **globally unique within a project**. The actual file name of the library built is constructed based on conventions of the native platform (such as `lib<name>.a` or `<name>.lib`).

💡New in version 3.1: Source arguments to `add_library` may use "generator expressions" with the syntax `$<...>`. See the [`cmake-generator-expressions(7)`](https://cmake.org/cmake/help/v3.25/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") manual for available expressions.
❔什么是生成表达式？

STATIC, SHARED, or MODULE may be given to specify the type of library to be created. 
- STATIC libraries are archives of object files for use when linking other targets. 
- SHARED libraries are linked **dynamically and loaded at runtime. **
- MODULE libraries are plugins that are not linked into other targets but may be loaded dynamically at runtime using dlopen-like functionality. ❓

If no type is given explicitly the type is STATIC or SHARED based on whether the current value of the variable BUILD_SHARED_LIBS is ON. 

For SHARED and MODULE libraries the `POSITION_INDEPENDENT_CODE` target property is set to ON automatically. A SHARED library may be marked with the FRAMEWORK target property to create an macOS Framework.

库文件的生成和相关存放位置
By default the library file will be created in the build tree directory corresponding to the source tree directory in which the command was invoked. See documentation of the `ARCHIVE_OUTPUT_DIRECTORY`, `LIBRARY_OUTPUT_DIRECTORY`, and `RUNTIME_OUTPUT_DIRECTORY` target properties to change this location. See documentation of the `OUTPUT_NAME` target property to change the `<name>` part of the final file name.

If `EXCLUDE_FROM_ALL` is given the corresponding property will be set on the created target. See documentation of the [`EXCLUDE_FROM_ALL`](https://cmake.org/cmake/help/v3.25/prop_tgt/EXCLUDE_FROM_ALL.html#prop_tgt:EXCLUDE_FROM_ALL "EXCLUDE_FROM_ALL") target property for details.


#### Object Libraries

#### Interface Libraries
An `INTERFACE` library target **does not compile sources and does not produce a library artifact on disk.** However, it may have properties set on it and it may be installed and exported. Typically, `INTERFACE_*` properties are populated on an interface target using the commands:

- set_property(),
- target_link_libraries(INTERFACE),
- target_link_options(INTERFACE),
- target_include_directories(INTERFACE),
- target_compile_options(INTERFACE),
- target_compile_definitions(INTERFACE), and
- target_sources(INTERFACE),
and then it is used as an argument to target_link_libraries() like any other target.

#### Imported Libraries
引入第三方库。
Creates an `IMPORTED` library target called `<name>`. **No rules are generated to build it, and the IMPORTED target property is True.** The target name has scope in the directory in which it is created and below, but the GLOBAL option extends visibility. **It may be referenced like any target built within the project.** `IMPORTED` libraries are useful for convenient reference from commands like target_link_libraries(). Details about the imported library are specified by setting properties whose names begin in `IMPORTED_` and `INTERFACE_`.


#### Alias Libraries