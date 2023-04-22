### 如何在线安装Qt
```
./qt-unified-windows-x64-4.5.1-online.exe --mirror https://mirrors.aliyun.com/qt
```

### 安装Qt后，如是使用vim编辑并编译相关程序
1. 设置环境变量
```powershell
QT_INCLUDE C:/Qt/6.5.0/mingw_64 
# 最后生成可执行文件执行时，需要此环境变量才能找到依赖的动态库
QT_DLL C:/Qt/6.5.0/mingw_64/bin
```

```cmake
编译使用
set(CMAKE_PREFIX_PATH "C:/Qt/6.5.0/mingw_64") # 依赖的动态库
set(CMAKE_PREFIX_PATH "C:/Qt/6.5.0/mingw_64/lib/cmake") # cmake源文件  find_package使用
```

遇到的问题：
```
使用Ninja进行构造的时候， cmake中设置的cpp编译器不生效，需要放在project之前才生效
但是使用make进行构造的时候不会发生这样的事情。
```
