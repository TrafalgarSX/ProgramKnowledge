### 

下面两个链接解释了这三个的不同，知乎链接解释的很清楚

[CMake的链接选项：PRIVATE，INTERFACE，PUBLIC - 知乎](https://zhuanlan.zhihu.com/p/493493849)

[CMake target\_link\_libraries Interface Dependencies - Stack Overflow](https://stackoverflow.com/questions/26037954/cmake-target-link-libraries-interface-dependencies)

官方文档的相关知识的解释。
[cmake-buildsystem(7) — CMake 3.15.7 Documentation](https://cmake.org/cmake/help/v3.15/manual/cmake-buildsystem.7.html#transitive-usage-requirements)
这三者改变了什么，为什么要用不同的属性
Using PUBLIC does add your dependencies to projects linking to your library. Using PRIVATE does NOT add your dependencies to projects linking to your library. **It is cleaner and it also avoids possible conflicts between your dependencies and your user's**