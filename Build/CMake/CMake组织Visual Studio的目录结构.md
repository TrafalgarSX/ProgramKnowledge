### Visual Studio目录结构
---
使用CMake本身并不需要管理目录，只需要管理依赖的源文件即可， 像gcc calng这些编译器并不需要一个像Visual Studio这样的完善的IDE。 但是在Windows下，我们在使用Visual Studio工程的时候希望能有目录结构。 实际开发中我们总是将添加的依赖的源文件和头文件添加到工程的指定目录下，像源文件的磁盘上的组织或者放到合适的地方，这样方便进行工程的管理和使用。

如果Windows 下CMake使用MSVC编译器，Visual Studio Generator, CMake会生成Visual Studio工程， ，所以CMake提供了如何在CMakeLists.txt中添加规则生成Visual Studio工程中的目录结构。

通过source_group命令，可以是源码保持原本磁盘上的目录组织方式，也可以自己重新组织。不过一般都是用磁盘上的组织方式， 没有理由自己重新组织。

### 目录组织-source_group()
---
Define a grouping for source files in IDE project generation. There are two different signatures to create source groups.
```cmake
source_group(<name> [FILES <src>...] [REGULAR_EXPRESSION <regex>])
source_group(TREE <root> [PREFIX <prefix>] [FILES <src>...])
```

- **TREE** CMake会自动从`<src>`文件路径中检测，创建对应的source groups，这样可以保证source group的结构与磁盘上文件目录结构一致。 `<src>`的文件路径会被 cut 成对于 `<root>`的相对路径，如果`<src>`中的文件路径不以 `<root>` 路径开头，这个命令会失败。
- **PREFIX**  `<root>`路径中的 source group 和文件会被放进 `<prefix>`的目录中(磁盘上的目录结构被放进去了一个你自定义的目录结构)
- **FILES** 这个列表中的所有文件都被防止进 group `<name>` 中
- **REGULAR_EXPRESSION** Any source file whose name matches the regular expression will be placed in group `<name>`.

The `<name>` of the group and `<prefix>` argument may contain forward slashes or backslashes to specify subgroups. Backslashes need to be escaped appropriately

👉⚠️可以在visual studio中显示目录结构。 但是只有在add_executable之后才能生效（添加到target_sources中和add_executable等效）

从效果上看，官方给出的第一个命令应该适用于自己重新组织目录结构，第二个命令是在Visual Studio中保持和磁盘上一样的目录结构，不过没有测试过第一个命令，并不确定是否正确，不过第二个命令一般就足够了。

### Example
下面的命令会把CMakeLists.txt同级的目录include src在Visual Stduio中显示并放入Serialization目录中。

```cmake
source_group(TREE ${PROJECT_SOURCE_DIR} PREFIX "Serialization" FILES ${HEADERS})
source_group(TREE ${PROJECT_SOURCE_DIR} PREFIX "Serialization" FILES ${EXTRA_SRC} )
```