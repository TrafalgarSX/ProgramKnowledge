### 如何编译libcurl
1. 可以使用CMake编译，但是目前还没有完整阅读过libcurl的CMakeLists.txt，不能从命令行控制生成curl的依赖等，eg.是否带着OpenSSL编译。CMakeLists.txt中可以进行控制，但是生成的结果找不到静态，需要进一步学习。
2. curl目录下有configure.ac文件，此文件配合命令行程序autoconf可以生成configure文件。
```bash
autoreconf -i
```
>带有参数运行configure文件，可以像cmake一样产生对应的Makefile文件。
>接下来运行 `make && make install` 就可以生成编译程序
```bash
# 常见的configure参数
./configure --prefix="/path/to/make/install"  --without-ssl
```

```
apt install build-essential

apt install autoconf

进入libcurl目录
autoreconf -i 

./configure   --without-ssl  
如果出现：# 无法解决 R_AARCH64_ADR_PREL_PG_HI21 重定向于符号 “sinh@@GLIBC_2.17” 有冲突
./configure   --without-ssl  --extra-cflags=-fPIC  ffmpeg可以这样
 make CFLAGS='$CFLAGS -fpic'  这样也行，直接修改Makefile也行
一般这个aarch64-himix100-linux- 编译工具编译的动态库编译的.so文件，中间的 *.o 都要用 -fPIC 生成，才会不报错。
make && make install

然后到 /usr/local/lib/文件夹下看有没有  libcurl.a

```

```bash
case 1:
./configure --without-ssl CC=/usr/bin/clang CXX=/usr/bin/clang++

make && make install

case 2:
add_library(libcurl SHARED ...)
target_link_libraries(libcurl ..    .so)
```