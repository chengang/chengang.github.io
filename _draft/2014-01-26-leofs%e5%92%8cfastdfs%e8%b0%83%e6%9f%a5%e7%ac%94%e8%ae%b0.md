---
title: LeoFS和FastDFS调查笔记
date: 2014-01-26T14:46:30+00:00
layout: post
---
## LeoFS

1、LeoFS中也把数据分为metadata和objectdata，但是它的metadata是p2p方式存储全网分发的；
  
2、通信全部采用标准协议。同步数据是iFCP，监控是SNMP，外部接口是HTTP；
  
3、Gateway带有object cache，里面包含metedata。使用LRU寻找热点、使用SlabAllocator控制内存，使用SkipGraph定位数据；
  
4、通过使用多个RING的方式支持多IDC部署；
  
5、失败检测是靠Gateway完成的；
  
6、Gateway使用HTTP协议的GET、PUT、DELETE、HEAD方法来操作数据；
  
7、Gateway采用S3接口；
  
8、metedata全部存储在内存之中；
  
9、objectdata使用AppendOnly来存储；
  
10、大文件会被分块，原始文件和切分后的文件都会有自己的metedata；
  
11、cache将会支持多级的热点；
  
12、龙存的程序也叫LeoFS；

## FastDFS

1、文件名是服务返回的，不可以自定义；
  
2、文件ID中除了包含group name和存储路径外，文件名中可以反解出如下几个字段：

a）文件创建时间（unix时间戳，32位整数）
      
b）文件大小
      
c）上传到的源storage server IP地址（32位整数）
      
d）文件crc32校验码
      
e）随机数（这个字段用来避免文件重名）；

3、同组的机器是全量复制，而且每个机器上会给同组的每一个机器起一个独立的同步文件的线程；
  
4、可以额外安装FastDHT来支持自定义文件名；
