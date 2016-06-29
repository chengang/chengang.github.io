---
id: 4555
title: DarwinStreamingServer搭建RTSP服务器
date: 2015-02-10T21:52:53+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4555
permalink: /4555.html
categories:
  - 备忘
tags:
  - DarwinStreamingServer
  - dss
  - rtsp
---
## 安装

<pre>wget http://blog.yikuyiku.com/down/DarwinStreamingServer-6.0.3-4.x86_64.rpm
md5sum DarwinStreamingServer-6.0.3-4.x86_64.rpm #24a8ba7c428106fa4da0ec0c37842d1d
yum install perl-Net-SSLeay
rpm -ivh DarwinStreamingServer-6.0.3-4.x86_64.rpm
</pre>

## 修改配置

<pre>cp /etc/dss/streamingserver.xml /etc/dss/streamingserver.xml.bak
vim /etc/dss/streamingserver.xml
diff /etc/dss/streamingserver.xml /etc/dss/streamingserver.xml.bak
77c77
&lt; 		&lt;PREF NAME="authentication_scheme" >none&lt;/PREF>
---
> 		&lt;PREF NAME="authentication_scheme" >digest&lt;/PREF>
164c164
&lt; 		&lt;PREF NAME="allow_invalid_hint_track_refs" TYPE="Bool16" >true&lt;/PREF>
---
> 		&lt;PREF NAME="allow_invalid_hint_track_refs" TYPE="Bool16" >false&lt;/PREF>
199c199
&lt; 		&lt;PREF NAME="ip_allow_list" >*.*.*.*&lt;/PREF>
---
> 		&lt;PREF NAME="ip_allow_list" >127.0.0.*&lt;/PREF>
</pre>

## 重启

<pre>/etc/init.d/dss restart
</pre>

## 增加推流权限

新增文件/var/dss/movies/qtaccess内容如下：

<pre>&lt;Limit WRITE>  
require any-user  
&lt;/Limit>  
require any-user  
</pre>

## FFmpeg推流命令

<pre>ffmpeg -re -i 360x480.mp4 -vcodec copy -acodec copy -f rtsp -y rtsp://192.168.0.1/top.sdp
</pre>

## 点播

点播文件默认放在/var/dss/movies/目录下。
  
要带有mp4hint才可以播放，不然就报错。