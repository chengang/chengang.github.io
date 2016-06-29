---
title: Perl Profiler
date: 2014-05-11T11:01:20+00:00
layout: post
---
<pre class="brush: bash">cpanm Devel::NYTProf Devel::SizeMe
cpanm Devel::Cover DBI::Profile 

perl -d:NYTProf split_chinese_word.pl d.txt m.txt2
nytprofhtml nytprof.out
cd nytprof
open index.html
</pre>
