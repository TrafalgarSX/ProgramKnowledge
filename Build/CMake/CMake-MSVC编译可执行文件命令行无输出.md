### CMake与MSVC合作
#### CMake与vcpkg
设置toolochain file, cmake_prefix_path后可以使用find_package找到vcpkg中安装的第三方库，直接引用这个库的话会依赖动态库， 如果想依赖静态库，目前我没有找到好的设置方法，只能用add_library方式静态引入。
```cmake
  set(CMAKE_TOOLCHAIN_FILE "D:/workspace/dev/vcpkg/scripts/buildsystems/vcpkg.cmake")
  set(CMAKE_PREFIX_PATH "D:/workspace/dev/vcpkg/installed/x64-windows")
  add_library(pthread STATIC IMPORTED)
  set_target_properties(pthread PROPERTIES IMPORTED_LOCATION "${CMAKE_PREFIX_PATH}/lib/pthreadVC3.lib")
  # 静态库
  # add_library(curl STATIC IMPORTED)
  # set_target_properties(curl PROPERTIES IMPORTED_LOCATION "${CMAKE_PREFIX_PATH}/lib/libcurl.lib")
  # 动态库
  find_package(CURL CONFIG REQUIRED)

  list(APPEND EXTRA_LIB CURL::libcurl pthread )
  # 第三方库的头文件（vcpkg安装的）
  list(APPEND EXTRA_INCLUDE "${CMAKE_PREFIX_PATH}/include")

  set_target_properties(gtest_test PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_PREFIX_PATH}/debug/bin")

```

#### CMake如何编译Debug和Release
下面首先设置configuration的类型， 一般设置Release Debug, 后面是对他们的配置
```cmake
if(CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_CONFIGURATION_TYPES Release Debug)
  set(CMAKE_CONFIGURATION_TYPES "${CMAKE_CONFIGURATION_TYPES}" CACHE STRING
        "Reset the configurations to what we need"
        FORCE)
endif()

  # MDd MTd MD MT
  # 设置运行时库的类型
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd /Od /Zi /RTC1 /Gz")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT /O2 /Ob2 /DNDEBUG ")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /machine:x64")

  # 取消CMake默认的Debug和Release目录
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/bin/Debug)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/bin/Release)

  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/lib)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/lib)

# 可执行文件后缀
  set_target_properties(${TARGET_NAME} PROPERTIES DEBUG_POSTFIX "d")
# 库文件后缀
  set(CMAKE_DEBUG_POSTFIX "d")
```

配置好后， 编译时运行这样的命令可以生成对应类型的结果
```
cmake --build . --config Debug
cmake --build . --config Release
```

#### 编译后，运行可执行程序没有输出
原因：
程序运行缺少依赖， 我遇到的情况是运行时找不到动态库。

解决方法：
1. 将动态库放到系统路径下: 64位动态库放到System32中， 32位动态库放到SysWOW64中
```
分别解压至C:\Windows\System32（64位）和C:\Windows\SysWOW64（32位），问题得以解决。

博主测试发现，去掉C:\Windows\SysWOW64（32位）相应dll文件后，在VSCode终端运行可执行文件会导致无输出的问题。
```

2. cmake的一个权宜之计，命令行还是没有输出，但是Visual Studio(Debug)中不会再提示找不到DLL了
```
  set_target_properties(gtest_test PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_PREFIX_PATH}/debug/bin")
```

3. 命令行添加DLL所在路径到环境变量中， 然后运行程序，程序可以从环境变量中找到DLL，就会正常运行有输出。
```powershell
function msvc_run{
  $prefix="D:\workspace\dev\vcpkg\installed\x64-windows"
  $env:PATH+=";${prefix}\bin"
  $env:PATH+=";${prefix}\debug\bin"
}
```

运行程序之前先执行一下msvc_run命令， 然后再执行可执行程序， 就可以正常工作输出了。