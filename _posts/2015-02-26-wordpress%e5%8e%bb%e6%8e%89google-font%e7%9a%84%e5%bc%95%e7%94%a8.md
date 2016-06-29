---
id: 4587
title: WordPress去掉google font的引用
date: 2015-02-26T10:36:23+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4587
permalink: /4587.html
categories:
  - 备忘
tags:
  - google font
  - script-loader.php
  - wordpress
---
目前所有插件都不靠谱，手动来吧。

<pre>[root@localhost wp-includes]# diff script-loader.php script-loader.old.php
590c590
&lt; 	if ( 'off' !== _x( 'off', 'Open Sans font: on or off' ) ) {
---
> 	if ( 'off' !== _x( 'on', 'Open Sans font: on or off' ) ) {
</pre>

我的主题没有用到google font，如果主题使用了google font还要记得把主题里的引用也要去掉。