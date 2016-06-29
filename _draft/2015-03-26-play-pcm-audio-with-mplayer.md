---
title: mplayer播放pcm
date: 2015-03-26T09:34:59+00:00
layout: post
---
<pre>mplayer -rawaudio samplesize=2:channels=2:rate=44100 -demuxer rawaudio test_pcm_s16le.pcm
</pre>
