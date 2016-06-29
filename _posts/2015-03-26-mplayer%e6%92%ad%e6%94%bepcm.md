---
id: 4606
title: mplayer播放pcm
date: 2015-03-26T09:34:59+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4606
permalink: /4606.html
categories:
  - 备忘
tags:
  - mplayer
  - pcm
---
<pre>mplayer -rawaudio samplesize=2:channels=2:rate=44100 -demuxer rawaudio test_pcm_s16le.pcm
</pre>