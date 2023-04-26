### Android命令行调试C/C++程序
传统方式调试 NDK 开发的程序比较麻烦，先要编译成 JNI，又要导出 java接口，还要再写一个 java 工程，改一个地方又要连续改几处，这样效率是很低的。最频繁使用的关键工作路径（编译/调试环节）如果能极致简化，那么可以带来开发效率的成倍提升。其实安卓官方是提供了命令行调试方法的，将你需要调试的 C代码用 NDK直接编译成可执行，然后到设备上执行：

#### 编译
~~使用 NDK 导出独立工具链，方便以后使用，在 cygwin 下面，将 $NDK 环境变量代表的路径设置好，然后：
~~
现在已不推荐导出独立工具链了，直接这样用：
```
 C:\Users\guoya\AppData\Local\Android\Sdk\ndk\25.1.8937393\toolchains\llvm\prebuilt\windows-x86_64\bin\aarch64-linux-android29-clang hello.c -o hello
```

#### 调试
于是你可以在命令行下直接开发 Android 的非 GUI 应用程序了。

调试也很简单，用 adb push 上传到 /data/local/tmp 下面，并且设置可执行模式为 755：
```batch
adb push hello /data/local/tmp/hello  
adb shell chmod 755 /data/local/tmp/hello
```

运行就是直接：
```batch
adb shell /data/local/tmp/hello
```

不可以传到其他目录， 比如/sdcard，这些目录mount时有NOEXEC权限，不能给文件增加可执行权限，而 /data/local/tmp 就是留给大家调试命令行用的，并且不需要 root 权限。


👉TODO 如何使用cmake交叉编译android工程，然后再push到手机上执行。先用libcurlTest工程进行测试。