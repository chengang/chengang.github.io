---
id: 2885
title: fanotify与inotify的区别
date: 2012-01-04T18:13:41+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2885
permalink: /2885.html
categories:
  - 备忘
tags:
  - fanotify
  - inotify
---
fanotify在2.6.36版本（2010-10-21）并入Linux内核（同时增加了CIFS本地缓存），它与原有的inotify的区别在于以下：

**1、inotify最麻烦的一点就是不能监控子目录**，要自己去弄n多个监控。fanotify也不能自动监控子目录，但它有一个Global模式，可以监控整个文件系统的所有事件。这样就可以监控所有事件，然后在程序里判断，这样也算是个比之前好一些的解决方案；

**2、inotify只能接受事件**。fanotify不仅可以接受事件，还可以阻塞事件，甚至进一步阻止事件执行成功。并且可以持续地给这个文件设置上acl，且不用每次触发都判断一次（省去麻烦，增加性能）；

**3、fanotify允许多个程序同时监控同一个文件系统对象**，并且可以设置优先级（消息到达前后）。这个机制可以处理通过fanotify机制本身做出的动态文件；

**4、fanotify可以指定不监控某些pid对文件的操作**，这是inotify做不到的。使用场景例子：可以让监控程序自己不会触发事件，避免了让程序员自己去处理死循环的麻烦；

**5、fanotify相对于inotify的致命缺陷**：fanotify可以触发的事件比inotify少，尤其是缺乏MOVE、ATTRIB、CREATE、DELETE事件（有ACCESS、MODIFY、CLOSE）。