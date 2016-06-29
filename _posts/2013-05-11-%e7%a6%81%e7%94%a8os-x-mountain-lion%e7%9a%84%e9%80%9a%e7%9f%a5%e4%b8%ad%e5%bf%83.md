---
id: 3448
title: 禁用OS X Mountain Lion的通知中心
date: 2013-05-11T11:43:50+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=3448
permalink: /3448.html
categories:
  - 备忘
tags:
  - mac
  - macos
  - Notification Center
---
禁用：

<pre class="brush: bash">launchctl unload -w /System/Library/LaunchAgents/com.apple.notificationcenterui.plist
</pre>

启用：

<pre class="brush: bash">launchctl load -w /System/Library/LaunchAgents/com.apple.notificationcenterui.plist
</pre>