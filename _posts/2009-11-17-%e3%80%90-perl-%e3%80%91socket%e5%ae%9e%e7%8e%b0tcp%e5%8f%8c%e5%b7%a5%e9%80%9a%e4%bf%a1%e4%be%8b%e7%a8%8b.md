---
id: 1468
title: 【 Perl 】socket实现TCP双工通信例程
date: 2009-11-17T17:47:37+00:00
author: chen
layout: post
guid: http://blog.chengang.cc/?p=1468
permalink: /1468.html
categories:
  - 实践
tags:
  - perl
  - server
  - socket
  - tcp
---
**Server端程序**，侦听9876端口。接受Client发送的第一个数据，加上一个字符串“back”和一个换行符后发回给Client。要注意的是recv函数在UDP里好用但是TCP里面并不好用，用sysread吧。代码如下：

<pre class="brush: perl">#!/usr/bin/perl
use Socket qw(:all);
my $address = sockaddr_in(9876,inet_aton('localhost'));
socket(SERVER, AF_INET, SOCK_STREAM, IPPROTO_TCP) || die "socket create: $!\n";
setsockopt(SERVER, SOL_SOCKET, SO_REUSEADDR, 1) || die "socket reuse: $!\n";
bind(SERVER, $address) || die "socket bind: $!\n";
listen(SERVER, 5) || die "socket listen: $!\n";
while (1)
{
    my $c = '';
    accept(SOCK, SERVER) || die "accept: $!\n";
    sysread(SOCK, $c, 1) or die "recv: $!\n";
    syswrite(SOCK, $c."back\n");
}</pre>

**Client端程序**，发送&#8217;gogo&#8217;和两个换行符给Server，然后把Server的回应打印出来：

<pre class="brush: perl">#!/usr/bin/perl
use Socket qw(:all);
$address = sockaddr_in(9876,inet_aton('localhost'));
socket(SOCK, AF_INET, SOCK_STREAM, IPPROTO_TCP) || die $!;
connect(SOCK, $address) || die $!;
syswrite(SOCK, "gogonn");
while(&lt;SOCK&gt;)
{
  print;
}
close SOCK or die "close: $!";</pre>

要说明的是，TCP是一个面向连接的协议。因此，如果想要Server端接收任意长度的数据，需要自己来处理——比如在开头的两个字节声明整个需要接收的数据长度——这是一个较为常用方法。