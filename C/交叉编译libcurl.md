### 如何在Ubuntu上交叉编译Android的libcurl库
使用autoconf 生成Makefile 方式
```cmake
# Check out the source.
# Only choose one of these, depending on your build machine..
#export TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/darwin-x86_64
export ANDROID_NDK_HOME=~/Android/android-ndk-r25b  # Point into your NDK.
export HOST_TAG=linux-x86_64 # Same tag for Apple Silicon. Other OS values here: https://developer.android.com/ndk/guides/other_build_systems#overview
export TOOLCHAIN=$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/$HOST_TAG

# Only choose one of these, depending on your device...
#export TARGET=aarch64-linux-android    # 64位Android
export TARGET=armv7a-linux-androideabi  # 32位Android
# export TARGET=i686-linux-android
# export TARGET=x86_64-linux-android

# Set this to your minSdkVersion.
export API=29
# Configure and build.
export AR=$TOOLCHAIN/bin/llvm-ar
export AS=$TOOLCHAIN/bin/llvm-as
#export AS=$CC

# 下面的变量拼接有问题，所以使用了绝对路径
#export CC=$TOOLCHAIN/bin/$TARGET$API-clang
#export CXX=$TOOLCHAIN/bin/$TARGET$API-clang++
export CC=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi29-clang
export CXX=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi29-clang++

export LD=$TOOLCHAIN/bin/ld
export RANLIB=$TOOLCHAIN/bin/llvm-ranlib
export STRIP=$TOOLCHAIN/bin/llvm-strip

./configure --host $TARGET   --disable-shared  --without-ssl --prefix="/root/curlInstallV7"
#--disable-shared  --with-pic

```

TODO 
AS AR RANLIB  STRIP 分别是什么？
为什么  交叉编译需要这几个环境变量

cmake也有相同的： CMAKE_AS等

TODO 了解如何用cmake进行交叉编译？