### CMake配置Visual Studio 属性
---
现在应该有三种方式。
#### `CMAKE_CXX_FLAGS_DEBUG`方式， 
不过这种方式已经不推荐了， 后续会被废弃。
```cmake
   set(CMAKE_C_FLAGS_DEBUG_INIT "/D_DEBUG /MTd /Zi /Ob0 /Od /RTC1")
   set(CMAKE_C_FLAGS_MINSIZEREL_INIT     "/MT /O1 /Ob1 /D NDEBUG")
   set(CMAKE_C_FLAGS_RELEASE_INIT        "/MT /O2 /Ob2 /D NDEBUG")
   set(CMAKE_C_FLAGS_RELWITHDEBINFO_INIT "/MT /Zi /O2 /Ob1 /D NDEBUG")
```

`CMAKE_<LANG>_FLAGS` is a global variable and the most error-prone to use. It also does not support generator expressions, which can come in very handy.

#### `set_property` `set_target_properties`方式
```cmake
set_property(TARGET ${PROJECT_NAME} PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
```
[CMAKE修改VS总结 - wenglabs - 博客园](https://www.cnblogs.com/arxive/p/13341761.html)
这个网站中的表格对应了每个属性如何使用`set_property`方式设置

#### `target_compile_options`方式
❓  官方说visual studio 目前不能很好的与target_compile_options配合使用，但是实际使用第一种方式反而没有生效，好像是使用了target_compile_options时，第一种方式就失效了。 

```cmake
list(APPEND EXTRA_COMPILE_FLAGS "/utf-8;$<IF:$<CONFIG:Debug>,MTd,MT>")
# 编译选项
target_compile_options(${PROJECT_NAME} PUBLIC ${EXTRA_COMPILE_FLAGS})
target_link_options(${PROJECT_NAME} PUBLIC ${EXTRA_LINK_FLAGS})
```

In theory, you could also set the respective properties directly using `set_target_properties`, but `target_compile_options` is usually more readable.

现代CMake应该使用`target_compile_options`的方式，


#### CMake参考
[CMake Reference Documentation — CMake 3.26.4 Documentation](https://cmake.org/cmake/help/latest/index.html#)
[Documentation | CMake](https://cmake.org/documentation/)

