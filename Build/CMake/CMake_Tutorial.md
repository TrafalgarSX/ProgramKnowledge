### Tutorial
---
#### Version Number and Configured Header File
```cmake 
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

config.h.in
```cmake
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

引入头文件的一种方式
```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

#### Specify the C++ Standard
```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

### Step1 Adding a Library
---
```cmake
# 添加库
add_library(MathFunctions mysqrt.cxx)

if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )
```

💡Note the use of the variable `EXTRA_LIBS` to **collect up any optional libraries to later be linked into the executable**. The variable `EXTRA_INCLUDES` is used similarly for optional header files. This is a classic approach when dealing with many optional components, we will cover the modern approach in the next step.

### Step2 Adding Usage Requirements for Library
---
Usage requirements allow for far better control over a library or executable’s link and include line while also **giving more control over the transitive property of targets inside CMake.** The primary commands that leverage usage requirements are:

-   `target_compile_definitions`
    
-   `target_compile_options`
    
-   `target_include_directories`
    
-   `target_link_libraries`

Let’s refactor our code from Adding a Library (Step 2) to **use the modern CMake approach of usage requirements.** We first state that anybody linking to MathFunctions needs to include the current source directory, while MathFunctions itself doesn’t. So this can become an INTERFACE usage requirement.

Remember `INTERFACE` means things that **consumers require but the producer doesn’t.** Add the following lines to the end of MathFunctions/CMakeLists.txt:

```cmake
# 取代前一章的用法，这是更加modern的用法
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )


# 这些都应该被删除掉
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
                           ${EXTRA_INCLUDES}
```

[[CMake_INTERFACE关键字]]

### Step3 Installing and Testing
---
[[CMake基础#安装和测试]]

```cmake
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

Rebuild the application and then cd to the binary directory and run `ctest -N` and `ctest -VV`. For multi-config generators (e.g. Visual Studio), the configuration type must be specified. To run tests in Debug mode, for example, use `ctest -C Debug -VV` from the build directory (not the Debug subdirectory!). Alternatively, build the `RUN_TESTS` target from the IDE.

#### Adding System Introspection
检查平台上是否有某个函数的。
CheckSymbolExists是CMake中的一个宏，用于检查指定的符号是否存在于指定的库中。（有利于编写跨平台的CMake脚本）

```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
```
>set(CMAKE_REQUIRED_LIBRARIES "m") 用于设置CMake的变量CMAKE_REQUIRED_LIBRARIES的值为"m"，其中"m"是一个数学库的名称，代表数学函数库。这个变量会在执行CheckSymbolExists宏时被使用，以便告诉CMake在检查符号存在性时需要链接这个数学库。如果要使用其他库来检查符号存在性，可以将该变量的值设置为相应的库名称。

>check_symbol_exists(log "math.h" HAVE_LOG)
>这是一条CMake的指令，用于检查在指定的头文件"math.h"中是否存在指定的符号"log"，如果存在则将变量HAVE_LOG设置为1，否则设置为0。 
>具体解释如下： 
>- check_symbol_exists：这是一个CMake的宏，用于检查指定的符号是否存在。 
>- log：这是要检查的符号，这里是数学库中的对数函数。 
>- "math.h"：这是要检查的头文件。 - HAVE_LOG：这是要设置的CMake变量，如果符号存在则设置为1，否则设置为0。

设置了变量之后，需要将这个值设置到config.h中（设置c/c++的宏）
```cmake
// does the platform provide exp and log functions?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```
>这是一个CMake的预定义指令，它的作用是定义一个C++宏变量HAVE_LOG，其值与CMake中检查的变量HAVE_LOG相同。 
>在CMake中，使用check_symbol_exists指令检查指定符号是否存在时，会自动定义一个C++宏变量，其名称与检查的变量名称相同，其值为0或1，表示符号是否存在。而cmakedefine指令则可以将这个C++宏变量定义为一个预定义的CMake变量，以便在C++代码中使用。

##### Specify Compile Definition
Is there a better place for us to save the `HAVE_LOG` and `HAVE_EXP` values other than in `TutorialConfig.h`? Let’s try to use `target_compile_definitions`.

```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)

if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
```
>这条命令中的private作用是将编译定义限定为MathFunctions库的私有定义，即这些定义只在MathFunctions库内部可见，不会被其它库或应用程序所使用或影响。这可以保持编译定义的独立性，避免与其它库或应用程序的定义冲突，从而提高代码的可维护性和可移植性。

>如果将PRIVATE修改为PUBLIC，则会将编译定义变为MathFunctions库的公共定义，即这些定义会被MathFunctions库的用户和其它依赖库所可见和使用。这可能会对其它库产生影响，因为它们也可能使用相同的编译定义名称，而这样定义的优先级是不确定的，可能会导致编译错误或不一致的结果。因此，建议将编译定义限定为库的私有定义，以确保其独立性和可移植性。

### Step4 Adding a Custom Command and Generated File
---
In this section, we will create the table as part of the build process, and then compile that table into our application.

In the `MathFunctions` subdirectory, a new source file named `MakeTable.cxx` **has been provided to generate the table.**

```cmake
add_executable(MakeTable MakeTable.cxx)

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )

target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE   ${CMAKE_CURRENT_BINARY_DIR}  # 将生成的Table.h路径加入include
          )
```

### Step5 Packaging an Installer
---
Next suppose that we want to distribute our project to other people so that they can use it. We want to provide both binary and source distributions on a variety of platforms. 

This is a little different from the install we did previously in [`Installing and Testing`](https://cmake.org/cmake/help/v3.25/guide/tutorial/Installing%20and%20Testing.html#guide:tutorial/Installing%20and%20Testing "tutorial/Installing and Testing"), where we were installing the binaries that we had built from the source code. 

In this example we will be building installation packages that support binary installations and package management features. To accomplish this we will use CPack to create platform specific installers. Specifically we need to add a few lines to the bottom of our top-level `CMakeLists.txt` file.

```cmake
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
```

That is all there is to it. We start by including InstallRequiredSystemLibraries. **This module will include any runtime libraries that are needed by the project for the current platform**. Next we set some CPack variables to where we have stored the license and version information for this project. The version information was set earlier in this tutorial and the License.txt has been included in the top-level source directory for this step. The CPACK_SOURCE_GENERATOR variable selects a file format for the source package.

```
CPack是CMake中的一个工具，用于打包构建好的软件成为安装包（如deb、rpm、zip、tar.gz等）。CPack的作用是将构建好的软件打包，以便于在其他机器上进行安装和部署。 

CPack可以根据CMake生成的安装目录和文件列表，自动创建安装包，并提供了许多选项来控制安装包的生成，比如安装包的格式、安装包的名称、版本号、依赖关系等等。 

使用CPack可以使软件的发布和部署过程更加方便和自动化，同时也可以保证软件的一致性和可重复性。
```

To specify the generator, use the [`-G`](https://cmake.org/cmake/help/v3.25/manual/cpack.1.html#cmdoption-cpack-G) option. For multi-config builds, use [`-C`](https://cmake.org/cmake/help/v3.25/manual/cpack.1.html#cmdoption-cpack-C) to specify the configuration. For example:

```cmake
cpack -G ZIP -C Debug
```

To create an archive of the _full_ source tree you would type:

```cmake
cpack --config CPackSourceConfig.cmake
```

Alternatively, run `make package` or right click the `Package` target and `Build Project` from an IDE.

### Step6 Selecting Static or Shared Libraries
---
In this section we will show how the [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/v3.25/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS "BUILD_SHARED_LIBS") variable can be used to **control the default behavior** of [`add_library()`](https://cmake.org/cmake/help/v3.25/command/add_library.html#command:add_library "add_library"), and allow control over how libraries without an explicit type (`STATIC`, `SHARED`, `MODULE` or `OBJECT`) are built.

To accomplish this we need to add [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/v3.25/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS "BUILD_SHARED_LIBS") to the **top-level** `CMakeLists.txt`. We use the [`option()`](https://cmake.org/cmake/help/v3.25/command/option.html#command:option "option") command as it allows users to optionally select if the value should be `ON` or `OFF`.

 linking fails as we are combining a static library without position independent code with a library that has position independent code. The solution to this is to explicitly set the [`POSITION_INDEPENDENT_CODE`](https://cmake.org/cmake/help/v3.25/prop_tgt/POSITION_INDEPENDENT_CODE.html#prop_tgt:POSITION_INDEPENDENT_CODE "POSITION_INDEPENDENT_CODE") target property of SqrtLibrary to be `True` no matter the build type.

💡位置无关代码（Position Independent Code，PIC）的概念。PIC 是指**代码段在内存中的位置与加载时的位置无关，因此可以被加载到任何内存地址而不需要进行重定位**。静态库中的代码段通常是不具有位置无关性的，因此如果将其与具有位置无关代码的库进行链接，可能会导致链接失败。

💡**BUILD_SHARED_LIBS** 是 CMake 自带的变量之一。它是一个布尔类型的选项，用于指定是否使用共享库进行构建。在 CMakeLists.txt 文件中，可以使用该变量来控制生成共享库或静态库。默认情况下，该变量的值为 OFF，即使用静态库进行构建。但是，如果在使用 CMake 时指定了 -DBUILD_SHARED_LIBS=ON，则会使用共享库进行构建。因此，该变量可以通过命令行参数进行设置，也可以在 CMakeLists.txt 文件中进行设置。
>If present and true, this will cause all libraries to be built shared unless the library was explicitly added as a static library. This variable is often added to projects as an [`option()`](https://cmake.org/cmake/help/latest/command/option.html#command:option "option") so that each user of a project can decide if they want to build the project using shared or static libraries.

### Step7 Adding Export Configuration
---
[Step 11: Adding Export Configuration — CMake 3.25.3 Documentation](https://cmake.org/cmake/help/v3.25/guide/tutorial/Adding%20Export%20Configuration.html)

这一章如果只是自己写代码，不发布的话作用不大， 以后可以多练习熟练一下。

这一章的主要目的是 install相关， 会输出很多.cmake的文件到安装文件夹里, 这样可以让其他CMake项目使用本项目。

The next step is to add the necessary information so that other CMake projects can use our project, be it from a build directory, a local install or when packaged.

The first step is to update our [`install(TARGETS)`](https://cmake.org/cmake/help/v3.25/command/install.html#command:install "install") commands to not only specify a `DESTINATION` but also an `EXPORT`. **The `EXPORT` keyword generates a CMake file containing code to import all targets listed in the install command from the installation tree**. So let's go ahead and explicitly `EXPORT` the `MathFunctions` library by updating the `install` command in `MathFunctions/CMakeLists.txt` to look like:
```cmake
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets
        DESTINATION lib)
```

Now that we have `MathFunctions` being exported, we also need to explicitly install the generated `MathFunctionsTargets.cmake` file. This is done by adding the following to the bottom of the top-level `CMakeLists.txt`:
```cmake
install(EXPORT MathFunctionsTargets  # 将生成的.cmake安装到lib文件夹下
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```

导出与当前机器无关的路径
```cmake
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
```

At this point, we have CMake properly packaging the target information that is required but we will still need to generate a `MathFunctionsConfig.cmake` so that the CMake [`find_package()`](https://cmake.org/cmake/help/v3.25/command/find_package.html#command:find_package "find_package") command can **find our project**. So let's go ahead and add a new file to the top-level of the project called `Config.cmake.in` with the following contents:
```config.cmake.in
@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```

正确安装上面生成的文件：
```cmake
include(CMakePackageConfigHelpers)
```

Next, we execute the [`configure_package_config_file()`](https://cmake.org/cmake/help/v3.25/module/CMakePackageConfigHelpers.html#command:configure_package_config_file "configure_package_config_file"). This command will configure a provided file but with a few specific differences from the standard [`configure_file()`](https://cmake.org/cmake/help/v3.25/command/configure_file.html#command:configure_file "configure_file") way. To properly utilize this function, the input file should have a single line with the text `@PACKAGE_INIT@` in addition to the content that is desired. That variable will be replaced with a block of code which turns set values into relative paths. These values which are new can be referenced by the same name but prepended with a `PACKAGE_` prefix.
```
include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )

export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```

### Step8 Packaging Debug and Release
---
By default, CMake's model is that a build directory only contains a single configuration, be it Debug, Release, MinSizeRel, or RelWithDebInfo. It is possible, however, to setup CPack to bundle multiple build directories and construct a package that contains multiple configurations of the same project.

First, we want to ensure that the debug and release builds use different names for the libraries that will be installed. Let's use d as the postfix for the debug libraries.

Set [`CMAKE_DEBUG_POSTFIX`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_DEBUG_POSTFIX.html#variable:CMAKE_DEBUG_POSTFIX "CMAKE_DEBUG_POSTFIX") near the beginning of the top-level `CMakeLists.txt` file:
And the [`DEBUG_POSTFIX`](https://cmake.org/cmake/help/v3.25/prop_tgt/DEBUG_POSTFIX.html#prop_tgt:DEBUG_POSTFIX "DEBUG_POSTFIX") property on the tutorial executable:
```cmake
set(CMAKE_DEBUG_POSTFIX d)

add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
```

add version numbering to the `MathFunctions` library.
```cmake
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
```

We can use [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE "CMAKE_BUILD_TYPE") to set the configuration type:
```cmake
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```