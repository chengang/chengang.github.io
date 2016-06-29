---
title: 欺骗Ipad让它播放High Profile的h264视频
date: 2010-09-20T17:07:06+00:00
layout: post
---
邓旭的发现，可以欺骗ipad播high profile的h.264视频，被发现的中文GUI可用软件下载地址如下：
  
http://sourceforge.net/projects/mkvavi2mp4/

貌似最先是发表在这里的：
  
http://bbs.weiphone.com/read-htm-tid-928805-page-1.html

大意是说，只用替换掉h264文件中的profile标志位即可骗过Ipad，我验证了一下，果真如此。

h264文件中profile的二进制定义如下：
  
Baseline：42E0xx
  
Main：4D40xx
  
High: 6400xx

更详细可见：
  
http://wiki.whatwg.org/wiki/Video\_type\_parameters#Video\_Codecs\_3
  
http://forum.doom9.org/showthread.php?t=132386

Perl实现如下（把视频改为Main3.0）：

<pre class="brush: perl">#! /usr/bin/env perl

my $h264file = shift;
local $/;
open my $fh,"&lt;",$h264file;
my $bin = &lt;$fh>;
my $h264_profile_level_flag = substr($bin, 4, 4);

$bin =~ s/$h264_profile_level_flag/x67x4dx40x1f/g;

close $fh;

open $fh,">",$h264file;
print $fh $bin;
close $fh;
</pre>

进一步搜索发现，这种超越的能力和ipad的硬件版本有关系。
