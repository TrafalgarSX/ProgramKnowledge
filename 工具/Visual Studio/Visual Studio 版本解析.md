
### visual studio 版本

Visual Studio 6 ： vc6
Visual Studio 2003 ： vc7
Visual Studio 2005 ： vc8
Visual Studio 2008 ： vc9
Visual Studio 2010 ： vc10
Visual Studio 2012 ： vc11
Visual Studio 2013 ： vc12
Visual Studio 2015 ： vc14
Visual Studio 2017 ： vc15
Visual Studio 2019 ： vc16
Visual Studio 2022 ： vc17

### vs不同版本对应的Platform Toolset(平台工具集)
**平台工具集**包含 
- C++ 编译器 (cl.exe)
- 链接器 (link.exe)
- C/C++ 标准库

Visual Studio 2015、Visual Studio 2017 和 Visual Studio 2019 是二进制兼容的。 这由工具集的主版本（仍为 14）显示。 在 Visual Studio 2019 或 Visual Studio 2017 中编译的项目与 2017 和 2015 项目 ABI 后向兼容。 **自 Visual Studio 2015 以来，次版本对于每个版本都按 1 更新：**

- Visual Studio 2012：v110
- Visual Studio 2013：v120
- Visual Studio 2015：v140
- Visual Studio 2017：v141
- Visual Studio 2019：v142
- Visual Studio 2022：v143

Visual Studio 还对 C++ 项目支持多定向。 可以使用最新 Visual Studio IDE 编辑和生成使用较旧版本的 Visual Studio 创建的项目。 无需将项目升级为使用新版工具集。 需要在计算机上安装旧版工具集。

#### 更改平台工具集(visual studio IDE 中修改)

1. 在 Visual Studio 中，在菜单栏上选择“项目”>“属性”以打开项目“属性页”对话框。
    
2. 在“属性页”对话框顶部，打开“配置”下拉列表，然后选择“所有配置”。
    
3. 在该对话框中，选择“配置属性”>“常规”属性页。
    
4. 在属性页中，选择“平台工具集”，然后从下拉列表中选择需要的工具集。 例如，如果已安装了 Visual Studio 2010 工具集，请选择“Visual Studio 2010 (v100)”以用于项目。
    
5. 选择“确定”按钮以保存更改。

#### 通过命令行使用 Microsoft C++ 工具集

[通过命令行使用 Microsoft C++ 工具集 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/build/building-on-the-command-line?view=msvc-170)

主要就是运行这个脚本配置环境
```bat

vcvarsall.bat [architecture] [platform_type] [winsdk_version] [-vcvars_ver=vcversion] [spectre_mode]
```

其中 
_`vcversion`_  
（可选）**指定要使用的 Visual Studio 编译器工具集**。 默认情况下，环境设置为使用当前的 Visual Studio 编译器工具集。

_`architecture`_  
此可选参数指定要使用的主机和目标体系结构。 如果未指定体系结构，则使用默认生成环境。 支持以下参数：

|_`architecture`_|编译器|主机计算机体系结构|生成输出（目标）体系结构|
|---|---|---|---|
|`x86`|x86 32 位本机|x86、x64|x86|
|`x86_amd64` 或 `x86_x64`|x86 跨平台上的 x64|x86、x64|x64|
|`x86_arm`|x86 跨平台上的 ARM|x86、x64|ARM|
|`x86_arm64`|x86 跨平台上的 ARM64|x86、x64|ARM64|
|`amd64` 或 `x64`|x64 64 位本机编译器|X64|X64|
|`amd64_x86` 或 `x64_x86`|x64 跨平台上的 x86|x64|x86|
|`amd64_arm` 或 `x64_arm`|x64 跨平台上的 ARM|X64|ARM|
|`amd64_arm64` 或 `x64_arm64`|x64 跨平台上的 ARM64|x64|ARM64|
_`platform_type`_  
可选择使用此自变量指定 **`store`** 或 **`uwp`** 作为平台类型。 默认情况下，环境设置为生成桌面或控制台应用。

_`winsdk_version`_  
（可选）指定要使用的 Windows SDK 的版本。 默认情况下，使用最新安装的 Windows SDK。 若要指定 Windows SDK 版本，可使用完整的 Windows SDK 编号，例如 **`10.0.10240.0`**，或指定 **`8.1`** 以使用 Windows 8.1 SDK。

_`vcversion`_  
（可选）指定要使用的 Visual Studio 编译器工具集。 默认情况下，环境设置为使用当前的 Visual Studio 编译器工具集。

使用“-vcvars_ver=14.2x.yyyyy”指定 Visual Studio 2019 编译器工具集的特定版本。

使用“-vcvars_ver=14.29”指定 Visual Studio 2019 编译器工具集的最新版本。

使用“-vcvars ver=14.0”指定 Visual Studio 2015 编译器工具集。

_`spectre_mode`_  
请保留此参数，以使用没有 Spectre 缓解措施的库。 使用 **`spectre`** 值以使用带有 Spectre 缓解措施的库。


目前还没有尝试在命令行中设置不同的工具集。(DONE)

### ref
[Microsoft Visual C++ compiler versioning (Visual C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/overview/compiler-versions?view=msvc-170)

[Site Unreachable](https://viml.nchc.org.tw/lots-of-version-number-of-visual-studio/)

[Site Unreachable](https://blog.knatten.org/2022/08/26/microsoft-c-versions-explained/)