
### vscode 中如何使用 cmakePresets.json

### 示例
这个可以用于 vscode cmake qt 项目结合， qt项目可以使用 vs的编译器：
但目前这两个配置分别有缺点：
1.  name: vs2019-64-presets 该配置可以使用和 qt 6.7 相对应的编译器 vs2019, 生成 2019 工程， 但是有个缺点就是在 vscode 中使用 visual studio 17 2022 的 generator 无法生成 compile_commands.json 这样就无法在 vscode 中更好的使用 clangd 提供的提示和代码跳转
2. name: vs2022-v64-ninja-presets 可以使用让 qt 在 vscode 中有clangd 的提示和代码跳转， 但是无法使其使用  qt6.7 对应的编译器 vs2019

1. 因为要在vs2022 的 generator 使用 vs2019 的编译器，就需要有 v142 的参数  `cmake -G "Visual Studio 17 2022" -T v142`。
2. 使用 ninja generator 的时候无法使用 -T v142 的参数，也就无法这样去选择编译器， 但是后者又会生成 compile_commands.json，和 vscode 搭配在一起有极佳的开发体验，visual studio 的体验比不上 vscode 

**所以我在探索如何在使用 Ninja generator 的时候，可以自由的使用编译器  vs2019 或者 vs2022。当然，vs2015后  MSVC编译器都有前向兼容性， vs2022 可以使用 vs2019 编译出来的库， 所以使用 vs2022和 qt6.7 来开发工程也没什么问题， 起码目前没有遇到什么问题。**

如果使用 visual studio 就可以既使用 vs2019 的编译器开发 qt6.7, 也有 IDE 的完整体验， 但是我还是更加喜欢 vscode。

#### 我的想法与期望
所以由于强迫症，我还是想在 vscode 摸索 cmakePresets.json， 看是否既能在 vscode 中使用 Ninja generator， 还能使用指定编译器， 比如 vs2019 的编译器， 这样我既能有 vscode 和 clangd 提供的极佳的代码编写，调试体验， 又能使用对应的编译器编译 qt6.7 的工程。

TODO 
如果手动设置对应编译器的路径， 还面临一个问题就是: 对于 使用 cl 编译器来说，不仅仅是设置其编译器路径就可以正常编译， 还要设置相关的环境变量，才能在编译的时候正确的链接 Headers 和 Libraries。

![[Qt+cmake+vscode开发配置#cmake-tools-kits.json 的组成与设置原理]]

```json
{
	"version": 3,
	"configurePresets": [
		{
			"hidden": true,
			"name": "Qt",
			"cacheVariables": {
				"CMAKE_PREFIX_PATH": "$env{QTDIR}"
			}
		},
		{
			"name": "vs2019-64-presets",
			"displayName": "Qt-vs2019-x64",
			"description": "Using compilers for Visual Studio 17 2022 (x64 architecture)",
			"generator": "Visual Studio 17 2022",
			"toolset": "v142",
			"architecture": "x64",
			"binaryDir": "${sourceDir}/out/build/${presetName}",
			"cacheVariables": {
				"CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}",
				"CMAKE_C_COMPILER": "cl.exe",
				"CMAKE_CXX_COMPILER": "cl.exe"
			}
		},
		{
			"name": "vs2022-v64-ninja-presets",
			"displayName": "VS-amd64",
			"description": "Using compilers for Visual Studio 17 2022 (x64 architecture)",
			"generator": "Ninja",
			"binaryDir": "${sourceDir}/out/build/${presetName}",
			"cacheVariables": {
				"CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}",
				"CMAKE_C_COMPILER": "cl.exe",
				"CMAKE_CXX_COMPILER": "cl.exe",
                "CMAKE_EXPORT_COMPILE_COMMANDS": "YES"
			}
		}
	],
	"buildPresets": [
		{
			"name": "vs2022-64-presets-debug",
			"displayName": "Qt-vs2019-x64 - Debug",
			"configurePreset": "vs2022-64-presets",
			"configuration": "Debug"
		},
		{
			"name": "vs2022-v64-ninja-presets-debug",
			"displayName": "VS-amd64 - Debug",
			"configurePreset": "vs2022-v64-ninja-presets",
			"configuration": "Debug"
		}
	]
}
```

### vscode 中使用 cmakePresets.json 时不支持的功能
The following commands are not supported when `CMakePresets.json` integration is enabled:

- **CMake: Quick Start**
- **CMake: Select Variant**
- **CMake: Scan for Kits**
- **CMake: Select a Kit**
- **CMake: Edit User-Local CMake Kits**

### 引用
[cmake-presets(7) — CMake 3.30.1 Documentation](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html#configure-preset)
这个文档里有 cmakePresets.json 各个字段的解释

[vscode-cmake-tools/docs/cmake-presets.md at main · microsoft/vscode-cmake-tools · GitHub](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-presets.md)   这篇说明应该仔细读， 相关的疑问都在这里面

[使用 CMake 预设进行配置和生成 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/build/cmake-presets-vs?view=msvc-170)  微软的文档也很好