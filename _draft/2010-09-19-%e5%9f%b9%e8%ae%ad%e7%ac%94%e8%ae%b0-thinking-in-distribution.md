---
id: 2003
title: '培训笔记 &#8211; Thinking in distribution'
date: 2010-09-19T10:50:56+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=2003
permalink: /2003.html
categories:
  - 备忘
tags:
  - 培训
---
1、什么是分布式？
  
冗余、分层、一致。

2、两将军问题 —— tcp协议可靠吗？

3、事务
  
分布式事务的不可能性（因为2将军）、两阶段提交、三阶段提交

4、一致性
  
拜占庭将军——4台机器里面最多1台坏人、向量时钟、PAXOS

5、快照
  
2阶段快照——先发送快照指令（等待同步），然后执行快照。