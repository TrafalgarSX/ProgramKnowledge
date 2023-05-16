### CMake编译动态库无法输出到指定目录
下面设置输出目录没有生效：
```cmake

set (RelativeWorkPath ${PROJECT_SOURCE_DIR}/../../../libs)
get_filename_component(CMAKE_LIBRARY_PATH "${RelativeWorkPath}" ABSOLUTE)
message("test CMAKE_LIBRARY_PATH = ${CMAKE_LIBRARY_PATH}")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_LIBRARY_PATH}/bin")
# 目标链接库的存放位置 unix: .so
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_PATH}/${ANDROID_ABI})

message("test CMAKE_LIBRARY_OUTPUT_DIRECTORY= ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
# windows: .exe .dll   unix: 可执行文件
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_LIBRARY_PATH}/bin") #
```

原因： `add_library(target ...)`  的target在这些命令之前，就无法正常的将编译的动态库输出到指定目录中， 这些项目上的设置应该尽量放到前面， target相关的放到后面

### 为什么Android Studio无法正常Debug native代码
#### 经典原因
Edit Configuration 中的  Deubgger项没有设置成  java and native

#### 疑难原因，坑我一天了快
依赖 动态库的app模块没有使用正确的编译生成后的动态库， 在上面设置好动态库的输出目录后， 在app模块的 build.gradle中添加以下代码，即可正常Debug动态库。
```
    sourceSets {
        main{
            jniLibs.srcDirs=['C:\\Android\\workplace\\database\\nativelib\\libs']
        }
    }
```
💡注意上面的srcDirs里面不能带有 `arm64-v8a`这样的后缀， android studio会自动根据abiFilters使用对应的动态库, 如果带有后缀，会报错，找不到正确的动态库

sourceSets项 与defaultConfig 同级， 是android项的子项。