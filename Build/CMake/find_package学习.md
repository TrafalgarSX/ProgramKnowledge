### find_package原理
---
首先明确一点，cmake本身不提供任何搜索库的便捷方法，所有搜索库并给变量赋值的操作必须由cmake代码完成，比如下面将要提到的FindXXX.cmake和XXXConfig.cmake。只不过，**库的作者通常会提供这两个文件，以方便使用者调用。**

find_package采用两种模式搜索库：

- **Module模式**：搜索**CMAKE_MODULE_PATH**指定路径下的**FindXXX.cmake**文件，执行该文件从而找到XXX库。其中，具体查找库并给**XXX_INCLUDE_DIRS**和**XXX_LIBRARIES**两个变量赋值的操作由FindXXX.cmake模块完成。
  👉vcpkg就是这种方式， 先在主CMakeLists.txt中设置 **CMAKE_MODULE_PATH**, 然后再去找对应的 **FindXXX.cmake**
  
- **Config模式**：搜索**XXX_DIR**指定路径下的**XXXConfig.cmake**文件，执行该文件从而找到XXX库。其中具体查找库并给**XXX_INCLUDE_DIRS**和**XXX_LIBRARIES**两个变量赋值的操作由XXXConfig.cmake模块完成。


两种模式看起来似乎差不多，**不过cmake默认采取Module模式，如果Module模式未找到库，才会采取Config模式**。

如果XXX_DIR路径下找不到XXXConfig.cmake文件，则会找/usr/local/lib/cmake/XXX/中的XXXConfig.cmake文件。

总之，Config模式是一个备选策略。通常，库安装时会拷贝一份XXXConfig.cmake到系统目录中，因此在没有显式指定搜索路径时也可以顺利找到。

❓Windows有吗？  好像没有， 应该就用vcpkg或者connan来管理第三方库比较方便

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