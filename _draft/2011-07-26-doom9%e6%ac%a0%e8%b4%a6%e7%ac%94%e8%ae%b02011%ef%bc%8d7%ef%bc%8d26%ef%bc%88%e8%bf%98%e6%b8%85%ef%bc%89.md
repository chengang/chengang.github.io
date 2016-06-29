---
title: doom9欠账笔记2011－7－26（还清！）
date: 2011-07-26T18:33:37+00:00
layout: post
---
**呼，终于把欠doom9的帐全都还完啦！~
  
** 

**http://forum.doom9.org/showthread.php?t=154406**

x264是怎么使用B帧的？
  
1、在pyramid改造之前，x264最多使用3个连续B帧；
  
2、有了新的pyramid之后，x264可以编出5-6个连续B帧了，最多6个；
  
3、越多的连续B帧可能越会影响到多线程带来的效率提升；
  
4、psy强力降低PSNR；

**http://forum.doom9.org/showthread.php?t=154628**
  
1、想要以质量为目的去噪，请x264 -nr 0 另请高明；
  
2、x264 -nr可以有效利用运动信息，在crf编码中，它可以降低输出码率（-nr 500）；
  
3、x264 -nr可以和预处理去噪一起用，效果还不赖；

**http://forum.doom9.org/showthread.php?t=109747**
  
x264的deblock小FAQ：
  
1、第一个值（Alpha deblocking）是去块范围，越大，去块越厉害，细节越少，视频也越模糊。一般来说默认值0就够了，不要超出+-2的范围；
  
2、第二个值（Beta deblocking）是去块阈值，越小，保留越多细节；越大，去块越猛，可以越好得去除明显的块效应；把这个值弄高点有利于去除各种错误；
  
3、总之，值越大画面越flat（1:2），值越小画面细节越多(-2:-1)；
  
4、一些建议值
  
Low- 0:3
  
Medium- 1:-1
  
High- 0:-3 

**http://forum.doom9.org/showthread.php?t=152419**
  
一个更改H.264流里sps的工具，目前可以更改sar, fps, crop information, level, ref frames, cfr/vfr flag, fullrange flag, colorprim, transfer and colormatrix。地址是https://direct264.svn.sourceforge.net/svnroot/direct264/Patches/

**http://forum.doom9.org/showthread.php?t=148686**
  
x264第一个引入mbtree的版本的声明（20090803）：
  
1、不和&#8211;b-pyramid共同工作；
  
2、速度影响不大，与&#8211;lookahead的值有线性关系；
  
3、70% SSIM 提升；
  
4、目的在于帮助帧决策，在低码率时尤为有效；

**http://forum.doom9.org/showthread.php?t=157278**
  
最新版的MP4Box去MeGUI里面也能找到，还有以下地方：http://komisar.gin.by/tools/、http://sada5.sakura.ne.jp/files/index.php?folder=TVA0Qm94。能帮助解决一些音画不同步的麻烦。
