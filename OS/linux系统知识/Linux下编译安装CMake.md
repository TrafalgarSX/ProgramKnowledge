### 编译安装程序的流程
编译安装CMake的流程，e.g.
```bash
# 通过wget下载源码
wget https://cmake.org/files/v3.23/cmake-3.23.0.tar.gz 
wget可能会报 无法建立ssl连接的错误，有几个解决方法，一个是wget版本过旧，update wget, 一个是在命令后加 --no-check-certificate,  或者关闭防火墙等

tar -zxvf cmake-3.23.0.tar.gz # 解压

mv cmake-3.23.0 cmake # 修改下文件夹的名称，方便处理

cd cmake

apt -y install libssl-dev(ubuntu)
yum -y install libssl-devel (centos)

./configure

make -j8

make install

cmake --version # 查看cmake版本

如果执行命令出现 。。CMAKE_ROOT。。。相关的错误,就执行：
hash -r

cmake --version # 再次查看cmake能否正常使用
```

在Linux系统下如果 yum源或者apt源的软件版本不能满足要求，就可以使用源码编译安装的方式安装自己想要的软件版本。

基本所有的软件都可以使用上面的流程来实现从源码编译安装到自己的机器上。

不过首要问题是去哪找 源码，源码的网址在哪。