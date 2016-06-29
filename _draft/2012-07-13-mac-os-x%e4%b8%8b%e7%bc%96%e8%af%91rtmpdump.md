---
title: Mac OS X下编译rtmpdump
date: 2012-07-13T15:43:18+00:00
layout: post
---
rtmpdump版本：2.3
  
Mac OS X版本：10.7.4

1. 将文件/rtmpdump-2.3/librtmp/makefile中的-soname 改为 -dylib\_install\_name；
  
2. make && make install

另：
  
vlc移除rtmp模块——http://git.videolan.org/gitweb.cgi/vlc.git/?a=commit;h=7f9f610042a3a3302f443663bb0399c3c7c859cd
