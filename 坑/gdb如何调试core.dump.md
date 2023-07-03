### 可执行程序生成的coredump
执行命令：
```bash
gdb test  core.1234
```

### jni调用的C/C++动态库生成的coredump
执行命令：
```bash
gdb java core.1234
```

### 如何查看相关信息
```
bt # 显示崩溃时的调用栈
frame ${number}  # 转到对应的调用栈
info locals # 需要使用Debug编译  -g参数
p  ${some var}
```
