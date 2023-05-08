CMake3.0 之后引入很多新的特性，有效提升了编写构建脚本的效率，称为 Modern CMake。本文总结了在 Modern CMake 使用中的一些最佳实践，供大家参考。

### Target概念
---
旧版 CMake 2.0 主要是基于 directory 来构建，很多复用只能靠变量实现。Modern CMake 最大的改进是引入了 target，支持了对构建的闭包性和传播性的控制  
，从而实现了构建可以模块化。

#### target分类
Target 中最核心的两个分类是：executable, library。

-   其中 executable 是可执行程序，在不同的操作系统会有不同的格式，同样一个工程内也可能需要生成多个可执行程序。 具体指令如下所示：
    
```css
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
              [EXCLUDE_FROM_ALL]
              [source1] [source2 ...])
```

- library 代表链接库，可以分为 share, static, object, module, interface 五个种类。
	
	-   share 表示共享库，在编译构建过程中，需要链接但不会添加到最后的可执行文件中。共享库在程序运行中可以被动态加载和替换，当被多个程序使用时还可以在内存中被共享。如果期望 library 可以被独立的部署和替换的话，需要选择这种方式。
	    
	-   static 表示静态库，会在编译过程中被一起添加生成到可执行文件中。当静态库的实现发生变更时，必须要重新编译整个系统才可以使用。使用静态库的一个好处是，生成的可执行程序可以独立的运行，不再需要依赖这个静态库。
	    
	-   module 也是共享库的一种，CMake 中限制了 moudle 类型的 libray 不能被编译时链接，只能通过 dlopen 在运行时动态加载使用。
	    
	-   object 类型的库表示一组编译后的文件，并不会打包和链接。使用 object 类型的库可以避免一些大的源文件被重复的编译，提升编译效率。
	    
	-   interface 类型的并不会编译输出文件，代表一组接口文件，可以在编译构建中被其他 target 使用。使用 interface 类型的库
	    

#### target闭包性
为了实现 target 闭包性，Modern CMake 实现 target 与 构建和使用中所有依赖建立绑定关系，从而可以拿来即用。正常情况下编译一个 target(可执行程序或者库)需要依赖如下所示：

-   源文件列表，通过 target_sources 配置。
-   头文件列表，通过 target_include_directories 配置。
-   预编译宏，通过 target_compile_definition 配置。
-   编译选项和特性，通过 target_compile_options，target_compile_features 配置。
-   链接选项，通过 target_link_options 配置。

在 C/C++软件系统中，一个 target 中大部分的头文件是仅在模块内使用，为内部接口，仅有小一部分接口头文件是外部使用，称为对外接口。在软件设计过程中，要从高内聚低耦合的角度出发，去严格设计每个 target 的外部接口和内部接口。同样构建过程中，在链接不同 target 时也需要明确指明依赖的外部接口文件，从而提高编译构建的效率。

为了更好支持这个特性，Modern CMake 针对 target 引入两个概念：user requriement(用户依赖) 和 build requirement（编译依赖）。用户依赖表示 target 使用方需要的依赖，而编译依赖表示当前 target 编译构建时需要依赖。

Modern CMake 增加了三个关键字 INTERFACE、PUBLIC、PRIVATE 分布表示不同作用域， 下面以添加头文件依赖命令为例说明：
```xml
target_include_directories(<target> [SYSTEM] [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
  
给 target 添加头文件依赖路径时：

-   **INTERFACE** ： 表示添加的头文件路径仅 target 的使用方需要，编译当前 target 并不需要。
-   **PRIVATE** ： 表示添加的头文件路径仅当前 target 编译时使用，其他 target 不需要。
-   **PUBLIC** ： 表示编译时和链接该 target 都需要使用。

**推荐 2： 在 Modern CMake 中强烈建议为 target 添加依赖接口时，从使用者角度考虑写明 INTERFACE， PRIVATE, PUBLIC。**

**推荐 3： 在 Modern CMake 中推荐使用 target_sources 来添加源文件依赖，保持每个接口的职责单一。**

#### target传播性
当构建工程中 包含比较多的 libary 时，编译和管理这些 Libary 之间的依赖就变得尤为重要。在 Modern CMake 中，当给 Libary 定义用户依赖和编译依赖后，通过在 target_link_libraries 中定义与其他组件间的依赖关系, 就可以自动传递和推演 target 之间的所有编译依赖。

组件间的依赖关系定义命令如下：
  ```xml
target_link_libraries(<target>
                      <PRIVATE|PUBLIC|INTERFACE> <item>...
                     [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

-   **PRIVATE**： 被依赖 libary 的 user requirement 的会变成当前 target 的 build requirement
-   **PUBLIC**：被依赖 libary 的 user requirement 的会变成当前 target 的 build requirement 和 user requirement.
-   **INTERFACE**：被依赖 libary 的 user requirement 的会变成当前 target 的 user requirement

**推荐 4： 充分利用 Modern CMake 强大的依赖传递功能，合理设计每个 target 间的依赖关系。**

Target properties are defined in one of two scopes: **INTERFACE** and **PRIVATE**. Private properties are used _internally_ to build the target, while interface properties are used _externally_ by users of the target. In other words, interface properties model usage requirements, whereas private properties model build requirements of targets.


[It’s Time To Do CMake Right | pablo arias](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)