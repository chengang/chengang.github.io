---
title: Adobe家的HTTP Dynamic Streaming介绍
date: 2011-04-25T12:45:45+00:00
layout: post
---
和APPLE家的HTTP Live Streaming差不多，主要异同如下：

1、文件切片采用MP4的格式而非ts格式；
  
2、索引在APPLE家是foo.m3u8文件，Adobe家是manifest文件；
  
3、Adobe家除了支持APPLE家支持的H.264/AAC之外还支持VP6/MP3编码；
  
4、不同于APPLE家，内容保护通过Flash Access Server来实现；
  
5、通过Adobe AIR可达范围更广（Mac OS、Windows、Linux都可以），但目前显然过不了APPLE家 iOS的审核；
  
6、两家同样都支持点播和直播；
  
7、Adobe家提供了一个全套的解决框架“Open Source Media Framework”。
  
8、HTTP Dynamic Streaming作为Adobe自家RTMP的补充。它自己的优点就不提了。相较RTMP之下它拥有：更低的延时、更短的载入时间、动态缓冲和基于流的加密。

更多内容请看：http://www.adobe.com/products/httpdynamicstreaming/
