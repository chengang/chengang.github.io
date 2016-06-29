---
title: wordpress中文标签页链接错误的解决
date: 2009-11-03T22:18:04+00:00
layout: post
---
转载自http://www.summerdiary.com/iis-wordpress-chinese-tags/

网站根目录的wp-includesclasses.php文件,找到153行和158行.这是我们要修改的地方,

<pre class="brush: php">$pathinfo = $_SERVER['PATH_INFO'];
$req_uri = $_SERVER['REQUEST_URI'];
</pre>

分别修改成以下程序:

<pre class="brush: php">$pathinfo = mb_convert_encoding($_SERVER['PATH_INFO'], “UTF-8“, “GBK“);
$req_uri = mb_convert_encoding($_SERVER['REQUEST_URI'], “UTF-8“, “GBK“);
</pre>

网上另外一个加叹号的方案没有抓到点子上，会导致新的标签不显示。
