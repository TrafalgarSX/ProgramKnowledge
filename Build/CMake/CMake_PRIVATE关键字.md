### PRIVATE的特殊之处
---
之前认为`target_compile_options`、`target_link_libraries` 之类的用PRIVATE修饰后会出现所有的依赖属性都不会被传递，实际上并不是。

> When A links in B as PRIVATE, it is saying that A uses B in its
> implementation, but B is not used in any part of A's public API.

> When A links in B as INTERFACE, it is saying that A does not use B in its
> implementation, but B is used in A's public API.

> When A links in B as PUBLIC, it is essentially a combination of PRIVATE and
> INTERFACE. It says that A uses B in its implementation and B is also used in
> A's public API.


Example:
```cmake
target_link_libraries(lib1 PRIVATE lib2)
target_link_libraries(e1 PRIVATE lib1)
```

Q:❓Will e1 necessarily be linked against lib2 when I build e1?

A: Yes.
  But e1 will not have **INCLUDE_DIRECTORIES** nor **COMPILE_DEFINITIONS** that are PUBLIC from lib2.

Static libraries will forward their private dependency link requirements to consumers to ensure that all symbols are provided.

💡经过测试，如果是动态库库的话， 比如el依赖的是lib1的动态库， el就不会链接lib2

>private linking just means that **cmake doesn’t propagate include paths and compiler flags**

>It doesn’t mean that linking to the library itself doesn’t propagate.


邮件解释PRIVATE INTERFACE PUBLIC：

When A links in B as *PRIVATE*, it is saying that A uses B in its implementation, but B is **not used in any part of A's public API**. Any code
that makes calls into A would not need to refer directly to anything from
B. 
   
An example of this could be a networking library A which can be built to
use one of a number of different SSL libraries internally (which B
represents). A presents a unified interface for client code which does not
reference any of the internal SSL data structures or functions. Client code
would have no idea what SSL implementation (B) is being used by A, nor does
that client code need to care.

When A links in B as *INTERFACE*, it is saying that A does **not use B
in its implementation**, but B is **used in A's public API**. Code that calls
into A may need to refer to things from B in order to make such calls. One
example of this is an interface library which simply forwards calls along
to another library but doesn't actually reference the objects on the way
through other than by a pointer or reference. Another example is where A is
defined in CMake as an interface library, meaning it has no actual
implementation itself, it is effectively just a collection of other
libraries (I'm probably over-simplifying here, but you get the picture).

When A links in B as *PUBLIC*, it is essentially a **combination of
PRIVATE and INTERFACE**. It says that A uses B in its implementation and B is
also used in A's public API.

1. Consider first what this means for **include search paths**. 
**If something links against A, it will also need any include search paths from B if B is in A's public API.** Thus, if A links in B either as PUBLIC or INTERFACE, then any header search paths defined for target B will also apply to anything that links to A. Any PRIVATE header search path for B will NOT be carried through to anything that links only to A. The target_include_directories() command handles this. The situation with compile flags is analogously handled with target_compile_definitions() and target_compile_options().

2. Now consider the situation for the **actual libraries involved**. 
**If A is a shared library**, then A will have encoded into it a dependency on B. This information can be inspected with tools like ldd on Linux, otool on Mac and something like Dependency Walker (a.k.a. depends.exe) on Windows. If other
code links directly to A, then it also will have encoded into it a dependency on A. It will not, however, **have a dependency on B unless A links in B as PUBLIC or INTERFACE.** So far, so good. 

**If, however, A is a static library**, the situation changes. Static libraries do not carry information about other libraries they depend on. For this reason, **when A links in B as PRIVATE and another target C links in A, CMake will still add B to the list of libraries to be linked for C because parts of B are needed
by A, but A itself doesn't have that dependency encoded into it.** So even though B is an internal implementation detail of A, C still needs B added
to the linker command, which CMake conveniently handles for you.(CMAKE帮忙处理了这个情况)

3. If you were paying careful attention, you would have noticed that **when A
links in B as PRIVATE, the include directories of B never propagate to
something linking to A**, but if A is a static library, then the *linking* of
B behaves as though the relationship was PUBLIC. 
This PRIVATE-becomes-PUBLIC behaviour for static libraries only applies to the
*linking*, not to the other dependencies (compiler options/flags and
include search paths). The upshot of all this is that if you select
PRIVATE, PUBLIC or INTERFACE based on the explanations in the dot points
above, then CMake will ensure dependencies propagate through to where they
are required, regardless of whether libraries are static or shared. This
does, of course, rely on you the developer not missing any dependencies or
specifying the wrong PRIVATE/PUBLIC/INTERFACE relationship.

As a final note, if you call target_link_libraries() and do not specify any
of PRIVATE, PUBLIC or INTERFACE, you may be tempted to believe that it will
be treated as PUBLIC. The situation is actually more complicated than that
though. It may be treated as PUBLIC or PRIVATE, depending on what other
target_link_library() calls and/or target property manipulations have been
performed. The documentation for target_link_libraries() talks a bit about
this, but you have to go digging into the documentation for the target
properties it mentions to get an understanding of what circumstances lead
to PRIVATE or PUBLIC behaviour.(默认不一定是PUBLIC， 具体为什么， 可以查资料)








官方解释（晦涩难懂, 难以理解）：
  The PUBLIC, PRIVATE and INTERFACE scope keywords can be used to specify both the **link dependencies** and the **link interface** in one command.

  Libraries and targets following **PUBLIC** are linked to, and are made part of the link interface. 
  
  Libraries and targets following **PRIVATE** are linked to, but are not made part of the link interface. 
  
  Libraries following **INTERFACE** are appended to the link interface and are not used for linking ``<target>`.

❓如何理解 link interface?

```cmake
target_link_libraries(<target> <item>...)
```

Library dependencies are **transitive by default** with this signature. When this target is linked into another target then the libraries linked to this target will appear on the link line for the other target too.

This transitive "link interface" is stored in the INTERFACE_LINK_LIBRARIES target property and may be overridden by setting the property directly.
