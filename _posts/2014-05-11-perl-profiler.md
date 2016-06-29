---
id: 4138
title: Perl Profiler
date: 2014-05-11T11:01:20+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4138
permalink: /4138.html
categories:
  - 备忘
tags:
  - Devel::NYTProf
  - perl
  - profile
---
<pre class="brush: bash">cpanm Devel::NYTProf Devel::SizeMe
cpanm Devel::Cover DBI::Profile 

perl -d:NYTProf split_chinese_word.pl d.txt m.txt2
nytprofhtml nytprof.out
cd nytprof
open index.html
</pre>