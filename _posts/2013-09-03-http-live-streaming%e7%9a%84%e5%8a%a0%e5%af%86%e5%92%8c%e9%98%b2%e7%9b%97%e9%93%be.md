---
title: HTTP Live Streaming的加密和防盗链备忘
date: 2013-09-03T18:09:04+00:00

---
1、大意就是用EXT-X-KEY来加密文件碎片，然后动态地改变EXT-X-KEY，并且在EXT-X-KEY上加权限控制；
  
2、iOS中加密使用AES-128 encryption using 16-octet keys的方式；
  
3、有三种传递EXT-X-KEY的方式：指定本地文件、指定一个地方放EXT-X-KEY所有文件共用、每n片使用一个EXT-X-KEY。

[Overview相关章节点此](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW15)

FAQ15：Does the Apple implementation of HTTP Live Streaming support DRM?
  
No. However, media can be encrypted, and key access can be limited by requiring authentication when the client retrieves the key from your HTTPS server.

[标准文档相关章节点此](http://tools.ietf.org/html/draft-pantos-http-live-streaming-11#section-3.4.4)

8.4. Playlist file with encrypted media segments

<pre>#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:7794
#EXT-X-TARGETDURATION:15

#EXT-X-KEY:METHOD=AES-128,URI="https://priv.example.com/key.php?r=52"

#EXTINF:2.833,
http://media.example.com/fileSequence52-A.ts
#EXTINF:15.0,
http://media.example.com/fileSequence52-B.ts
#EXTINF:13.333,
http://media.example.com/fileSequence52-C.ts

#EXT-X-KEY:METHOD=AES-128,URI="https://priv.example.com/key.php?r=53"

#EXTINF:15.0,
http://media.example.com/fileSequence53-A.ts
</pre>
