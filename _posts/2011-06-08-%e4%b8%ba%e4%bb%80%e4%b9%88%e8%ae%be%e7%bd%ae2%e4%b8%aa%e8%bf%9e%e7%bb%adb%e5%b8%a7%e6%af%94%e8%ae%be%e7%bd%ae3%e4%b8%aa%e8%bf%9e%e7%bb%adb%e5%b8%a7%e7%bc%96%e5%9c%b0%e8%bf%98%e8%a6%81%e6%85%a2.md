---
id: 2400
title: 为什么设置2个连续B帧比设置3个连续B帧编地还要慢？
date: 2011-06-08T11:34:52+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2400
permalink: /2400.html
categories:
  - 备忘
tags:
  - b-pyramid
  - x264
---
因为默认设置是这样——
  
设置3个及以上连续B帧时，Pyramid被动开启。而Pyramid 会增进转码速度，这样就造就了标题的结果。

翻译自：http://forum.doom9.org/showthread.php?t=161492