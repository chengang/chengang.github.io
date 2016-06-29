---
title: '[rust]unused_variables'
date: 2015-07-03T14:43:53+00:00
layout: post
---
有时候会出现类似这样的警告：

<pre>warning: unused variable: `req`, #[warn(unused_variables)] on by default
</pre>

有两个方法去掉这个警告

## 方法一

在方法前面声明#[allow(unused_assignments)]

## 方法二

在变量命名前加一个下划线。
