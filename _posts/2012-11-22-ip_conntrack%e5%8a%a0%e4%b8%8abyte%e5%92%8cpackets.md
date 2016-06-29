---
id: 3218
title: ip_conntrack加上byte和packets
date: 2012-11-22T13:10:25+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=3218
permalink: /3218.html
categories:
  - 备忘
tags:
  - ip_conntrack
---
sysctl -w net.netfilter.nf\_conntrack\_acct=1

http://stackoverflow.com/questions/12106535/tracking-the-network-bandwidth-used-by-a-process-and-its-children

待验证。