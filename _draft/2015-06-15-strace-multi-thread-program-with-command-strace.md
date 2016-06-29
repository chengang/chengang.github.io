---
title: strace追踪多线程程序
date: 2015-06-15T16:41:19+00:00
layout: post
---
## 方法一

先用ps -mp pid或者top -H查出线程pid。

然后strace -p pid追踪其中一个线程。

## 方法二

直接用strace -fp pid追踪进程下所有线程。
