---
title: 【 Perl 】Perl的accept不会阻塞住程序
date: 2010-08-30T00:10:49+00:00
layout: post
---
以前一直用

<pre class="brush: perl">while(accept)
</pre>

还以为它会阻塞住。其实不会，没有连接进来的时候，它会返回空。

所以还是需要判断——

<pre class="brush: perl">next unless accept
</pre>
