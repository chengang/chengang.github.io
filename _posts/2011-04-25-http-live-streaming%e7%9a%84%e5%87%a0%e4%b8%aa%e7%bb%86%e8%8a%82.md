---
id: 2318
title: HTTP Live Streaming的几个细节
date: 2011-04-25T12:13:13+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2318
permalink: /2318.html
categories:
  - 备忘
  - 实践
tags:
  - HTTP Live Streaming
  - Redundant Streams
---
## 如何做内容加密

HTTP Live Streaming支持在m3u8中指定一个key文件（目前支持16-octet 的AES-128加密），然后每个视频切段都使用这个key来加密。
  
可以所有切段共用一个key，也可以几个切段使用一个key，最细可以一个切段使用一个key。
  
然后把这些key文件加上验证功能，比如登陆才能读取到，这样就可以达到内容加密的效果了。
  
建议使用HTTPS来传输key文件。
  
注意：每个新key文件都会发起一个新的HTTP请求，因此每个切段一个key会大大加大服务器的连接数。
  


## cache时的2个注意事项

1、最好不要用HTTPS来传送视频文件和m3u8文件，因为这样很容易穿透cache服务器。
  
2、另外，cache服务器必须知道m3u8文件缓存的时间不能大于一个视频切段的长度。
  


## 如何做冗余

可以在m3u8中指定2个BANDWIDTH相同的外部地址，这样客户端会自己在其中一路不可达时切换到另一路。达到提高可用性的效果。
  
[Redundant Streams in Overview](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW22)
  


附APPLE官方HTTP Live Streaming的Overview地址：
  
http://developer.apple.com/library/ios/#documentation/networkinginternet/conceptual/streamingmediaguide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html