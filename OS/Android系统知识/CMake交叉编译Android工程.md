### 使用CMake交叉编译Android工程
```bash
cmake .. \
  -DCMAKE_SYSTEM_NAME=Android \
  -DCMAKE_SYSTEM_VERSION=29 \
  -DCMAKE_ANDROID_ARCH_ABI=arm64-v8a \
  -DCMAKE_ANDROID_NDK=$env:NDK \
  -DCMAKE_ANDROID_STL_TYPE=gnustl_static

```
使用以上命令进行编译的时候并不能运行， Android官方CMake教程使用以下命令(linux)
```bash
$ cmake \
    -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake \
    -DANDROID_ABI=$ABI \
    -DANDROID_PLATFORM=android-$MINSDKVERSION \
    $OTHER_ARGS
```

windows下也类似， 不同之处在于，powershell的换行是 反引号， 环境变量的使用方式也不同， 对于字符串类型的变量要用 引号 "" 才能使用字符串。

#### Windows下使用脚本交叉编译Android工程
完整脚本如下：
```powershell
# $env:CMAKE_MAKE_PROGRAM = ""
# $env:CMAKE_GENERATOR = ""

# 定义参数, 有默认值
param{
  [string]$api_level = "29",
  [string]$abi = "arm64-v8a",
  [string]$stl = "c++_static",
  [string]$build_dir = "build_android",
}

# 简化命令行参数
$api_level = "android-" + "$api_level"

if($abi -eq "arm64-v8a"){
  $build_dir = "$build_dir" + "_arm64"
}else {
  $build_dir = "$build_dir" + "_arm32"
}

echo "--------arguments start-----------"
write-output $api_level
write-output $abi
write-output $stl
write-output $build_dir
echo "--------arguments end-----------"

# 定义变量
$android_ndk_path = $env:NDK # 替换为你的 Android NDK 路径
$android_ndk_path  = $android_ndk_path.Replace("\", "/")
$android_toolchain_file = "$android_ndk_path" + "/build/cmake/android.toolchain.cmake"

# $source_dir = "source"  # 替换为你的源码目录

# 创建构建目录
New-Item -ItemType Directory -Force -Path $build_dir | Out-Null

$android_toolchain_file

# 运行 CMake 命令, 创建一个build目录(与top level CmakeLists.txt同级)
# 在build目录下有不同的构建系统, 如: arm64-v8a, armeabi-v7a, Ninja, make
cmake -S .. -B "$build_dir" -DCMAKE_TOOLCHAIN_FILE="$android_toolchain_file" `
  -DCMAKE_SYSTEM_NAME=Android `
  -DANDROID_NATIVE_API_LEVEL="$api_level" `
  -DANDROID_ABI="$abi" `
  -DANDROID_STL="$stl" `
  -G "Ninja" 


#  CMAKE_HOST_SYSTEM_NAME  Windows
# 编译程序
# cmake --build .

# 编译程序和动态库
# cmake --build $build_dir --target my_program --config Release
#
```

#### Android中使用CMake教程
工具链参数：
以下参数可以传递给 CMake 工具链文件。如果使用 Gradle 进行构建，请按照 [ExternalNativeBuild 文档](https://developer.android.com/reference/tools/gradle-api/7.1/com/android/build/api/dsl/ExternalNativeBuildOptions?hl=zh-cn)中所述向 `android.defaultConfig.externalNativeBuild.cmake.arguments` 添加参数。如果通过命令行进行构建，请使用 `-D` 将参数传递给 CMake。例如，要强制 armeabi-v7a 始终使用 Neon 支持进行构建，请传递 `-DANDROID_ARM_NEON=TRUE`。

💡**注意**：任何必需参数均由 Gradle 自动传递，只有在通过命令行构建时才需明确传递。
💡`ANDROID_ABI`是必需参数

| Value                 | 备注                                                        |
| --------------------- | ----------------------------------------------------------- |
| armeabi-v7a           |                                                             |
| armeabi-v7a with NEON | 与 `-DANDROID_ABI=armeabi-v7a -DANDROID_ARM_NEON=ON` 相同。 |
| arm64-v8a             |                                                             |
| x86                   |                                                             |
| x86_64                |                                                             |

- `ANDROID_ARM_NEON`
为 armeabi-v7a 启用或停用 NEON。对其他 ABI 没有影响。对于 API 级别（`minSdkVersion` 或 `ANDROID_PLATFORM`）23 或更高版本，默认为 true，否则为 false。

| Value | 备注 |
| ----- | ---- |
| TRUE  |API 级别 23 或更高版本的默认值。      |
| FALSE      |API 级别 22 或更低版本的默认值。      |

- `ANDROID_LD`
选择要使用的链接器。[lld](https://lld.llvm.org) 目前处于 NDK 实验阶段，可通过此参数启用。

- `ANDROID_NATIVE_API_LEVEL`
[ANDROID_PLATFORM](https://developer.android.com/ndk/guides/cmake?hl=zh-cn#android_platform) 的别名。

- `ANDROID_PLATFORM`
指定应用或库所支持的最低 API 级别。此值对应于应用的 `minSdkVersion`。
此参数支持多种格式：
	-  `android-$API_LEVEL`
	-  `$API_LEVEL`
	-  `android-$API_LETTER`
`$API_LETTER` 格式允许您指定 `android-N`，而无需确定与该版本关联的编号。请注意，对于某些版本，API 只是编号增加了，但字母次序没有增加。您可以通过附加 `-MR1` 后缀来指定这些 API。例如，API 级别 25 为 `android-N-MR1`。

- `ANDROID_STL`
指定要为此应用使用的 STL。如需了解详情，请参阅 [C++ 库支持](https://developer.android.com/ndk/guides/cpp-support?hl=zh-cn)。默认情况下将使用 `c++_static`。

#### 了解CMake构建命令
Android Gradle 插件会将用于为每对 ABI 和[构建类型](https://developer.android.com/studio/build/build-variants?hl=zh-cn)执行 CMake 构建的构建参数保存至 `build_command.txt`。这些文件位于以下目录中：
```
<project-root>/<module-root>/.cxx/cmake/<build-type>/<ABI>/
```

Android Studio 使用Cmake编译时的参数
```
                    Executable : ${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/cmake
arguments :
-H${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/src/main/cpp
-DCMAKE_FIND_ROOT_PATH=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/.cxx/cmake/universalDebug/prefab/armeabi-v7a/prefab
-DCMAKE_BUILD_TYPE=Debug
-DCMAKE_TOOLCHAIN_FILE=${HOME}/Android/Sdk/ndk/22.1.7171670/build/cmake/android.toolchain.cmake
-DANDROID_ABI=armeabi-v7a
-DANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670
-DANDROID_PLATFORM=android-23
-DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a
-DCMAKE_ANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670
-DCMAKE_EXPORT_COMPILE_COMMANDS=ON
-DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_RUNTIME_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_MAKE_PROGRAM=${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/ninja
-DCMAKE_SYSTEM_NAME=Android
-DCMAKE_SYSTEM_VERSION=23
-B${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/.cxx/cmake/universalDebug/armeabi-v7a
-GNinja
jvmArgs :

                    Build command args: []
                    Version: 1

```
