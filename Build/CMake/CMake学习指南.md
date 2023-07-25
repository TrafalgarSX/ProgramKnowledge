### 学习资料
初学者最应该看的Tutorial（如果不会英文，那就上翻译）：[https://cmake.org/cmake/help/la](https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/guide/tutorial/index.html)

然后，根据我个人的经验，可以再看一下CMake 的包使用（find_package），同样是官方文档：[Using Dependencies Guide — CMake 3.26.3 Documentation](https://cmake.org/cmake/help/latest/guide/using-dependencies/index.html)

<a href="https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html" class=" wrap external" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">cmake-buildsystem这一章</a>

<a href="https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html" class=" wrap external" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">cmake-toolchains 交叉编译可能会用到，以及一些C++包管理会用到，比如Conan</a>

<a href="https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/manual/cmake-packages.7.html" class=" wrap external" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">cmake-packages 如何将你的C++ library打包成一个cmake模块，一般适用于库开发，模块使用则是find_pakcage的使用了</a>

<a href="https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/manual/cmake-policies.7.html" class=" wrap external" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">cmake-policies cmake各个版本间有时会有不兼容的行为，而这个东西则类似于windows系统上可执行文件属性里的兼容性设置，一般容易在开源库的CMakeLists.txt中看到</a>

<a href="https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/8abf754654c4" class=" wrap external" target="_blank" rel="nofollow noreferrer" data-za-detail-view-id="1043">Modern CMake 最佳实践</a>