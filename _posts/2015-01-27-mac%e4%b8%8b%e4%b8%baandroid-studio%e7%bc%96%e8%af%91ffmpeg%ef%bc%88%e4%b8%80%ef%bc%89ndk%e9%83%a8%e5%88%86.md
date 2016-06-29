---
id: 4533
title: Mac下为Android Studio编译Ffmpeg（一）ndk部分
date: 2015-01-27T11:25:00+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4533
permalink: /4533.html
categories:
  - 备忘
  - 实践
tags:
  - Android
  - Android Studio
  - ffmpeg
  - mac
  - ndk
  - 安卓
---
本文参考http://www.roman10.net/how-to-build-ffmpeg-with-ndk-r9/。
  
但它只适合做编码，而且没有Android Studio配置的部分。

## 1、下载ndk

我下的是r10d版本。

## 2、解压ndk

不要解压，文件权限会出错。执行之，会自动解压，然后mv到想放的地方。我放到了&#8221;/usr/local/bin/android-ndk-r10d&#8221;（此目录之后用$NDK_DIR指代）。

## 3、下载Ffmpeg

我下的是2.5.3版本。

## 4、解压Ffmpeg

解压Ffmpeg到$NDK_DIR/sources/ffmpeg-2.5.3。

## 5、修改Ffmpeg编译配置

在ffmpeg-2.5.3目录下把configure文件中的这几行，目的是去掉默认生成的库名字libavcodec.so.55最后那个&#8221;55&#8243;的版本号。

<pre>SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
</pre>

<pre>SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
</pre>

## 6、编译Ffmpeg

在ffmpeg-2.5.3目录下创建文件build_android.sh。
  
注意前三行要按照自己的路径正确配置。

<pre>#!/bin/bash
NDK=/usr/local/android-ndk-r10d
SYSROOT=$NDK/platforms/android-15/arch-arm/
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
function build_one
{
./configure \
    --prefix=$PREFIX \
    --enable-shared \
    --disable-static \
    --disable-doc \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --disable-avdevice \
    --disable-doc \
    --disable-symver \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --target-os=linux \
    --arch=arm \
    --enable-cross-compile \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}
CPU=arm
PREFIX=$(pwd)/android/$CPU 
ADDI_CFLAGS="-marm"
build_one
</pre>

保存后执行

<pre>sudo chmod +x build_android.sh
./build_android.sh
</pre>

编译会花上一段时间。

## 7、查看编译结果

编译完成后$NDK_DIR/sources/ffmpeg-2.5.3下面会多出一个android目录，里面就是我们想要的编译好的库。

<pre>[cg@local]# ls -R android/
arm

android//arm:
Android.mk include    lib

android//arm/include:
libavcodec    libavfilter   libavformat   libavutil     libswresample libswscale

android//arm/include/libavcodec:
avcodec.h       avfft.h         dv_profile.h    dxva2.h         old_codec_ids.h vaapi.h         vda.h           vdpau.h         version.h       vorbis_parser.h xvmc.h

android//arm/include/libavfilter:
asrc_abuffer.h  avcodec.h       avfilter.h      avfiltergraph.h buffersink.h    buffersrc.h     version.h

android//arm/include/libavformat:
avformat.h avio.h     version.h

android//arm/include/libavutil:
adler32.h        avstring.h       cast5.h          downmix_info.h   hash.h           macros.h         opt.h            replaygain.h     time.h
aes.h            avutil.h         channel_layout.h error.h          hmac.h           mathematics.h    parseutils.h     ripemd.h         timecode.h
attributes.h     base64.h         common.h         eval.h           imgutils.h       md5.h            pixdesc.h        samplefmt.h      timestamp.h
audio_fifo.h     blowfish.h       cpu.h            ffversion.h      intfloat.h       mem.h            pixelutils.h     sha.h            version.h
audioconvert.h   bprint.h         crc.h            fifo.h           intreadwrite.h   motion_vector.h  pixfmt.h         sha512.h         xtea.h
avassert.h       bswap.h          dict.h           file.h           lfg.h            murmur3.h        random_seed.h    stereo3d.h
avconfig.h       buffer.h         display.h        frame.h          log.h            old_pix_fmts.h   rational.h       threadmessage.h

android//arm/include/libswresample:
swresample.h version.h

android//arm/include/libswscale:
swscale.h version.h

android//arm/lib:
libavcodec-56.so   libavfilter-5.so   libavformat-56.so  libavutil-54.so    libswresample-1.so libswscale-3.so    pkgconfig
libavcodec.so      libavfilter.so     libavformat.so     libavutil.so       libswresample.so   libswscale.so

android//arm/lib/pkgconfig:
libavcodec.pc    libavfilter.pc   libavformat.pc   libavutil.pc     libswresample.pc libswscale.pc
</pre>

其中libavcodec.so、libavfilter.so、libavformat.so、libavutil.so、libswresample.so、libswscale.so都是软链，没有用，可以删掉。

## 8、给Ffmpeg库写Android.mk使其可用

创建$NDK_DIR/sources/ffmpeg-2.5.3/android/arm/Android.mk文件，内容如下：
  
要注意其中.so前面的数字应该改成你的ffmpeg版本编译出来的数字。

<pre>LOCAL_PATH:= $(call my-dir)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libavcodec
LOCAL_SRC_FILES:= lib/libavcodec-56.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libavformat
LOCAL_SRC_FILES:= lib/libavformat-56.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libswscale
LOCAL_SRC_FILES:= lib/libswscale-3.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libavutil
LOCAL_SRC_FILES:= lib/libavutil-54.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libavfilter
LOCAL_SRC_FILES:= lib/libavfilter-5.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)
 
 include $(CLEAR_VARS)
LOCAL_MODULE:= libswresample
LOCAL_SRC_FILES:= lib/libswresample-1.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_SHARED_LIBRARY)

</pre>

至此ndk配置完毕，后面是配置Android Studio的部分。
  
见这个链接：[Mac下为Android Studio编译Ffmpeg（二）Android Studio部分](http://blog.yikuyiku.com/?p=4539 "Mac下为Android Studio编译Ffmpeg（二）Android Studio部分")