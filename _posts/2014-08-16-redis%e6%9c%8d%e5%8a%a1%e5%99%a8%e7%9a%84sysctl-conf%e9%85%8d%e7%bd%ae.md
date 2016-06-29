---
title: Redis服务器的sysctl.conf配置
date: 2014-08-16T07:52:09+00:00
layout: post
---
## 调整思路

  1. 减少Linux系统自身，特别是IO系统对内存的使用，避免在内存上与Redis冲突；
  2. Redis的Dump动作是一个大型IO动作，为避免Dump冲击系统，将其触发的尖峰写操作适当削平成一个持续的写操作；
  3. 尽量让tcp栈保持干净；

## 参数选择理由

#0 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
  
#1 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
  
#2 表示内核允许分配超过所有物理内存和交换空间总和的内存
  
#我们设置为1，表示尽量给Redis内存，但不要把硬盘当内存分配给它。
  
#这是redis官方要求必须配置的选项。

vm.overcommit_memory = 1

#这个参数控制文件系统的文件系统写缓冲区的大小，单位是百分比，表示系统内存的百分比，表示当写缓冲使用到系统内存多少的时候，开始向磁盘写出数据。增大之会使用更多系统内存用于磁盘写缓冲，也可以极大提高系统的写性能。但是，当你需要持续、恒定的写入场合时，应该降低其数值，：

vm.dirty_ratio = 1

#这个参数控制文件系统的pdflush进程，在何时刷新磁盘。单位是百分比，表示系统内存的百分比，意思是当写缓冲使用到系统内存多少的时候，pdflush开始向磁盘写出数据。增大之会使用更多系统内存用于磁盘写缓冲，也可以极大提高系统的写性能。但是，当你需要持续、恒定的写入场合时，应该降低其数值，

vm.dirty\_background\_ratio = 1

#这个参数控制内核的脏数据刷新进程pdflush的运行间隔。单位是 1/100 秒。缺省数值是500，也就是 5 秒。如果你的系统是持续地写入动作，那么实际上还是降低这个数值比较好，这样可以把尖峰的写操作削平成多次写操作。设置方法如下

vm.dirty\_writeback\_centisecs = 100

#这个参数声明Linux内核写缓冲区里面的数据多“旧”了之后，pdflush进程就开始考虑写到磁盘中去。单位是 1/100秒。缺省是 30000，也就是 30 秒的数据就算旧了，将会刷新磁盘。对于特别重载的写操作来说，这个值适当缩小也是好的，但也不能缩小太多，因为缩小太多也会导致IO提高太快。

vm.dirty\_expire\_centisecs = 100

#该文件表示系统进行交换行为的程度，数值（0-100）越高，越可能发生磁盘交换。
  
#我们设置为0，表示不要进行swap

vm.swappiness = 10

#该文件表示内核回收用于directory和inode cache内存的倾向；缺省值100表示内核将根据pagecache和swapcache，把directory和inode cache保持在一个合理的百分比；降低该值低于100，将导致内核倾向于保留directory和inode cache；增加该值超过100，将导致内核倾向于回收directory和inode cache

vm.vfs\_cache\_pressure = 110

#1 To free pagecache:
  
#2 To free reclaimable slab objects (includes dentries and inodes):
  
#3 To free slab objects and pagecache:
  
#经过实验，此选项只在设置那一次会生效释放掉内存，此后IO占用的内存还会持续增长
  
#且再次执行sync也不释放，只有再次设置此选项才会再次释放，因此我们选择不设置此项

#vm.drop_caches = 3

#64位系统所有内存都被划分为lowmem，无需此项配置

#vm.lowmem\_reserve\_ratio = 32 32 8

#FreeBSD参数

#kern.maxvnodes = 3

#zone\_reclaim\_mode在内核默认就是禁用的。
  
#mongodb明确要求在NUMA系统上禁用此选项
  
#内核文档中说到如果打开此选项，NUMA节点在访问远端内存时会遭遇严重的性能问题。要求只在完全可以handle数据（cpu cache）位置时才调整此选项

vm.zone\_reclaim\_mode = 0

## 最终使用的配置

以下是一个64位的Linux-Redis服务器的最终sysctl.conf优化配置。
  
包括了 [百次掉坑一朝总结之net.ipv4参数](http://blog.yikuyiku.com/?p=4252 "百次掉坑一朝总结之net.ipv4参数") 中介绍的关于网络连接的优化配置。

<pre class="brush: bash">#maximize the available memory
vm.overcommit_memory = 1
vm.dirty_ratio = 1
vm.swappiness = 10
vm.vfs_cache_pressure = 110
vm.zone_reclaim_mode = 0

#keep the IO performance steady
vm.dirty_background_ratio = 1
vm.dirty_writeback_centisecs = 100
vm.dirty_expire_centisecs = 100

#keep the tcp stack clean
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 1024    65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
</pre>

## 参考文档

[kernel.org的sysctl官方文档](https://www.kernel.org/doc/Documentation/sysctl/ "sysctl官方文档") https://www.kernel.org/doc/Documentation/sysctl/

freebsd官方内核调整建议 http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/configtuning-kernel-limits.html

linux服务器lowmem不足引起系统崩溃的解决 http://blog.csdn.net/i3myself/article/details/8663801

淘宝内核维基 http://kernel.taobao.org/index.php/Kernel\_Documents/mm\_sysctl

磁盘读写参数设置 http://hi.baidu.com/roxws/item/4fb9fe2c368fdbd00e37f9e9

mongodb的NUMA问题 http://www.ttlsa.com/mongodb/mongodb-numa/
