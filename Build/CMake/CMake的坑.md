### target_link_directories
---
❓CMake生成的visual studio 工程中用 target_link_directories 设置的路径后总有${Configuration}, 按理说路径使用生成表达式生成的， 不应该再有 ${Configuration}了， 不知道为什么。

stackoverflow 上与我相同的疑惑：
[xcode - CMake target\_link\_directories is adding extra unexpected directories. How can I remove them? - Stack Overflow](https://stackoverflow.com/questions/76574802/cmake-target-link-directories-is-adding-extra-unexpected-directories-how-can-i)

At the time of this writing (when CMake 3.27 is the latest version), **for the Visual Studio generator**, it looks like this is **hardcoded** in the CMake source code without any way to disable it.

In [cmVisualStudio10TargetGenerator.cxx](https://gitlab.kitware.com/cmake/cmake/-/blob/master/Source/cmVisualStudio10TargetGenerator.cxx)'s `cmVisualStudio10TargetGenerator::ComputeLinkOptions`
```cpp
  std::vector<std::string> const& ldirs = cli.GetDirectories();
  std::vector<std::string> linkDirs;
  for (std::string const& d : ldirs) {
    // first just full path
    linkDirs.push_back(d);
    // next path with configuration type Debug, Release, etc
    linkDirs.push_back(d + "/$(Configuration)");
  }
```

You could either [build your own modified version of CMake](https://gitlab.kitware.com/cmake/cmake/-/blob/master/README.rst), or [ask the maintainers to add a feature](https://gitlab.kitware.com/cmake/cmake/-/issues/new) to toggle this behaviour.

For the Xcode generator, it looks like you can disable this by changing [policy CMP0142](https://cmake.org/cmake/help/latest/policy/CMP0142.html) (`cmake_policy(SET CMP0142 NEW)`). Quoting the docs:

> _New in version 3.25_.
> 
> The [Xcode](https://cmake.org/cmake/help/latest/generator/Xcode.html#generator:Xcode) generator does **not append per-config suffixes** to library search paths.
> 
> In CMake 3.24 and below, the Xcode generator preceded each entry of a library search path with a copy of itself appended with `$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)`. This was left from very early versions of CMake in which per-config directories were not well modeled. Such paths often do not exist, resulting in warnings from the toolchain. CMake 3.25 and above prefer to not add such library search paths. This policy provides compatibility for projects that may have been accidentally relying on the old behavior.
> 
> The `OLD` behavior for this policy is to append `$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)` to all library search paths. The `NEW` behavior is to not modify library search paths.
> 
> This policy was introduced in CMake version 3.25. Use the [`cmake_policy()`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command to set it to `OLD` or `NEW` explicitly. Unlike many policies, CMake version 3.27.0-rc3 does not warn when this policy is not set and simply uses `OLD` behavior.


You'd need to ask the maintainers why exactly this stuff is the way it is (at least from a historical perspective). See [https://discourse.cmake.org/](https://discourse.cmake.org/). If you do ask, please comment here with a link to the Discourse page.