---
id: 4745
title: OAuth和OpenID的区别
date: 2015-09-19T16:11:21+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4745
permalink: /4745.html
categories:
  - 备忘
  - 翻译
tags:
  - oauth
  - openid
---
OAuth和OpenID都是开放的Web第三方登录标准，都使用了HTTP重定向的方式，也都可以集中管理登录有效期。

它们不同于，OpenID的目标在于使你只用一次登录动作就可以登录多个网站。当你想要登录一个使用了OpenID服务的网站时，它会将你重定向至提供OpenID服务的网站，如果你已经登录过，会自动重定向回来，此时你已经是登录状态。

OAuth的目标则在于让你可以独立授权“消费者”网站是否可以读取“提供者”网站上的数据。

比如说，你想让一个打印网站帮你打印微博上的相片。它会把你重定向到微博，微博问你“允许它读取你的相片吗？”如果你允许，这个打印网站就把你微博上的相片都下载走了。

使用OAuth需要你登录“提供者”网站，至于如何“提供者”网站提供何种登录方式，完全与OAuth机制无关。可以是用户名密码，物理令牌或者是OpenID。

OpenID不关心数据的事。虽然它的扩展协议也会支持存储像你的名字、头像、电话号码等内容，并将它们提供给使用了OpenID服务的网站。但本质上，它只是为了让你少输入几次这种通用信息而已。它不会存储与应用相关的敏感信息。

OAuth提供商会向别人提供你应用相关的数据，如果你用一个网站请求你的微博OAuth授权，你应该准备好它可以拥有你微博的所有权限。

你给了人家OAuth权限，那效果就像你把用户名密码给人家了。OAuth的好处就在于你不用真的把用户名和密码给他，而且可以做一些权限范围的控制。

但OpenID就没有这个效果，即使你用了两个使用同一个OpenID服务的网站，它们互相之间也读取不了任何信息，做不了任何操作。OpenID比OAuth更加“粗粒度”一些。

译自：http://softwareas.com/oauth-openid-youre-barking-up-the-wrong-tree-if-you-think-theyre-the-same-thing/