### 如何使用 visual studio 调试 QML 程序

#### 默认使用 visual studio 打开 cmakelists.txt 目录
如果是使用 visual studio 打开包含 cmakelists.txt 的目录， visual studio 会生成 cmakepresets.json 和 cmakeuserpresets.json， 此时默认就可以使用 visual studio 调试qml 程序， 只要在 qml 文件中打断点即可。

这是因为 cmakeuserpresets.json 中设置了个环境变量是 qml 进程的调试参数之类的， 不太清楚原理

这种方式用的很少， 因为我不喜欢 vs 自动生成的 cmakeUserPresets.json， 还是想自己控制比较好。
### vs code 生成工程， vs 调试
因为我喜欢在 vscode 中进行编码， 但是 vs code 对于 qml 进程的调试支持几乎等于没有（有个插件可以，但是操作起来比较麻烦）， 所以实践上我在 vs code 中使用 cmake 生成 visual studio 工程， 然后打开， 在 vs 中调试。

虽然这样可以使用 vs 进行调试 qml 程序， 但是不是默认就可以的， **需要自己添加可执行程序的命令行参数，才能启动 qml 程序的调试功能。**

打开工程的属性， 在属性窗口的调试界面中添加命令参数：
```bash
-qmljsdebugger=port:12150,block,services:DebugMessages,QmlDebugger,V8Debugger,QmlInspector
或者
-qmljsdebugger=port:12150,block  就可以了
```

或者在生成的VS工程的属性框中的调试页的环境变量设置：
```
QML_DEBUG_ARGS=-qmljsdebugger=file:{B4ACC961-E013-4339-A542-6A167AD1C00E},block
```

以上三种方式都可以在 cmake 生成的 VS工程中开启 QML 工程的调试


可以在使用 cmake 生成 VS工程的时候自动添加对应的设置。

```cmake
    # 设置调试属性
    set_target_properties(${PROJECT_NAME} PROPERTIES
        VS_DEBUGGER_ENVIRONMENT "$<$<CONFIG:Debug>:QML_DEBUG_ARGS=-qmljsdebugger=port:55555,block>"
    )
```

**note**: cmake 生成的 VS工程没有相关的 qt project settings 属性， 不过可以将普通工程转换成 QT 工程， 不过一般不推荐，因为cmake 重新生成后会覆盖修改。

![[Pasted image 20240810042817.png]]
### 使用 qt vs tool 也就是 vs 向导创建 QT工程
这种方式创建的工程是有  qt project settings 属性的

这种方式创建的 qt 工程属性窗口的调试页的环境中会有以下环境变量：
```
QML_DEBUG_ARGS=-qmljsdebugger=file:{B4ACC961-E013-4339-A542-6A167AD1C00E},block
```
这样默认就可以调试 QML 工程。

