# NeShellFFmpegCompilation Shell之脚本编写与执行编译ffmpeg库

[TOC]

## 支持流媒体播放的FFmpeg编译参考这里：[Linux环境下FFmpeg编译with libRTMP]()

## 一、概念

### 1.1 FFmpeg简介

FFmpeg是一套开源免费跨平台的多媒体框架，它既是一款音视频编解码工具，同时也是一组音视频编解码开发套件，作为编解码开发套件，它可以为我们提供丰富的音视频处理的调用接口，提供了录制、转换以及流化音视频的完整解决方案。

FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。它提供了媒体格式的封装与解封装，包括音视频编码、多种协议的流媒体、彩色格式转换、采样率转换、码率转换等。

FFmpeg是一个多媒体视频处理工具，具有非常强大的功能，包括：视频采集功能、视频格式转换、视频抓图、给视频加水印等。

FFmpeg框架提供了多种丰富的插件模块，包括封装与解封装的插件、编码与解码的插件等。

> FFmpeg中的“FF”指的是“Fast Forward”，"mpeg"指的是一种压缩率比较大的活动图像和声音的压缩标准。


### 1.2 FFmpeg组成
#### 1.2.1 命令行工具
* ffmpeg: 包含FFmpeg的各种功能，如gif、格式转换、截图、编解码等  
* ffmpeg-all: 包含FFmpeg工具及FFmpeg组件  
* ffplay: 一个使用了FFmpeg和SDL库的简单可移植的媒体播放器  
* ffplay-all: 包含ffplay工具及FFmpeg组件  
* ffprobe: 用来查看多媒体文件的信息  
* ffprobe-all: 包含ffprobe工具及FFmpeg组件  
* ffserver: 与流媒体服务器相关，负责响应客户端的流媒体请求，吧流媒体数据发送给客户端  
* ffserver-all: 包含ffserver工具及FFmpeg组件

#### 1.2.2 组件
* Utilities: libavutil提供的通用的features及工具  
* Video scaling and pixel format converter: 视频缩放和像素格式转换器  
* Audio resampler: 音频重新采用  
* Encoder and decoders(codecs): 编码和解码  
* Bitstream filters: 码流过滤器  
* Muxers and demuxers(formats): 数据合成及数据分离  
* Protocols: 协议  
* Input and output devices: 输入和输出设备  
* Filters: 过滤器  

#### 1.2.3 类库
* Libavutil：包含一些公共的工具函数使用库（包括算术运算、字符操作、随机数字产生器、数据结构、核心多媒体应用程序）  
* Libswscale：执行高度图像缩放和色彩空间、像素格式的转换操作，如RGB565、RGB888等于YUV420等之间的转换  
* Libswresample：执行高度优化的音视频重采样工具，重新矩阵化和样本格式转换工作 
* Libavcodec：包含音视频编解码库，用于各种类型声音/图像编解码。该库是音视频编解码核心库，实现了市面上可见的绝大部分编解码器的功能  
* Libavformat：用于各种音视频封装格式的生成和解析，包括获取解码所需信息以生成解码上下文结果和读取音视频帧等功能。音视频的格式解析协议，为libavcodec分享码流提供度量的音频或视频码流源  
* Libavdevice：硬件采集、加速、显示，包含输入、输出的设备，用于多媒体输入输出的实现。操作计算机中常见的音视频捕获或输出设备有ALSA、AUDIO_BEOS、JACK、OSS、1394、VFW等 
* Libavfilter：包含多媒体过滤器的库，filter(FileIO、FPS、DrawText)音视频滤波器的开发，如宽高比、裁剪、格式化、非格式化、伸缩等  
* Libavresample: 音视频封装编解码格式预设等  
* Libpostproc: (同步、时间计算的简单算法)用于后期效果处理，音视频应用的后处理，如图像的去块效应
#### 1.2.1 FFmpeg工具(已经编译好的)

* ffmpeg（编解码工具）

  ffmpeg 是FFmpeg源码编译后生成的可执行程序，可以作为命令行工具来使用。

  > 例如：`./ffmpeg -i input.mp4 output.avi`
  >
  > 这是一条简单的ffmpeg命令，通过`-i`参数将`input.mp4`作为输入源，然后进行转码和转封装操作，输出到`output.avi`中。
  >
  > 上面指令也可以写成：`./ffmpeg -i input.mp4 -f avi output.dat`
  >
  > `-f`参数指定了输出文件的容器格式，输出的文件`output.dat`同样是`avi`格式文件（不要光看文件后缀名就断定文件封装格式）。

  ffmpeg 大致处理流程：

  ​	① 读取输入源

  ​	② 解封装

  ​	③ 解码

  ​	④ 重新编码

  ​	⑤ 重新封装

  ​	⑥ 输出目标文件

  

* ffplay（播放器）

  ffplay也是FFmpeg源码编译生成的一个可执行程序，通常作为测试工具来使用。ffplay提供了音视频显示和播放相关的图像信息，音频的波形信息等。

  > 播放`input.mp4`文件：`./ffplay input.mp4`  

  

* ffprobe（多媒体分析工具）    

  ffplay也是FFmpeg源码编译生成的一个可执行程序，可以从媒体文件或媒体流中获取详细的媒体信息，如音视频的编码格式、媒体容器的参数信息等。

  > 查看`input.mp4`中流的信息：`./ffprobe -show_streams input.mp4`

### 1.3 如何使用FFmpeg
> FFmpeg是由c代码编写而成，功能多，代码量大  
> 在Android平台使用需要先编译，后使用，编译可以通过MakeFile语法来进行编译  

## 二、NDK编译FFmpeg
### 2.1 准备工作
#### 2.1.1 下载NDK r17 版本
从[https://developer.android.google.cn/ndk/downloads/older_releases.html](https://developer.android.google.cn/ndk/downloads/older_releases.html)下载 r17 版本，并解压。  
```bash
unzip android-ndk-r17c-linux-x86_64.zip
```
#### 2.1.2 配置NDK环境变量
* 进入NDK解压目录，查看当前路径  
```bash
pwd		# /Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
```
* 修改配置文件  
```bash
vim /etc/profile    # 若没有写权限，请先添加权限
```
* 添加NDK路径到配置文件中  
```bash
NDKROOT=/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
export PATH=$NDKROOT:$PATH 		# 添加到环境变量中
```
* 使配置生效  
```bash
source /etc/profile
```
* 测试配置是否成功
```bash
ndk-build
```
若打印以下信息则表示配置成功：  
```bash
Android NDK: Could not find application project directory !    
Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.    
/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c/build/core/build-local.mk:151: *** Android NDK: Aborting    .  Stop.
```

#### 2.1.3 下载FFmpeg
从[http://www.ffmpeg.org/download.html](http://www.ffmpeg.org/download.html)下载FFmpeg 4.0.5 版本，并解压。  
```bash
tar xvf ffmpeg-4.0.5.tar.gz
```

### 2.2 编译FFmpeg(Ubuntu的Linux 环境)
#### 2.2.1 修改configure文件
进入FFmpeg目录，修改configure文件：不修改的话编译出来的.so文件会有一串数字，无法使用，所以得修改命名规则，使编译出来的so后缀不带数字，可以被Android识别。
```bash
cd ffmpeg-4.0.5
vim configure
```
修改内容如下：  
```bash
#SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
#LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
#SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
#SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
#### 2.2.2 编写编译动态库脚本
在FFmpeg目录中，编写build.sh脚本  
```bash 
vim build.sh
```
脚本内容如下：  
```bash
#!/bin/bash
NDK_ROOT=/home/tian/NDKWorkspace/tools/NDK/android-ndk-r17c
#TOOLCHAIN 变量指向ndk中的交叉编译gcc所在的目录
TOOLCHAIN=$NDK_ROOT/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/
#FLAGS与INCLUDES变量 可以从AS ndk工程的.externativeBuild/cmake/debug/armeabi-v7a/build.ninja中拷贝，需要注意的是**地址**
FLAGS="-isystem $NDK_ROOT/sysroot/usr/include/arm-linux-androideabi -D__ANDROID_API__=21 -g -DANDROID -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -Wa,--noexecstack -Wformat -Werror=format-security -std=c++11  -O0 -fPIC"
INCLUDES="-isystem $NDK_ROOT/sources/cxx-stl/llvm-libc++/include -isystem $NDK_ROOT/sources/android/support/include -isystem $NDK_ROOT/sources/cxx-stl/llvm-libc++abi/include"

#执行configure脚本，用于生成makefile
#--prefix : 安装目录
#--enable-small : 优化大小
#--disable-programs : 不编译ffmpeg程序(命令行工具)，我们是需要获得静态(动态)库。
#--disable-avdevice : 关闭avdevice模块，此模块在android中无用
#--disable-encoders : 关闭所有编码器 (播放不需要编码)
#--disable-muxers :  关闭所有复用器(封装器)，不需要生成mp4这样的文件，所以关闭
#--disable-filters :关闭视频滤镜
#--enable-cross-compile : 开启交叉编译（ffmpeg比较**跨平台**,并不是所有库都有这么happy的选项 ）
#--cross-prefix: 看右边的值应该就知道是干嘛的，gcc的前缀 xxx/xxx/xxx-gcc 则给xxx/xxx/xxx-
#disable-shared enable-static 不写也可以，默认就是这样的。
#--sysroot: 
#--extra-cflags: 会传给gcc的参数
#--arch --target-os :
PREFIX=./android/armeabi-v7a2
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
--enable-shared \
--disable-static \
--sysroot=$NDK_ROOT/platforms/android-21/arch-arm \
--extra-cflags="$FLAGS $INCLUDES" \
--extra-cflags="-isysroot $NDK_ROOT/sysroot" \
--arch=arm \
--target-os=android 

make clean
make install
```
#### 2.2.3 执行脚本
```bash
sh build.sh
```
编译一段时间，成功后会在当前目录下生成 android/armeabi-v7a2 文件夹，文件夹下lib目录下的就是编译冲的.so动态库文件。  


### 2.3 编译FFmpeg(MAC 环境)
#### 2.3.1 修改configure文件
进入FFmpeg目录，修改configure文件：不修改的话编译出来的.so文件会有一串数字，无法使用，所以得修改命名规则，使编译出来的so后缀不带数字，可以被Android识别。
```bash
cd ffmpeg-4.0.5
vim configure
```
修改内容如下：  
```bash
#SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
#LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
#SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
#SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
#### 2.3.2 编写编译脚本
进入FFmpeg目录，编写buildmac.sh脚本  
```bash
cd ffmpeg-4.0.5 
vim buildmac.sh
```
脚本内容如下：   
```bash
#!/bin/bash
ADDI_CFLAGS="-marm"
# 指明交叉编译时使用的是哪个版本的头文件和库文件，它是SYSROOT的一部分
PLATFORM=arm-linux-androideabi
API=27
CPU=armv7-a
# ndk环境
NDK_ROOT=/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
# 指明交叉编译目标机器的头文件和库文件目录
SYSROOT=$NDK_ROOT/platforms/android-$API/arch-arm/
ISYSROOT=$NDK_ROOT/sysroot
ASM=$ISYSROOT/usr/include/$PLATFORM
# 指明交叉编译工具链的位置
TOOLCHAIN=$NDK_ROOT/toolchains/$PLATFORM-4.9/prebuilt/darwin-x86_64
# 交叉编译后的输出目录，这里保存在当前目录下的android/armeabi-v7a2
PREFIX=/Users/tian/NeCloud/NDKWorkspace/linuxdir/ffmpeg/ffmpeg-4.0.5/android/armeabi-v7a2
ADDI_CFLAGS="-marm"

function build_one
{
./configure \
--target-os=android \
--prefix=$PREFIX \
--enable-cross-compile \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-avdevice \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--arch=arm \
--sysroot=$SYSROOT \
--extra-cflags="-I$ASM -isysroot$ISYSROOT -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG

# -isysroot $ISYSROOT 
make clean
# 不确定自己在上面的目录或者环境有没有错误时，可以先注销一下下面两个命令
make
make install
}
echo "开始编译ffmpeg"
build_one
echo "完成编译..."
```
#### 2.3.2 执行脚本
```bash
sh buildmac.sh
```
编译一段时间，成功后会在当前目录下生成 android/armeabi-v7a2 文件夹，文件夹下lib目录下的就是编译冲的.so动态库文件。  


## 三、采坑(主要是Mac环境)
### 3.1 坑一
Mac环境下执行脚本时报`nasm/yasm not found `错误 
```bash
nasm/yasm not found or too old. Use --disable-x86asm for a crippled build.
```
**解决方案**  
1. Download yasm source code from   
[http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz](http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz)  
2. 在终端执行命令
```bash
tar xvzf yasm-1.2.0.tar.gz  # unpack
cd yasm-1.2.0
./configure && make -j 4 && sudo make install    #Configure and build   -j 4 表示4个并发执行线程
```

### 3.2 坑二
解决完坑一之后又报了如下错误：  
```bash
/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-gcc is unable to create an executable file.
C compiler test failed.
```
**解决方案**  
参考[在Mac下使用NDK编译FFmpeg3.3.4](https://www.jianshu.com/p/5d738f645697),严格按照该文档中构建方法中`./configure` 后参数的顺序修改了参数顺序，结果就解决了，有些莫名其妙。

### 3.3 坑三
一坑一坑又一坑，刚解决完坑二，又报了下面的错误：  
```bash
ffmpeg 30:20: fatal error: stdlib.h: No such file or directory  #include <stdlib.h>
```
**解决方案**  
参考[FFmpeg编译报错[libavfilter/aeval.o] Error 1](https://www.jianshu.com/p/e8c6c634bcf5),文中提到：  

> 后来发现是因为新版本的NDK引起的，因为NDK将头文件和库文件进行了分离，指定的sysroot只有库文件，头文件放在NDK目录下的sysroot中，只需要在--extra-cflags中添加“-isysroot$NDK/sysroot"即可，  
> 还有有关汇编的头文件也进行了分离，需要根据目标平台进行指定”-I$NDK/sysroot/usr/inclued/arm-linux-androideabi",将“arm-linux-androideabi"改成需要的平台就可以，终于可以顺利地进行编译了。  

所以将  
```bash
--extra-cflags="-fpic $ADDI_CFLAGS" \
```
改为  
```bash
--extra-cflags="-I$ASM -isysroot$ISYSROOT -fpic $ADDI_CFLAGS" \
```
就可以了。  
最后终于是成功编译出了动态库，真的太不容易了。
