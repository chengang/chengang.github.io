---
id: 2060
title: HTTP Live Streaming的几点备忘
date: 2010-10-25T17:32:42+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2060
permalink: /2060.html
categories:
  - 实践
tags:
  - HTTP Live Streaming
  - 视频
---
1、更长的分段导致更长的延迟和更长的初始化时间，切换码率（只能在切换分段的时候切）也更慢；
  
2、更短的分段导致对m3u8文件更密集的请求，从而导致更多网络流量；
  
3、Apple推荐的分段时长为10秒；
  
4、MPEG-TS流有比通常文件更多的头信息，会导致文件整体码率明显上升。可以使用Apple家的分段软件来减少和压缩其中不必要的头；
  
5、Apple家文档说10s分段的话会有约30s时延；
  
6、m3u8中可阻止客户端缓存文件，否则客户端会为了提高seek效率而缓存文件。