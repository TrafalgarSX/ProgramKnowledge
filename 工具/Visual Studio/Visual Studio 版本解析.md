
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

目前还没有尝试在命令行中设置不同的工具集。

### ref
[Microsoft Visual C++ compiler versioning (Visual C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/overview/compiler-versions?view=msvc-170)

[Site Unreachable](https://viml.nchc.org.tw/lots-of-version-number-of-visual-studio/)

[Site Unreachable](https://blog.knatten.org/2022/08/26/microsoft-c-versions-explained/)