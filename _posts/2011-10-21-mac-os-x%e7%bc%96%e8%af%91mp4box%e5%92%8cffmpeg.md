---
id: 2654
title: Mac OS X编译MP4Box和ffmpeg
date: 2011-10-21T09:18:47+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2654
permalink: /2654.html
categories:
  - 备忘
  - 实践
tags:
  - ffmpeg
  - mac
  - mp4box
---
相关代码已经打好包了，安装脚本在tar里。

## Mac平台Ffmpeg编译

<pre class="brush: bash">cd faac-*/
CFLAGS="-D__unix__" ./configure && make -j 4 && sudo make install; cd ..

cd lame-*/
./configure && make -j 4 && sudo make install; cd ..

cd libogg-*/
./configure && make -j 4 && sudo make install; cd ..

cd libvorbis-*/
./configure --disable-oggtest && make -j 4 && sudo make install; cd ..

cd libtheora-*/
./configure --disable-oggtest --disable-vorbistest --disable-examples --disable-asm
make -j 4 && sudo make install; cd ..

cd vo-amrwbenc-*/
./configure && make -j 4 && sudo make install; cd ..

cd yasm-*/
./configure && make -j 4 && sudo make install; cd ..

cd libvpx-*/
./configure --enable-vp8 --enable-psnr --enable-postproc --enable-multithread --enable-runtime-cpu-detect --disable-install-docs --disable-debug-libs --disable-examples && make -j 4 && sudo make install; cd ..

cd x264-*
CFLAGS="-I. -fno-common -read_only_relocs suppress" ./configure --enable-pic --enable-shared && make -j 4 && sudo make install; cd ..

cd xvidcore/build/generic
./configure --disable-assembly && make -j 4 && sudo make install; cd ../../..

# For Lion, we have to change which compiler to use (--cc=clang).
# If you're building on Snow Leopard, you can omit this flag so it defaults to gcc.
cd ffmpeg-*/
CFLAGS="-DHAVE_LRINTF" ./configure --cc=clang --enable-nonfree --enable-gpl --enable-version3 --enable-postproc --enable-swscale --enable-avfilter --enable-libmp3lame --enable-libvorbis --enable-libtheora --enable-libfaac --enable-libxvid --enable-libx264 --enable-libvpx --enable-libvo-amrwbenc --enable-shared --enable-pthreads --disable-indevs && make -j 4 && sudo make install

# FFMpeg creates MP4s that have the metadata at the end of the file.
# This tool moves it to the beginning.
cd tools
gcc -D_LARGEFILE_SOURCE qt-faststart.c -o qt-faststart
sudo mv qt-faststart /usr/local/bin

</pre>

## 下载链接

[Mac OS X编译MP4Box 下载](down/MP4Box_mac.tar.gz)

[Mac OS X编译ffmpeg 下载](down/ffmpeg_mac.tar.gz)

[Linux编译ffmpeg\mencoder\MP4Box\etc 下载](down/ffmpeg_mencode_mp4box_i386_linux.tar.gz)

参考链接：http://hunterford.me/compiling-ffmpeg-on-mac-os-x/