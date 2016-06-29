---
id: 3401
title: flash.net.NetStream.appendBytes()在chrome下直播爆音的问题解决
date: 2013-05-11T12:44:06+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=3401
permalink: /3401.html
categories:
  - 实践
tags:
  - appendBytes
  - as
  - chrome
  - cilacila
  - flash
  - HDS
  - noise
  - OSMF
---
去年底和[东东](http://weibo.com/iceash "http://weibo.com/iceash")、[耿星](http://weibo.com/u/1806697365)一起，研究了一下Adobe家[HDS](http://www.adobe.com/cn/products/hds-dynamic-streaming.html "HTTP Dynamic Streaming")的具体实现 [OSMF](http://www.opensourcemediaframework.com/ "http://www.opensourcemediaframework.com/")。利用其中的一个核心方法 [flash.net.NetStream.appendBytes()](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/net/NetStream.html#appendBytes() "flash.net.NetStream.appendBytes()")构建了我们自己的HTTP点直播播放框架。

但今年年初发现一个问题：新框架直播的时候有爆音。

经过一段时间的现象收集，问题总结出来是这样：
  
1、直播的时候，播放一段时间后出现爆音；
  
2、同时去听别的电脑，并不是每台电脑上都会出现爆音；
  
3、出现爆音的不同电脑之间，Mac的爆音现象比Windows明显；
  
4、出现爆音的同一台电脑上，Mac上Chrome的爆音现象比Safari明显，Windows上Chrome的爆音现象比IE明显；
  
5、如果不经过浏览器，本地打开swf来播放流，则无爆音。

首先，我们想到了浏览器问题。于是升级最新的浏览器和FP，无效。

然后，开始怀疑Ffmpeg的segment muxer，于是俺自己写了一个[Flv segmenter](https://github.com/chengang/flv_segmenter "flv segmenter")，无效。

接着，尝试看看播放框架本身是不是有什么问题。

我们原有的逻辑是这样：每个flv片都自带flv头以及AVC和AAC的第0帧，每次调用appendBytes()都调用一次appendBytesAction(RESET_BEGIN)。

现在我先用[gnu split](http://www.gnu.org/software/coreutils/manual/html_node/split-invocation.html "coreutils split")将一个大的flv文件按照300k一个切开，这样就只有第一个文件片有flv头和AVC的AAC的第0帧了。再构造对应的假直播接口，然后将播放框架的改成只有下载到第一片的时候才调用flash.net.NetStream.appendBytesAction(RESET_BEGIN)，而后所有的片都只调用flash.net.NetStream.appendBytes()。

此时我惊喜地发现，爆音现象解除了。

但出现了另一个问题，播放画面开始出现卡顿。于是继续查找卡顿的原因。

卡顿的原因很简单。appendBytes()方法自带了2个buffer。除开正常的IO buffer，还有一个FIFO的tag buffer。它最多存放一个flv tag，每拼装出一个完整的flv tag就将其推入IO buffer。这样它就可以负责保证虽然数据都是一片一片碎片拼装起来的，但IO buffer中都是完整的flv tag。

我之前使用gnu split切分出来的文件肯定不是按照tag的边缘切分的文件。经观察，在片间填充tag buffer的时候就会出现卡顿。

于是使用自己的flv segmenter严格按照tag边缘来切分文件。

卡顿消失！但爆音归来……

最后，我们就想到这爆音会不会跟buffer有点关系呢？

编写框架初期，我们为了让用户减少直播延迟，设置了[bufferTimeMax](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/net/NetStream.html#bufferTimeMax "bufferTimeMax")为20，作用是让buffer太满的用户加速播放它们buffer中的流，以追上最新直播点。是不是这个机制导致的爆音呢？

查看OSMF，它是将bufferTimeMax写死为0的。于是我们也将bufferTimeMax设为0。

爆音消失。

反推出几个结论：
  
1、之前部分电脑无法再现爆音问题，很有可能是因为它们的网速没有快到可以填满buffer的地步；
  
2、我们可能要使用跳过一些片的方式让网速不稳定用户减少直播的延迟；
  
3、OSMF里面还是有很多营养值得去吸取的。

这是次历时月余的查错。

现在终于解除了纠结，这感觉真好，真好。