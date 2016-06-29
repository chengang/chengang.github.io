---
id: 4279
title: 【 Perl 】控制Firefox实现自动化Web测试
date: 2014-08-17T17:35:56+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4279
permalink: /4279.html
categories:
  - 备忘
tags:
  - Firefox
  - mozrepl
  - perl
  - WWW::Mechanize::Firefox
  - 自动化Firefox
  - 自动化测试
---
## 安装

1、安装Firefox插件

<pre class="brush: bash">git clone git://github.com/bard/mozrepl
cd mozrepl
zip -r ../mozrepl.zip *
cd ..
mv mozrepl.zip mozrepl.xpi
</pre>

2、安装Perl模块

<pre class="brush: bash">cpanm WWW::Mechanize::Firefox
cpanm HTML::Display::Common
</pre>

3、启动插件
  
先打开Firefox；
  
然后Firefox -> 工具 -> MozRepl -> Start

4、开始写程序控制你的Firefox吧

## 一个简单的例子

以打开新浪首页为例。

<pre class="brush: perl">use WWW::Mechanize::Firefox;
my $mech = WWW::Mechanize::Firefox->new( autoclose => 0, activate => 1 );
$mech->get('http://sina.com.cn');
</pre>

## 一个稍复杂的例子

让Firefox自动打开百度首页
  
1、在终端上输出页面title；
  
2、在浏览器中搜索一个词&#8221;perl&#8221;。

<pre class="brush: perl">use strict;
use warnings;

use WWW::Mechanize::Firefox;
binmode STDOUT, ':utf8';

my $mech = WWW::Mechanize::Firefox->new( autoclose => 0, activate => 1 );
$mech->get('http://baidu.com');
print $mech->title();
$mech->form_name( "f" );
$mech->set_fields( "wd", "perl" );
$mech->submit;

</pre>

效果如这个视频所示

<embed src="http://player.youku.com/player.php/sid/XNzU3MTY2NTg4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash">
</embed>

v.youku.com/v\_show/id\_XNzU3MTY2NTg4.html

## 抽取某条微博的全部评论

<pre class="brush: perl">#!/usr/bin/perl
use strict;
use warnings;

use WWW::Mechanize::Firefox;
use utf8::all;
use File::Slurp;
use JSON::XS;
use Encode;
use URL::Encode qw(url_decode_utf8 url_decode);
use Data::Dump qw(ddx);

my $mech = WWW::Mechanize::Firefox->new( autoclose => 1, activate => 1 );
my $url = 'http://weibo.com/aj/comment/big?_wv=5&id=3744295209134458&max_id=3744609358765392&filter=0&__rnd=1408272337687';
for ( 1 .. 42)
{
	$mech->get( "$url&page=$_" );
	my $a = encode( 'utf8', $mech->content( raw => 1 ));
	$a =~ s/.*?<prre>//g;
	$a =~ s/<\/prre><\/body><\/html>//g;
	#print $a;

	my %b = %{decode_json $a};
	my $c = $b{'data'}{'html'};
	$c =~ s/\n//smg;
	$c =~ s/\s+/ /smg;
	$c =~ s/>/>/smg;
	$c =~ s/</</smg;
	$c =~ s/&/&#038;/smg;
	#print "[$c]\n";

	my @abc = $c =~ /(usercard="id=\d+">.*?<\/a>：.*?<span class="S_txt2">)/smg;
	foreach (@abc)
	{
		my ($name, $words) = $_ =~ /usercard="id=\d+">(.*?)<\/a>：(.*?)<span class="S_txt2">/;
		$name =~ s/<\/a>//g;
		$name =~ s/<a.*>//g;
		$words =~ s/<\/a>//g;
		$words =~ s/<a.*>//g;
		$words =~ s/<img.*>//g;
		print "$name => $words\n";
	}
}

</pre>


<h2>
  相关文档
</h2>


<p>
  WWW::Mechanize https://metacpan.org/pod/WWW::Mechanize<br />
  WWW::Mechanize::Firefox https://metacpan.org/pod/WWW::Mechanize::Firefox<br />
  WWW::Mechanize::Firefox::Examples https://metacpan.org/pod/WWW::Mechanize::Firefox::Examples<br />
  WWW::Mechanize::Firefox::Cookbook https://metacpan.org/pod/distribution/WWW-Mechanize-Firefox/lib/WWW/Mechanize/Firefox/Cookbook.pod
</p>


<p>
  和QTP、WinRunner、Selenium相比看看，Perl确实让这件事变得简单些了。
</p>