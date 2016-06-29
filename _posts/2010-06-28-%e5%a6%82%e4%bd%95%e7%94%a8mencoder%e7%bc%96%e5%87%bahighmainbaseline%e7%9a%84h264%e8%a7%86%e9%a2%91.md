---
title: 如何用mencoder编出HighMainBaseline的H264视频
date: 2010-06-28T15:11:51+00:00
layout: post
---
从上往下判断：

1、开启8x8dct，则视频为High。
  
2、开启以下任意一项weightpcabacbframes，则视频为Main。
  
3、以上项皆关闭，则视频为Baseline。（no8x8dct:nocabac:weightp=0:bframes=0）

p.s. 旧版本的mencoder不支持weightp，所以不用设置它。
