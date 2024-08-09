
[Qt+Cmake+Vscode或Visual Studio(开发) - 掘金](https://juejin.cn/post/7163550390879780900)

###  Qt cmake 相关的设置

qt6之后就需要区分 .ui .qrc .cpp 文件了， 直接一起调用 `qt_add_executable` 就会自动做处理
```
qt_add_executable(${PROJECT_NAME} ${PROJECT_SOURCES})
```

qt6 之前有可以通过 cmake 设置
```
        set(CMAKE_AUTOMOC ON)
        set(CMAKE_AUTORCC ON)
        set(CMAKE_AUTOUIC ON)
```

这样也会自动处理 qt 工程中的 .ui .qrc 和 需要 moc 的相关的文件。

也可以不使用设置 `CMAKE_AUTOMOC`,  使用 qt5_wrap_ui 这些宏手动去处理。不过应该只使用其中一种方法。

现在应该都是直接用  `CMAKE_AUTOMOC` 了， keepassxc 就是

[AUTOUIC — CMake 3.30.1 Documentation](https://cmake.org/cmake/help/latest/prop_tgt/AUTOUIC.html#prop_tgt:AUTOUIC)

https://cmake.org/cmake/help/latest/manual/cmake-qt.7.html#autouic
### cmakelist.txt 模板
qt.cmake
```cmake
if(QT_VERSION VERSION_LESS 6.3)
    macro(qt_standard_project_setup)
        set(CMAKE_AUTOMOC ON)
        set(CMAKE_AUTORCC ON)
        set(CMAKE_AUTOUIC ON)
    endmacro()
endif()

if(QT_VERSION VERSION_LESS 6.0)
    macro(qt_add_executable name)
         if(ANDROID)
            add_library(name SHARED ${ARGN})
        else()
            add_executable(${ARGV})
        endif()
    endmacro()
endif()

```


```cmake
cmake_minimum_required(VERSION 3.29)
project(daemon)

include(qt.cmake)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR}
    COMPONENTS
        Core
        Gui
        Widgets
        Xml
)
qt_standard_project_setup()

list(APPEND PROJECT_SOURCES
    ${PROJECT_SOURCE_DIR}/src/main.cpp
)

list(APPEND PROJECT_INCLUDES
    ${PROJECT_SOURCE_DIR}/include
    ${Qt${QT_VERSION_MAJOR}Core_INCLUDE_DIRS}
    ${Qt${QT_VERSION_MAJOR}Gui_INCLUDE_DIRS}
    ${Qt${QT_VERSION_MAJOR}Widgets_INCLUDE_DIRS}
    ${Qt${QT_VERSION_MAJOR}Xml_INCLUDE_DIRS}
)

qt_add_executable(${PROJECT_NAME} ${PROJECT_SOURCES})

target_include_directories(${PROJECT_NAME}
    PUBLIC
        ${PROJECT_INCLUDES}
)

target_link_libraries(${PROJECT_NAME}
    PUBLIC
        Qt::Core
        Qt::Gui
        Qt::Widgets
        Qt::Xml
)
```

### qt 如何使用这些配置

qt6 的项目可以直接使用上面的 CMakeLIsts.txt 模板， 对于编译器的要求，可以在下面一章中按要求设置。

比如， 设置 按照 QT-vs2019-x64 所写， 就可以选择 visual studio 2022  IDE v142编译器，也就是 vs2019 的编译器， 这样就可以与安装的 qt6.7 的依赖编译器统一(虽然说 vs2022 也兼容 vs2019 的编译器，但是最好还是使用一样的编译器)。 

### vscode  cmake-tools-kits.json 的设置
这个配置实际上是 vscode 的 cmake-tool 插件设置的配置， 所以并不通用， 所以如果我想全平台，不依赖于 vscode， 要求仅在 命令行中就可以编译项目， 最好还是使用 cmakePresets.json

```json
  {
    "name": "Qt-vs2019-x64",
    "visualStudio": "013c37f7",
    "visualStudioArchitecture": "x64",
    "isTrusted": true,   
    "preferredGenerator": {
      "name": "Visual Studio 17 2022",
      "platform": "x64",
      "toolset": "v142"
    },
    "cmakeSettings": {
      "CMAKE_TOOLCHAIN_FILE": "C:/workspace/opensource/vcpkg/scripts/buildsystems/vcpkg.cmake",
      "CMAKE_PREFIX_PATH": "$env{QT_DIR}"
    },
    "environmentVariables": {
      "VCPKG_ROOT": "C:/workspace/opensource/vcpkg",
      "QT_DIR": "C:/Qt/6.7.2/msvc2019_64"
    }
  }
```

#### cmake-tools-kits.json 的组成与设置原理
cmake-tools-kits.json:

A _kit_ defines project-agnostic and configuration-agnostic info about how to build code. A kit can include (套件定义了有关如何构建代码的**与项目无关和与配置无关**的信息)

- **A set of compilers**: these are locked at specific versions so that you can switch your compiler version quickly and easily.
- **Visual Studio installation**: building for Visual Studio **involves more than just finding the necessary compiler executable**. Visual C++ **requires certain environment variables** to be set to tell it how to find and link to the Visual C++ toolchain headers and libraries.
##### note:
visual studio 不仅仅需要编译器的路径， 还需要**设置环境变量告诉 cmake 如何找到和链接 visual C++ 的工具链 Headers 和 库**

**Kits are mostly CMake-generator-agnostic** (a CMake generator writes the input files for the native build system). Visual Studio kits have a preferred generator that will be used as a fallback to ensure a matching MSBuild and .sln generator are used for the Visual C++ compiler.

#### 如何在 cmake-tools-kits.json 中设置 visual studio 的编译器
CMake Tools **automatically** sets up the environment for working with Visual C++. It is best to let CMake Tools generate the kits first, **then duplicate and modify them:**

- `visualStudio` : the name of a Visual Studio installation obtained by `VSWhere`.  
- `visualStudioArchitecture`: the Visual Studio target architecture that would be passed to the `vcvarsall.bat` file when entering the VS dev environment.

**Note:** To use Visual C++, both `visualStudio` and `visualStudioArchitecture` must be specified. Omitting either one won't work.

这两个设置都是  **必要的**， 从生成的其他设置里面 copy 过来就行
####  Unsupported commands:
当 cmakePresets.json 在工程中的时候， 是不支持  cmake-tools-kits.json 文件的， 也就是需要自己在 cmakePresets.json 中设置 编译器或者 visual studio 的编译环境什么的， 还有就是windows 下 cmakePresets.json 如果没有指定编译器， 会默认使用 gcc 的编译器 

### 参考
[vscode-cmake-tools/docs/kits.md at main · microsoft/vscode-cmake-tools · GitHub](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/kits.md)

[cmake-presets(7) — CMake 3.30.1 Documentation](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html#configure-preset)