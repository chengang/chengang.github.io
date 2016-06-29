---
id: 4402
title: phantomjs抓取ajax页面一例
date: 2014-10-01T08:25:11+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4402
permalink: /4402.html
categories:
  - 备忘
tags:
  - phantomjs
---
<pre># file: example.coffee
# usage: phantomjs example.coffee
page = require('webpage').create()

console.log 'The default user agent is ' + page.settings.userAgent

page.settings.userAgent = 'Mozilla/5.0 (iPad; CPU OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53'
page.open 'http://www.sina.com.cn/', (status) ->
  if status isnt 'success'
    console.log 'Unable to access network'
  else
    console.log page.content

phantom.exit()
</pre>