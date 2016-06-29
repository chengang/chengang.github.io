---
title: mencoder的三个小memo
date: 2011-04-15T14:10:32+00:00
layout: post
---
1、mencoder封装的MP4（-of lavf -lavfopts format=psp），它自己的文档中说是“在存在B帧的时候出错”。当然了，完全不用之最好；

2、滤镜dsize=-1可以达到sar永远为1:1的目的；

3、2pass编码时，如果第一趟没有加入音频编码，第二趟又有音频编码。那么第二趟在因为需要音视频同步而skip一些帧时，会出现问题（弄乱1pass时）的帧决策。
