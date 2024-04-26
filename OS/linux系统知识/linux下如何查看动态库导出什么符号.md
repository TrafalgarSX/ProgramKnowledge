
```shell
readelf -a libpthread.so.0 | grep pthread_init

nm -D libpthread.so.0 | grep pthread_init

objdump -tT libpthread.so.0 | grep pthread_init

# 查看静态库或动态库定义了哪些函数
nm -n --defined-only xxxx.a
nm -g -C --defined-only xxxx.so
nm -D xxxx.so

# 显示hello.a 中的未定义符号，需要和其他对象文件进行链接.
nm -u hello.o

# ❗这个挺有用的
# 在 ./ 目录下找出哪个库文件定义了close_socket函数. 
nm -A ./* 2>/dev/null | grep "T close_socket"


# 查看静态库定义的函数
readelf -c xxx.a

# 查看静态库定义的函数
readelf -A xxx.so 

# # 查看动态库有哪些符号，包括数据段、导出的函数和引用其他库的函数
objdump -tT xxx.so
objdump -x xxx.a

# 查看动态库依赖项
objdump -x xxx.so | grep "NEEDED" 

# 查看动态符号表
objdump -T xxx.so
## 假如想知道 xxx.so 中是否导出了符号 yyy ，那么命令为 objdump -T xxx.so | grep "yyy" 。

# 查看动态符号表
objdump -t xxx.so
## -T 和 -t 选项在于 -T 只能查看动态符号，如库导出的函数和引用其他库的函数，而 -t 可以查看所有的符号，包括数据段的符号。查看动态库有哪些符号，包括数据段、导出的函数和引用其他库的函数
objdump -tT xxx.so
objdump -x xxx.a

# 查看动态库依赖项
objdump -x xxx.so | grep "NEEDED" 

# 查看动态符号表
objdump -T xxx.so
## 假如想知道 xxx.so 中是否导出了符号 yyy ，那么命令为 objdump -T xxx.so | grep "yyy" 。

# 查看动态符号表
objdump -t xxx.so
## -T 和 -t 选项在于 -T 只能查看动态符号，如库导出的函数和引用其他库的函数，而 -t 可以查看所有的符号，包括数据段的符号。
```