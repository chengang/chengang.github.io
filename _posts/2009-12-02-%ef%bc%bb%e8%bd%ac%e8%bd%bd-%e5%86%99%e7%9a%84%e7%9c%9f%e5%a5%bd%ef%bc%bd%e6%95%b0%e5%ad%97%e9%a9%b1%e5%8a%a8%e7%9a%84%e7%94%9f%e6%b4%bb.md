---
id: 1489
title: ubuntu9.10下qq崩溃的解决
date: 2009-12-02T18:11:32+00:00
author: chen
layout: post
guid: http://blog.chengang.cc/?p=1489
permalink: /1489.html
categories:
  - 转载
tags:
  - linux
  - qq
  - ubuntu
---
在/usr/bin/qq里最开始的地方增加一行

<pre class="brush: bash">export GDK_NATIVE_WINDOWS=true
</pre>

这样就不会崩溃了。

这是因为9.10中采用的GDK的新特性&#8217;client-side windows&#8217;与qq不兼容造成的。
  
把GDK\_NATIVE\_WINDOWS标志位设上，就关闭了这个特性。然后就好了，烦扰你的linux下qq崩溃的问题就解决了。