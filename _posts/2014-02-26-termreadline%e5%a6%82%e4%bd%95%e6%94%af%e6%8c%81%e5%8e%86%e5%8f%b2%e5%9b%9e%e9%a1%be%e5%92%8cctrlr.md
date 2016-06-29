---
title: Term::ReadLine如何支持历史回顾和ctrl+r
date: 2014-02-26T11:51:47+00:00
layout: post
---
利用Perl的[Term::ReadLine](https://metacpan.org/pod/Term::ReadLine "Term::ReadLine")可以造出和shell一样的交互式运行环境。

但默认使用的话可能会遭遇两个小麻烦。

## 麻烦一

不支持用上箭头键重复之前输入过的命令，也不支持ctrl+r进行历史搜索。
  
即使调用了$term->addhistory()也一样。

解决这个问题需要把原先的——

<pre class="brush: perl">use Term::ReadLine;
</pre>

显式写成

<pre class="brush: perl">use Term::ReadLine;
use Term::ReadLine::Perl;
</pre>

如果使用这个呢，为了保证在所有系统上的兼容性，最好再显示表明

<pre class="brush: perl">use Term::ReadKey;
</pre>

或者这样——

<pre class="brush: perl">use Term::ReadLine;
use Term::ReadLine::Gnu;
</pre>

$term->addhistory()可以不用。

## 麻烦二

提示符下面有一条横线。
  
我们可以在初始化完成Term::ReadLine后，这样去掉那画蛇添足的横线——

<pre class="brush: perl">$term->ornaments(0);
</pre>

另外，附赠一个[Term::UI](https://metacpan.org/pod/Term::UI "Term::UI")，相信对使用[Term::ReadLine](https://metacpan.org/pod/Term::ReadLine "Term::ReadLine")的人会有帮助。
