---
id: 4096
title: xcode5.1使用ffmpeg-lib
date: 2014-03-24T22:17:44+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4096
permalink: /4096.html
categories:
  - 备忘
tags:
  - ffmpeg
  - libavcodec
  - libavformat
  - xcode
---
1、编译ffmpeg；
   
configuration: &#8211;enable-ffplay &#8211;enable-nonfree &#8211;enable-gpl &#8211;enable-version3 &#8211;enable-postproc &#8211;enable-swscale &#8211;enable-avfilter &#8211;enable-libmp3lame &#8211;enable-libvorbis &#8211;enable-libtheora &#8211;enable-libfaac &#8211;enable-libxvid &#8211;enable-libx264 &#8211;enable-libvpx &#8211;enable-hardcoded-tables &#8211;enable-shared &#8211;enable-static &#8211;enable-runtime-cpudetect &#8211;enable-pthreads &#8211;disable-indevs &#8211;cc=clang

2、加一个编译选项，如下图红框；

3、将下图蓝框中的内容拖拽进xcode，下面那个蓝色的部分有的需要从TARGETS &#8211; General &#8211; Linked Frameworks and Libraries里面添加；

[<img src="http://blog.yikuyiku.com/wp-content/uploads/ffmpeg-xcode51.jpg" alt="ffmpeg-xcode51" width="1400" height="873" class="alignnone size-full wp-image-4102" />](http://blog.yikuyiku.com/wp-content/uploads/ffmpeg-xcode51.jpg)

这些文件拖拽之前都需要先拷贝到项目文件夹里面来。