### PUBLIC_PRIVATE_INTERFACE

下面两个链接解释了这三个的不同，知乎链接解释的很清楚

[CMake的链接选项：PRIVATE，INTERFACE，PUBLIC - 知乎](https://zhuanlan.zhihu.com/p/493493849)

[CMake target\_link\_libraries Interface Dependencies - Stack Overflow](https://stackoverflow.com/questions/26037954/cmake-target-link-libraries-interface-dependencies)

官方文档的相关知识的解释。
[cmake-buildsystem(7) — CMake 3.15.7 Documentation](https://cmake.org/cmake/help/v3.15/manual/cmake-buildsystem.7.html#transitive-usage-requirements)
这三者改变了什么，为什么要用不同的属性
Using PUBLIC does add your dependencies to projects linking to your library. Using PRIVATE does NOT add your dependencies to projects linking to your library. **It is cleaner and it also avoids possible conflicts between your dependencies and your user's**

 suppose you are creating a library A that uses some Boost headers. You would do:

- target_include_directories(A PRIVATE ${Boost_INCLUDE_DIRS}) if you only use those Boost headers inside your source files (.cpp) or private header files (.h).

- target_include_directories(A INTERFACE ${Boost_INCLUDE_DIRS}) if you don't use those Boost headers inside your source files (therefore, not needing them to compile A). I can't actually think of a real-world example for this.

- target_include_directories(A PUBLIC ${Boost_INCLUDE_DIRS}) if you use those Boost headers in your public header files, which are included BOTH in some of A's source files and might also be included in any other client of your A library.

另一种解释：
If you are creating a shared library and your source cpp files `#include` the headers of another library (Say, QtNetwork for example), but your header files don't include QtNetwork headers, then QtNetwork is a PRIVATE dependency.

If your source files and your headers include the headers of another library, then it is a PUBLIC dependency.

If your header files other than your source files include the headers of another library, then it is an INTERFACE dependency.

```
.-----------.------------------.----------------.
|           | Linked by target | Link interface |
:-----------+------------------+----------------:
| PUBLIC    |        X         |        X       |
:-----------+------------------+----------------:
| PRIVATE   |        X         |                |
:-----------+------------------+----------------:
| INTERFACE |                  |        X       |
'-----------'------------------'----------------'
```

- **Linked by target**: libraries included in target sources (not a dependency for projects linking the library).
- **Link interface**: libraries included in target public headers (dependencies for projects linking the library).
