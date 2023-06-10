### 今天遇见了一个vcpkg的大坑
---
今天想测试一下使用libcurl静态库，之前不知道为什么一直用不起来libcurl的静态库， 今天编译了libcurl的静态库版本，发现使用libcurl的静态库版本需要一些额外的库和宏定义

```cmake 
# libcurl的静态库还依赖ws2_32 wldap32 crypt32， 否则无法使用，会有无法解析的符号
list(APPEND EXTRA_LIB my_zlib  libcurl ws2_32 wldap32 crypt32)
# 使用libcurl静态库还需要额外的宏 CURL_STATICLIB
list(APPEND EXTRA_DEFINES "ZLIB_WINAPI;CURL_STATICLIB;_UNICODE")
```

原本这样再引入zlib的静态库就可以了， 但是从下午3点开始到11点都无法找到为什么引入了zlib静态库，但是仍然找不到zlib相关的符号(无法解析的符号)

```powershell
libcurlTest.obj : error LNK2019: unresolved external symbol __imp_deflate referenced in function main [D:\workspace\cprojects\libcurlTest\msvcbuild\curl_test.vcxproj]
libcurlTest.obj : error LNK2019: unresolved external symbol __imp_deflateEnd referenced in function main [D:\workspace\cprojects\libcurlTest\msvcbuild\curl_test.vcxproj]
libcurlTest.obj : error LNK2019: unresolved external symbol __imp_deflateInit_ referenced in function main [D:\workspace\cprojects\libcurlTest\msvcbuild\curl_test.vcxproj]
D:\workspace\cprojects\libcurlTest\msvcbuild\AMD64\Release\bin\curl_test.exe : fatal error LNK1120: 3 unresolved externals [D:\workspace\cprojects\libcurlTest\msvcbuild\curl_test.vcxproj]
```

这里注意：  `__imp_deflate` 有 `__imp_` 前缀的符号找不到， 这是动态库的符号， 不是静态库的符号，出现这样的符号说明函数用  `__declspec(dllimport)` 修饰了的， 但是静态库是不会有这样的修饰的，只有动态库的函数声明中会有。

❓那我明明是想链接静态库，为什么函数声明出现了`__declspec(dllimport)`呢？

问题出现在头文件上： 头文件根据宏定义， 定义了`__declspec(dllimport)`， 在每个函数声明前加了这个声明 

#### 大坑

```powershell
./vcpkg install zlib:x64-windows
```

这条命令默认安装的是动态库版本， 在路径  `path/vcpkg/installed/x64-windows` 下的lib和include都是针对动态库的，所以会出现上面我无法找到 `__imp_` 前缀的符号， 因为这里面的头文件不是原版的头文件， 都是针对动态库的。e.g.

```
#ifdef ZLIB_DLL

#endif
```

正常情况下， 原版的头文件是这样的， 但是如果是vcpkg安装的动态库版本， 那么对应路径下的头文件就会是
```c
#ifdef 1

#endif
```

它会写死😅😅😅, 妈的， 就这个坑了我将近一天。

你如果安装的是静态库版本(x64-windows-static)：

```c
#ifdef 0

#endif
```

这样libcurl去找依赖的zlib的头文件时，因为我设置的环境变量是动态库的路径在前， 所以它找到的是动态库版本的头文件， 这样在我链接静态库版本时，它就会找不到 `__imp_`的符号。
