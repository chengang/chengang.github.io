---
id: 1973
title: 【 MySQL 】id列报Duplicate entry错误
date: 2010-08-15T23:29:21+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=1973
permalink: /1973.html
categories:
  - 实践
tags:
  - Duplicate entry
  - mysql
---
我终于查出这个错误了！

我们有个表一直报Duplicate entry错误，在日志里写了好多好多条。

程序查了没查出问题，打印的SQL没问题，数据写入也没问题，我查了好久都没查出来，今天终于查到原因了。

原因是：这一列是指定了auto\_increase的列，于是就会在报Duplicate entry的同时又正确写入数据（如果重复的话，写入的不是指定的值，而是auto\_increase的值）。