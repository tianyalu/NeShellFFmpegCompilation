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

