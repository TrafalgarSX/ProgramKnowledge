### 如何在Ubuntu上交叉编译Android的libcurl库
使用autoconf 生成Makefile 方式
```shell
#!/bin/bash

# 不受参数影响
# Only choose one of these, depending on your build machine..
#export TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/darwin-x86_64
export ANDROID_NDK_HOME=~/Android/android-ndk-r25b  # Point into your NDK.
export HOST_TAG=linux-x86_64 # Same tag for Apple Silicon. Other OS values here: https://developer.android.com/ndk/guides/other_build_systems#overview
export TOOLCHAIN=$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/$HOST_TAG
# Configure and build.
export AR=$TOOLCHAIN/bin/llvm-ar
export AS=$TOOLCHAIN/bin/llvm-as
#export AS=$CC
export LD=$TOOLCHAIN/bin/ld
export RANLIB=$TOOLCHAIN/bin/llvm-ranlib
export STRIP=$TOOLCHAIN/bin/llvm-strip

# 默认值
prefix="/root/iconvInstallV7"
# prefix="/root/iconvInstall"

# -t target
# -a api minSdkVersion
# -p prefix
while getopts "t:a:p:h" arg # 选项后面的冒号标识该选项需要参数
do
  case "$arg" in
      t)
          echo "t's target:$OPTARG"
          target=${OPTARG}
          ;;
      a)
          echo "a's api:$OPTARG"
          api=${OPTARG}
          ;;
      p)
          echo "p's prefix:$OPTARG"
          prefix="${OPTARG}"
          ;;
      h)
          echo "-t target: 32bit or 64bit"
          echo "-a api: Android compile use this minSdkVersion 21 - 29"
          echo "-p prefix: install to this directory"
          echo "-h help"
          ;;
      ?)
          echo "unkonw argument"
          ;;
  esac
done

if [ ${target} -eq 32 ]; then
    # 32bit
    export TARGET=armv7a-linux-androideabi

else
    # 64bit
    export TARGET=aarch64-linux-android
fi

# Set this to your minSdkVersion.
export API=${api}

# ${TARGET}${API} aarch64-linux-android21 这是目标三元组， 最后是minSdkVersion
export CC=${TOOLCHAIN}/bin/$TARGET${API}-clang
export CXX=${TOOLCHAIN}/bin/${TARGET}${API}-clang++

echo ${CC}
echo ${CXX}
echo ${prefix}

# 最后的执行命令
./configure --host ${TARGET}   --disable-shared  --prefix=${prefix} > /tmp/out
#--disable-shared  --with-pic

cat /tmp/out | grep -niw "warning\|error\|FAILED\|SUCCEEDED\|no" -A3 --color=auto
# 彩色输出
echo -e "\033[41m----------------------------\033[0m";
tail -n 30 /tmp/out
# make clean 
# make && make install

# almost don't use
# export TARGET=i686-linux-android
# export TARGET=x86_64-linux-android

# export CC=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi21-clang
# export CXX=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi21-clang++

# export CC=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang
# export CXX=/root/Android/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang++
```

TODO 
AS AR RANLIB  STRIP 分别是什么？
为什么  交叉编译需要这几个环境变量

cmake也有相同的： CMAKE_AS等

TODO 了解如何用cmake进行交叉编译？