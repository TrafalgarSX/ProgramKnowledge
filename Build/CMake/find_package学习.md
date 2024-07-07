### find_package原理
---
首先明确一点，cmake本身不提供任何搜索库的便捷方法，所有搜索库并给变量赋值的操作必须由cmake代码完成，比如下面将要提到的FindXXX.cmake和XXXConfig.cmake。只不过，**库的作者通常会提供这两个文件，以方便使用者调用。**
(并不是， 目前vcpkg 提供 XXXConfig.cmake, cmake 自带一些 FindXXX.cmake)

#### find_package 的两种模式
find_package采用两种模式搜索库：

- **Module模式**：搜索**CMAKE_MODULE_PATH**指定路径下的**FindXXX.cmake**文件，执行该文件从而找到XXX库。其中，具体查找库并给**XXX_INCLUDE_DIRS**和**XXX_LIBRARIES**两个变量赋值的操作由FindXXX.cmake模块完成。
   cmake 自带很多 FindXXX.cmake, 一般都是去这里找
  
- **Config模式**：搜索**XXX_DIR**指定路径下的**XXXConfig.cmake**文件，执行该文件从而找到XXX库。其中具体查找库并给**XXX_INCLUDE_DIRS**和**XXX_LIBRARIES**两个变量赋值的操作由XXXConfig.cmake模块完成。
  👉vcpkg就是这种方式， 不过要先设置 `CMAKE_PREFIX_PATH`, 才能找到对应的 XXXConfig.cmake
  
`find_package`命令在没有明确指定模式的情况下，默认首先尝试**配置模式**（Config mode）。如果在配置模式下未找到指定的包，它会自动回退到**模块模式**（Module mode）。

这意味着，`find_package`首先会寻找由包自身提供的配置文件（如`<PackageName>Config.cmake`），这些文件通常由库的开发者随库一起安装(vcpkg 安装的话，会自带)。如果这些配置文件不存在，**`find_package`会寻找CMake提供或用户自定义的`Find<PackageName>.cmake`模块文件，这些文件包含了寻找库的逻辑。**

如果XXX_DIR路径下找不到XXXConfig.cmake文件，则会找/usr/local/lib/cmake/XXX/中的XXXConfig.cmake文件。

总之，Config模式是一个备选策略。通常，库安装时会拷贝一份XXXConfig.cmake到系统目录中，因此在没有显式指定搜索路径时也可以顺利找到。

❓Windows有吗？  好像没有， 应该就用vcpkg或者connan来管理第三方库比较方便

#### find_package 找 .cmake 的规则

当使用`find_package(<PackageName>)`命令时，CMake在配置模式下查找`<PackageName>Config.cmake`或`<packagename>-config.cmake`文件的过程遵循一定的搜索路径。以下是CMake在配置模式下寻找这些配置文件的一般路径：

1. **CMake变量指定的路径**:
    
    - `CMAKE_PREFIX_PATH`: 包含多个路径，CMake会在这些路径下搜索。(vcpkg 就是先设置这个)
    - `<PackageName>_DIR`: 对于特定包，可以通过设置这个变量直接指定包的配置文件目录。(`<PackageName>_DIR` 这个也可以指定查找)
	
2. **环境变量指定的路径**:
    
    - `CMAKE_PREFIX_PATH`: 与CMake变量同名，但是从环境变量中读取。
    - `<PackageName>_DIR`: 环境变量形式，直接指定配置文件目录。
	
3. **标准安装路径**:
    
    - 在Unix-like系统上，CMake会在如`/usr/local`、`/usr`等标准目录下搜索。
    - 在Windows系统上，CMake会在安装目录下搜索，这些目录可能包括`C:\Program Files\<PackageName>`等。
	
4. **CMake系统变量**:
    
    - `CMAKE_SYSTEM_PREFIX_PATH`: 类似于`CMAKE_PREFIX_PATH`，但是用于系统级的搜索。
	
5. **内置的CMake模块路径**:
    
    - 对于一些常用的库，CMake可能已经内置了寻找这些库的模块，这些模块会在CMake的安装目录下。

CMake会按照上述顺序在这些路径中查找`<PackageName>Config.cmake`或`<packagename>-config.cmake`文件。如果找到了配置文件，CMake会使用该文件中的指令来设置包的相关变量和目标，从而使得包可以在当前项目中被使用。

note: 最重要的 note
*为了确保`find_package`能够成功找到外部包，开发者可以通过设置`CMAKE_PREFIX_PATH`或者特定的`<PackageName>_DIR`变量来指导CMake搜索特定的路径。*

#### openssl 的例子（FindOpenSSL.cmake)

在Windows系统上，CMake的[`FindOpenSSL.cmake`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "c:\Users\guoya\scoop\apps\cmake\3.29.5\share\cmake-3.29\Modules\FindOpenSSL.cmake")模块会在多个可能的位置查找OpenSSL库。这些位置包括但不限于：

1. **环境变量**:
    
    - `OPENSSL_ROOT_DIR`: 如果这个环境变量被设置，[`FindOpenSSL.cmake`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "c:\Users\guoya\scoop\apps\cmake\3.29.5\share\cmake-3.29\Modules\FindOpenSSL.cmake")会首先在这个目录下查找OpenSSL。
    - [`OPENSSL_INCLUDE_DIR`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "CMakeLists.txt")和`OPENSSL_LIBRARY_DIR`: 这些变量可以分别指定包含文件和库文件的位置。
2. **标准安装路径**:
    
    - 在Windows上，OpenSSL可能被安装在`C:\OpenSSL-Win32`或`C:\OpenSSL-Win64`等标准位置，具体取决于OpenSSL的版本和Windows的体系结构（32位或64位）。
    - `Program Files`和`Program Files (x86)`目录下的OpenSSL安装目录也会被检查。
3. **系统路径**:
    
    - [`FindOpenSSL.cmake`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "c:\Users\guoya\scoop\apps\cmake\3.29.5\share\cmake-3.29\Modules\FindOpenSSL.cmake")可能也会检查系统的`PATH`环境变量中列出的目录，尽管这不是查找库的首选方法。
4. **CMake变量**:
    
    - 在调用`find_package(OpenSSL)`时，可以通过设置CMake变量（如通过命令行参数`-DOPENSSL_ROOT_DIR=path/to/openssl`）来指定OpenSSL的安装位置。

[`FindOpenSSL.cmake`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "c:\Users\guoya\scoop\apps\cmake\3.29.5\share\cmake-3.29\Modules\FindOpenSSL.cmake")模块使用这些路径信息来查找OpenSSL的头文件和库文件（如`libssl`和`libcrypto`），并设置相应的CMake变量（如[`OPENSSL_INCLUDE_DIR`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "CMakeLists.txt")、[`OPENSSL_SSL_LIBRARY`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "../../../Users/guoya/scoop/apps/cmake/3.29.5/share/cmake-3.29/Modules/FindOpenSSL.cmake")等），以便在CMake项目中使用OpenSSL。

如果自动查找失败，通常建议手动设置`OPENSSL_ROOT_DIR`环境变量或CMake变量，指向OpenSSL的安装根目录，以帮助[`FindOpenSSL.cmake`](vscode-file://vscode-app/c:/Users/guoya/scoop/apps/vscode/1.91.0/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "c:\Users\guoya\scoop\apps\cmake\3.29.5\share\cmake-3.29\Modules\FindOpenSSL.cmake")正确定位OpenSSL库。

详情请看  FindOpenSSL.cmake （cmake 自带） 其实就是到可能安装的文件夹下面找相关的库， 各种情况都做了判断。

#### other 
👉别人遇到的问题中，由于Caffe安装时没有安装到系统目录，因此无法自动找到CaffeConfig.cmake，在CMakeLists.txt最前面添加了一句话之后就可以了。（也就是用的Config模式)

```cmake
set(Caffe_DIR /home/wjg/projects/caffe/build)   #添加CaffeConfig.cmake的搜索路径

find_package(Caffe REQUIRED)

if (NOT Caffe_FOUND)
    message(FATAL_ERROR "Caffe Not Found!")
endif (NOT Caffe_FOUND)

include_directories(${Caffe_INCLUDE_DIRS})

add_executable(useSSD ssd_detect.cpp)
target_link_libraries(useSSD ${Caffe_LIBRARIES})
```