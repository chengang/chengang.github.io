---
id: 1673
title: lighttpd正确禁止缓存文件
date: 2010-05-05T17:28:36+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=1673
permalink: /1673.html
categories:
  - 备忘
tags:
  - cache
  - lighttp
---
添加头CacheControl = &#8220;no-cache&#8221;（这是个http1.1协议中的头），结果造成ie无法正确访问swf文件了。

参见微软的建议http://support.microsoft.com/kb/234067/EN-US/，如下设置头：

<pre class="brush: perl">$HTTP["url"] =~ ".swf" {
        setenv.add-response-header  = ("Pragma" =>"no-cache","Expires" => "-1")
}
</pre>

一切正常了。

经测试，

<pre class="brush: perl">$HTTP["url"] =~ ".swf" {
        setenv.add-response-header  = ("CacheControl" =>"max-age=0")
}
</pre>

也是可以的。

附带个lighttpd的启动脚本

<pre class="brush: bash">ulimit -c unlimited
/data1/lighttpd//sbin/lighttpd -f /data1/lighttpd//etc/lighttpd.conf
</pre>

再附带个lighttpd的停止脚本

<pre class="brush: bash">killall lighttpd
</pre>