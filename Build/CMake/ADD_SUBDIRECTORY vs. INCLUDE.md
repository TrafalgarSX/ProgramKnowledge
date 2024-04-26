对于简单的项目，将所有内容保存在一个目录中是可以的，**但是大多数实际项目倾向于将它们的文件分割到多个目录中**。

通常可以找到不同的文件类型或分组在各自的目录下的独立模块，或者将属于逻辑功能组的文件放在项目目录层次结构的各自部分中。

在任何多目录项目中，两个基本的CMake命令是**add_subdirectory()** 和 **include()**。

这些命令将来自另一个文件或目录的内容引入到构建中，允许构建逻辑分布在目录层次结构中，而不是强制所有内容都在最顶层定义。这样做有很多好处:

- 构建逻辑是本地化的，这意味着构建的特征可以在它们最相关的目录中定义。
- **构建可以由子组件组成**，子组件的定义独立于使用它们的顶级项目。这对于使用git子模块或嵌入第三方源代码树的项目来说尤为重要。
- 因为目录可以是自包含的，所以仅仅通过选择是否在该目录中添加就可以打开或关闭构建的部分。

### add_subdirectory()
---
add_subdirectory()命令允许项目将另一个目录带入构建。**该目录必须有自己的CMakeLists.txt文件**，该文件将在add_subdirectory()被调用的地方进行处理，**并在项目的构建树中为它创建一个相应的目录。**

```cmake
add_subdirectory(sourceDir [ binaryDir ] [ EXCLUDE_FROM_ALL ])
```

**sourceDir不一定是源树中的子目录，尽管它通常是**。可以添加任何目录，sourceDir可以指定为**绝对路径或相对路径**，后者相对于当前源目录。绝对路径通常只在添加主源代码树之外的目录时才需要。

通常，binaryDir不需要指定。省略时，CMake会在构建树中创建一个与sourceDir同名的目录。如果sourceDir包含任何路径组件，它们将被镜像到CMake创建的binaryDir中。或者，binaryDir可以显式地指定为绝对路径或相对路径，后者相对于当前二进制目录(稍后将更详细地讨论)求值。**如果sourceDir是源树之外的一个路径，CMake需要指定binaryDir，因为相应的相对路径不能再被自动构造。**

可选的 **EXCLUDE_FROM_ALL** 关键字用于控制在添加的子目录中定义的目标在默认情况下是否应该包含在项目的ALL目标中。不幸的是，对于一些CMake版本和项目生成器，它并不总是像预期的那样工作，甚至会导致构建破裂。

(1)Source和Binary目录变量

有时，开发人员需要知道与当前源目录对应的构建目录的位置，例如当在运行时需要复制文件或者执行自定义构建任务时。使用add_subdirectory()，源代码树和构建树的目录结构可以任意复杂。甚至可以在同一个源代码树中使用多个构建树。因此，开发人员需要CMake的帮助来确定感兴趣的目录。为此，CMake提供了许多变量来跟踪当前正在处理的CMakeLists.txt文件的源目录和二进制目录。当CMake处理每个文件时，以下只读变量会自动更新。**它们总是包含绝对路径。**

- CMAKE_SOURCE_DIR
    
    源码树的最顶层目录(也就是CMakeLists.txt文件所在的地方)。这个变量永远不会改变它的值。
    
- CMAKE_BINARY_DIR
    
    **构建树的最顶层目录**。这个变量永远不会改变它的值。
    
- CMAKE_CURRENT_SOURCE_DIR
	    
	**CMake正在处理的CMakeLists.txt文件所在的目录**。每次在add_subdirectory()调用的结果中处理新文件时，它都会更新，并在完成对该目录的处理后再次恢复。

- CMAKE_CURRENT_BINARY_DIR
	
	**当前CMake正在处理的CMakeLists.txt文件对应的构建目录**。每次调用add_subdirectory()时它都会改变，并在add_subdirectory()返回时再次恢复。
 
(2)范围

调用add_subdirectory()的效果之一是，**CMake为处理该目录的CMakeLists.txt文件创建了一个新的作用域**。这个新的作用域就像调用作用域的子作用域，有很多效果:

- **调用作用域中定义的所有变量对子作用域都是可见的**，子作用域可以像读取其他变量一样读取它们的值。 (子作用域可以看见父作用域的变量)

- 在子作用域中创建的任何新变量对**调用作用域（父作用域）都不可见**。

- 对子作用域中的变量的任何更改都是该子作用域中的局部变量。即使该变量存在于调用作用域中，调用作用域的变量也保持不变。**在子作用域中修改的变量就像一个新变量，在处理离开子作用域中时丢弃该变量。**

换句话说，在进入子作用域时，**它会接收到那个时间点上在调用作用域中定义的所有变量的副本**。对子变量的任何更改都将在子变量的副本上执行，而不会改变调用者的变量。

但是，**有时候，希望在添加的目录中对变量进行的更改对调用者是可见的**。例如，该目录可能负责收集一组源文件名，并将其作为文件列表向上传递给父目录。这就是set()命令中**PARENT_SCOPE**关键字的作用。当使用PARENT_SCOPE时，所设置的变量是父作用域中的变量，而不是当前作用域中的变量。重要的是，这并不意味着同时在父范围和当前范围中设置变量。

受范围影响的不仅仅是变量，**策略和一些属性在这方面也与变量有类似的行为**。对于策略，每个add_subdirectory()调用都会创建一个新的范围，在此范围内可以进行策略更改，而不会影响父策略的设置。类似地，可以在子目录的CMakeLists.txt文件中设置目录属性，这对父目录的目录属性没有影响。

### include()
---
CMake提供的另一个从其他目录中获取内容的方法是include()命令，它有以下两种形式:
```cmake
include(fileName [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
include(module [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
```

第一种形式有点类似于add_subdirectory()，但有一些重要的区别:

- **include()需要读取文件的名称，而add_subdirectory()需要一个目录，并在该目录中查找CMakeLists.txt文件**。传递给include()的文件名通常**扩展名为.cmake，但可以是任何名称**。

- include()**没有引入新的变量范围**，而add_subdirectory()引入了。

- 默认情况下，**这两个命令都引入了一个新的策略范围，但是可以使用NO_POLICY_SCOPE选项告诉include()命令不要这样做(add_subdirectory()没有这样的选项)。**

- CMAKE_CURRENT_SOURCE_DIR 和 CMAKE_CURRENT_BINARY_DIR 变量的值在处理由include()命名的文件时不会改变，然而它们在add_subdirectory()中会改变。

本质上像 c/c++ 的 `#include`

include()命令的第二种形式具有完全不同的目的。它用于加载命名的模块，除了第一点之外，上述所有观点都适用于第二种形式。

因为当include()被调用时，CMAKE_CURRENT_SOURCE_DIR 的值不会改变，所以包含的文件似乎很难计算出它所在的目录。CMAKE_CURRENT_SOURCE_DIR 将包含调用include()的文件的位置，而不是包含包含文件的目录。此外，与文件名总是为CMakeLists.txt的add_subdirectory()不同，当使用include()时，**文件的名称可以是任何东西，所以所包含的文件很难确定自己的名称**。为了解决这样的情况，CMake提供了一组额外的变量:

- CMAKE_CURRENT_LIST_DIR

类似于CMAKE_CURRENT_SOURCE_DIR，除了**它会在处理包含的文件时更新**。这是在需要处理当前文件的目录时使用的变量，不管它是如何被添加到构建中的。它总是保持一条绝对路径。

- CMAKE_CURRENT_LIST_FILE

**总是给出当前正在处理的文件的名称**。它总是保存文件的绝对路径，而不仅仅是文件名。

- CMAKE_CURRENT_LIST_LINE

保存当前正在处理的文件的行号。这个变量很少需要，但是在一些调试场景中可能会被证明是有用的。

- 需要注意的是，上述三个变量适用于任何CMake处理的文件，而不仅仅是那些include()命令的文件。即使是通过add_subdirectory()拉入的CMakeLists.txt文件，它们的值也与上面描述的相同，在这种情况下，CMAKE_CURRENT_LIST_DIR将与CMAKE_CURRENT_SOURCE_DIR具有相同的值。

### 早期终止处理
---
在某些情况下，项目可能希望停止处理当前文件的剩余部分，并将控制权返回给调用者。return()命令可以完全用于此目的，但请注意，它不能向调用者返回值。它的唯一作用是结束当前作用域的处理。如果不是在函数内部调用，return()将结束对当前文件的处理，无论它是通过include()还是add_subdirectory()引入的。

如前一节所述，项目的不同部分可能包括来自多个位置的相同文件。有时，最好检查这个文件，只包含该文件一次，并尽早返回后续包含的内容，以防止多次重新处理该文件。这与C/ C++头文件的情况非常相似，通常会看到类似形式的include guard被使用:

```scss
if(DEFINED cool_stuff_include_guard)
  return()
endif()
set(cool_stuff_include_guard 1)
# ...
```

在CMake 3.10或更高版本中，可以使用类似于C/ c++中的#pragma once的专用命令来更简洁、更健壮地表达这一点:

```scss
include_guard()
```

与手动编写if-endif代码相比，这更健壮，因为它在内部处理保护变量的名称。该命令还接受一个可选的关键字参数DIRECTORY或GLOBAL，以指定一个不同的范围，在这个范围内检查以前处理过的文件，但在大多数情况下不太可能需要这些关键字。如果没有指定任何参数，则假定变量的作用域，其效果与上面的if-endif代码完全相同。GLOBAL确保如果该文件在项目中的其他地方之前被处理过，则该命令将结束对该文件的处理(即忽略变量作用域)。DIRECTORY仅在当前目录范围内及以下检查以前的处理。

[CMake(六)：使用子目录 - DoubleLi - 博客园](https://www.cnblogs.com/lidabo/p/16661100.html)

### 不同之处
They are very different.

Think of INCLUDE as an `#include` in C or C++. It is useful when you  
have defined custom commands, custom targets, all your CMakeLists.txt  
have parts in common, etc and you want to write them only once.  
**INCLUDE helps you to re-use CMake "code".** 

For instance, I have an arisnova.cmake file where I have the following custom targets and commands: 
GENERATE_DOCUMENTATION(Doxyfile), 
GENERATE_DISTCLEAN_TARGET() and 
GENERATE_UNINSTALL_TARGET(). 

As I want to use these commands in all my projects, I have an  
INCLUDE(arisnova.cmake) in the "root" CMakeLists.txt of every project.

**You would use ADD_SUBDIRECTORY and its sibling SUBDIRS when your  
project has multiple directories.** 

For instance, imagine you have these  
source tree:
```markdown
alarmclock
      |
      \--- libwakeup
      \--- clock
```

Usually, you'd have three CMakeLists.txt here: one in the 'alarmclock'  
directory (the "root" CMakeLists.txt), one in the 'libwakeup'  
directory and one in the 'clock' directory. For instance:

* alarmclock/CMakeLists.txt
```cmake
PROJECT( alarmclock )
ADD_SUBDIRECTORY( libwakeup )
ADD_SUBDIRECTORY( clock )
```

* alarmclock/libwakeup/CMakeLists.txt
```cmake
PROJECT( libwakeup )
SET( wakeup_SRCS wakeup.cpp )
ADD_LIBRARY( wakeup SHARED ${wakeup_SRCS} )
```

* alarmclock/clock/CMakeLists.txt
```cmake
PROJECT( clock )
SET( clock_SRCS clock.cpp )
ADD_EXECUTABLE( clock ${clock_SRCS} )
TARGET_LINK_LIBRARIES( clock wakeup )
```

You can of course combine INCLUDE and ADD_SUBDIRECTORY/SUBDIRS in the  
same CMakeLists.txt, as they have very different purposes.