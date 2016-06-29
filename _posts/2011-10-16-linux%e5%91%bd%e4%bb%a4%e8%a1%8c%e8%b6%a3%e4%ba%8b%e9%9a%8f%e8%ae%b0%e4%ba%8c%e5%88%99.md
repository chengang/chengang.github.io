---
title: 啥ISO文件在mount的时候被称作loop设备
date: 2011-10-16T22:29:34+00:00
layout: post
---

## 为啥ISO文件在mount的时候被称作loop设备

突然间就对为什么loop设备叫“loop”发生了兴趣，查之。

a> “loop”这个名字的由来和127.0.0.1一样，重定向了一次回本地的缘故；
  
b> loop设备就是虚拟的设备类型，通过这种设备类型可以指定一个文件像一个块设备一样访问;
  
c> 挂载光盘和磁盘的镜像文件或者加密内容是最经常的用武之地;
  
d> NetBSD和OpenBSD上称作&#8221;virtual node device&#8221;（vnd）， Solaris上称作&#8221;loopback file interface&#8221;（lofi）；、

更多的信息可见http://en.wikipedia.org/wiki/Loop_device
