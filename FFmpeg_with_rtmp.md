# Linux环境下 FFmpeg 编译 with rtmp

[TOC]

本文参考：[最新版本FFmpeg编译(基于v4.2.1)](https://www.jianshu.com/p/212c61cac89c)

## 1. 环境准备

本文环境为阿里云服务器，系统为Centos.  

### 1.1 下载ndk

[ndk历史版本下载地址](https://developer.android.google.cn/ndk/downloads/older_releases.html)

这里我们下载Linux版ndk(android-ndk-r17c=linux-x86_64.zip)

```bash
wget https://dl.google.com/android/repository/android-ndk-r17c-linux-x86_64.zip
```

解压ndk

```bash
unzip android-ndk-r17c-linux-x86_64.zip
```

## 2. 编译FFmpeg

### 2.1 下载FFmpeg

下载 [FFmpeg](https://ffmpeg.org/download.html) （ffmpeg-4.2.1.tar.bz2）  

```bash
wget https://ffmpeg.org/releases/ffmpeg-4.2.2.tar.bz2
```

解压FFmpeg

```bash
tar -xjvf ffmpeg-4.2.1.tar.bz2
```

### 2.2 修改configure文件

configure文件其实是一个shell脚本，用于生成MakeFile文件，然后可以使用make命令执行。我们可以通过执行 `./configure --help` 来查看编译相关信息：  

```bash
#将help信息输出到文本中便于查看
./configure --help > ffmpeg_configure_help.txt
#发送到windows端查看更happy
sz ffmpeg_configure_help.txt
```

最新版本的ffmpeg默认使用clang编译，可以修改configure文件来关闭:

```bash
# 注释4210-4213行，简单粗暴
# set_default target_os
# if test "$target_os" = android; then
#     cc_default="clang"
# fi
```

### 2.3 编写交叉编译FFmpeg的shell脚本 build.sh  

```bash
#!/bin/bash

#NDK_ROOT 变量指向ndk目录
NDK_ROOT=/root/software/android-ndk-r17c
#TOOLCHAIN 变量指向ndk中的交叉编译gcc所在的目录
TOOLCHAIN=$NDK_ROOT/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

#指定android api版本
ANDROID_API=17

#此变量用于编译完成之后的库与头文件存放在哪个目录
PREFIX=./android/armeabi-v7a

#执行configure脚本，用于生成makefile
#--prefix : 安装目录
#--enable-small : 优化大小
#--disable-programs : 不编译ffmpeg程序(命令行工具)，我们是需要获得静态(动态)库。
#--disable-avdevice : 关闭avdevice模块，此模块在android中无用
#--disable-encoders : 关闭所有编码器 (播放不需要编码)
#--disable-muxers :  关闭所有复用器(封装器)，不需要生成mp4这样的文件，所以关闭
#--disable-filters :关闭视频滤镜
#--enable-cross-compile : 开启交叉编译
#--cross-prefix: gcc的前缀 xxx/xxx/xxx-gcc 则给xxx/xxx/xxx-
#disable-shared enable-static 不写也可以，默认就是这样的。
#--sysroot: 
#--extra-cflags: 会传给gcc的参数
#--arch --target-os : 必须要给
./configure \
--prefix=$PREFIX \
--enable-small \
--disable-programs \
--disable-avdevice \
--disable-encoders \
--disable-muxers \
--disable-filters \
--enable-cross-compile \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--disable-shared \
--enable-static \
--sysroot=$NDK_ROOT/platforms/android-$ANDROID_API/arch-arm \
--extra-cflags="-isysroot $NDK_ROOT/sysroot -isystem $NDK_ROOT/sysroot/usr/include/arm-linux-androideabi -D__ANDROID_API__=$ANDROID_API -U_FILE_OFFSET_BITS  -DANDROID -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -Wa,--noexecstack -Wformat -Werror=format-security  -O0 -fPIC" \
--arch=arm \
--target-os=android

#上面运行脚本生成makefile之后，使用make执行脚本
make clean
make install
```

### 2.4 执行编译脚本build.sh

```bash
# 首先要给脚本加上可执行权限
chmod +x build.sh
# 执行脚本
./build.sh
```

编译结束后静态库会在`./android/armeabi-v7a` 目录下生成。

也可以将静态库打包生成一个so库：  

```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ ffmpeg-4.2.2]# cd android/
[root@iZwz9ci7skvj0jj2sfdmqgZ android]# cd armeabi-v7a/
[root@iZwz9ci7skvj0jj2sfdmqgZ armeabi-v7a]# ls
include  lib  share
[root@iZwz9ci7skvj0jj2sfdmqgZ armeabi-v7a]# cd lib
[root@iZwz9ci7skvj0jj2sfdmqgZ lib]# ls
libavcodec.a  libavfilter.a  libavformat.a  libavutil.a  libswresample.a  libswscale.a  pkgconfig
[root@iZwz9ci7skvj0jj2sfdmqgZ lib]# export CC=/root/software/android-ndk-r17c/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc
[root@iZwz9ci7skvj0jj2sfdmqgZ lib]# export AA="--sysroot=/root/software/android-ndk-r17c/platforms/android-17/arch-arm/"
[root@iZwz9ci7skvj0jj2sfdmqgZ lib]# $CC $AA -shared -o libffmpeg.so -Wl,--whole-archive *.a -Wl,--no-whole-archive
[root@iZwz9ci7skvj0jj2sfdmqgZ lib]# ls
libavcodec.a  libavfilter.a  libavformat.a  libavutil.a  libffmpeg.so  libswresample.a  libswscale.a  pkgconfig
```

## 3. 编译librtmp

### 3.1 下载librtmp

下载 [rempdump](http://rtmpdump.mplayerhq.hu/download/) （rtmpdump-2.3-android.zip）

```bash
wget http://rtmpdump.mplayerhq.hu/download/rtmpdump-2.3.tgz
```

解压

```ba
tar -xjvf rtmpdump-2.3.tgz
```

### 3.2 编写编译脚本build.sh

脚本内容如下：  

```bash
#!/bin/bash

NDK_ROOT=/root/software/android-ndk-r17c

CPU=arm-linux-androideabi

TOOLCHAIN=$NDK_ROOT/toolchains/$CPU-4.9/prebuilt/linux-x86_64

export XCFLAGS="-isysroot $NDK_ROOT/sysroot -isystem $NDK_ROOT/sysroot/usr/include/arm-linux-androideabi -D__ANDROID_API__=17"
export XLDFLAGS="--sysroot=${NDK_ROOT}/platforms/android-17/arch-arm "
export CROSS_COMPILE=$TOOLCHAIN/bin/arm-linux-androideabi-

make install SYS=android prefix=`pwd`/install CRYPTO= SHARED=  XDEF=-DNO_SSL 

```

### 3.3 执行编译脚本build.sh

```bash
# 首先要给脚本加上可执行权限
sudo chmod +x build.sh
# 执行脚本
./build.sh
```

编译结束后目标文件在`./install` 目录下生成

## 4. FFmpeg编译集成librtmp

### 4.1 继续修改ffmpeg下的configure文件

由于ffmpeg默认开启librtmp需要pkgconfig，这里我们手动关闭，修改ffmpeg的configure文件：

```bash
# 注释掉6256行，简单粗暴

enabled libpulse          && require_pkg_config libpulse libpulse pulse/pulseaudio.h pa_context_new
enabled librsvg           && require_pkg_config librsvg librsvg-2.0 librsvg-2.0/librsvg/rsvg.h rsvg_handle_render_cairo
# enabled librtmp           && require_pkg_config librtmp librtmp librtmp/rtmp.h RTMP_Socket
enabled librubberband     && require_pkg_config librubberband "rubberband >= 1.8.1" rubberband/rubberband-c.h rubberband_new -lstdc++ && append librubberband_extralibs "-lstdc++"
```

### 4.2 修改编译脚本

修改ffmpeg编译脚本，这里将前面脚本copy一份重命名为`build_ffmpeg_with_librtmp.sh` 来修改：

```bash
cp build.sh build_ffmpeg_with_librtmp.sh
```

脚本内容如下：

```bash
#!/bin/bash

#NDK_ROOT 变量指向ndk目录
NDK_ROOT=/root/software/android-ndk-r17c

CPU=arm-linux-androideabi

#TOOLCHAIN 变量指向ndk中的交叉编译gcc所在的目录
TOOLCHAIN=$NDK_ROOT/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

#指定android api版本
ANDROID_API=17

#此变量用于编译完成之后的库与头文件存放在哪个目录
PREFIX=./build_outputs/armeabi-v7a/ffmpeg_rtmp

#rtmp路径
RTMP=/root/software/rtmpdump-2.3/install

#执行configure脚本，用于生成makefile
#--prefix : 安装目录
#--enable-small : 优化大小
#--disable-programs : 不编译ffmpeg程序(命令行工具)，我们是需要获得静态(动态)库。
#--disable-avdevice : 关闭avdevice模块，此模块在android中无用
#--disable-encoders : 关闭所有编码器 (播放不需要编码)
#--disable-muxers :  关闭所有复用器(封装器)，不需要生成mp4这样的文件，所以关闭
#--disable-filters :关闭视频滤镜
#--enable-cross-compile : 开启交叉编译
#--cross-prefix: gcc的前缀 xxx/xxx/xxx-gcc 则给xxx/xxx/xxx-
#disable-shared enable-static 不写也可以，默认就是这样的。
#--sysroot: 
#--extra-cflags: 会传给gcc的参数
#--arch --target-os : 必须要给
./configure \
--prefix=$PREFIX \
--enable-small \
--disable-programs \
--disable-avdevice \
--disable-encoders \
--disable-muxers \
--disable-filters \
--enable-librtmp \
--enable-cross-compile \
--cross-prefix=$TOOLCHAIN/bin/$CPU- \
--disable-shared \
--enable-static \
--sysroot=$NDK_ROOT/platforms/android-$ANDROID_API/arch-arm \
--extra-cflags="-isysroot $NDK_ROOT/sysroot -isystem $NDK_ROOT/sysroot/usr/include/arm-linux-androideabi -D__ANDROID_API__=$ANDROID_API -U_FILE_OFFSET_BITS  -DANDROID -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -Wa,--noexecstack -Wformat -Werror=format-security  -O0 -fPIC" \
--extra-libs="-lrtmp" \
--arch=arm \
--target-os=android

#上面运行脚本生成makefile之后，使用make执行脚本
make clean
make install

```

### 4.3 执行编译脚本 build_ffmpeg_with_librtmp.sh

```bash
# 首先要给脚本加上可执行权限
chmod +x build_ffmpeg_with_librtmp.sh
# 执行脚本
./build_ffmpeg_with_librtmp.sh
```

编译较长一段时间后目标文件在`./build_outputs/armeabi-v7a/ffmpeg_rtmp` 目录下生成。

4.4 将生成的目标文件压缩回传到windows

```bash
 zip -r ffmpeg_rtmp.zip ffmpeg_rtmp/
```

可以通过Xshell的Xftp将压缩文件传回到windows。

## 5. 使用

参考：[https://github.com/tianyalu/NeFFmpegPlayer](https://github.com/tianyalu/NeFFmpegPlayer)