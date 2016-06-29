---
title: HAProxy、Nginx、Varnish分别实现简易CDN调度器
date: 2013-10-27T12:46:16+00:00

---
## 引子

**简单地说，CDN功能如下：**

1、将全网IP分为若干个IP段组，分组的依据通常是运营商或者地域，目的是让相同网络环境中的用户聚集到相同的组内；
  
2、依据CDN服务器们的网络和容量，确定哪些CDN服务器适合服务哪些IP段组；
  
3、根据以上两步得到的结论，让用户去最适合他的服务器得到服务。

说白了，就是根据用户不同的来源IP把用户请求重定向到不同的CDN服务器上去。
  
那么，如何实现呢？

**智能DNS是办法之一，稳定可靠且有效。
  
但至少在以下环境下它很难满足我们：**

1、需要特别精细的调度时。由于大多数DNS Server不支持DNS扩展协议，所以拿不到用户的真实IP，只能根据Local DNS来调度。
  
2、访问特别频繁时。由于每次调度都将触发一次DNS，如果请求变得密集，DNS请求本身带来的开销也会相应变大；
  
3、需要根据服务器的带宽容量、连接数、负载情况、当机与否来调度时。由于DNS Server没有CDN节点服务器的信息，这种调度会变得困难。

**这时候我们可以：**

1、将用户先行引导到某一台或几台统一的服务器上去；
  
2、让它拿到用户的真实IP，计算出服务他的服务器；
  
3、通过HTTP302或其它方式把用户定位到最终服务器上。

部署在用户先访问到的那几台服务器上，负责定位IP然后重定向用户请求的那个软件，我们叫它“调度器”。

## HAProxy实现

HAProxy不支持形如0.0.0.1-0.8.255.255 cn的IP段表示方法，只支持1.1.4.0/22 &#8220;CN&#8221;的IP段表示方法。

**1、我们需要先把IP段转化成它认识的方式；**

a> 下载[iprang.c](http://haproxy.1wt.eu/git?p=haproxy.git;a=blob_plain;f=contrib/iprange/iprange.c;hb=HEAD)或者[iprang.c本地镜像](http://blog.yikuyiku.com/down/iprang.c)；
  
b> 编译gcc -s -O3 -o iprange iprange.c；
  
c> 整理IP段列表geo.txt形如：

<pre class="brush: bash"># head geo.txt
"1.0.0.0","1.0.0.255","AU"
"1.0.1.0","1.0.3.255","CN"
"1.0.4.0","1.0.7.255","AU"
"1.0.8.0","1.0.15.255","CN"
"1.0.16.0","1.0.31.255","JP"
"1.0.32.0","1.0.63.255","CN"
"1.0.64.0","1.0.127.255","JP"
"1.0.128.0","1.0.255.255","TH"
"1.1.0.0","1.1.0.255","CN"
"1.1.1.0","1.1.1.255","AU"
</pre>

d> 输出HAProxy认识的IP段列表：

<pre class="brush: bash"># cut -d, -f1,2,5 geo.txt | ./iprange | head
1.0.0.0/24 "AU"
1.0.1.0/24 "CN"
1.0.2.0/23 "CN"
1.0.4.0/22 "AU"
1.0.8.0/21 "CN"
1.0.16.0/20 "JP"
1.0.32.0/19 "CN"
1.0.64.0/18 "JP"
1.0.128.0/17 "TH"
1.1.0.0/24 "CN"
1.1.1.0/24 "AU"
</pre>

e> 便于管理的目的，将整合后的IP段归类到同一个文件中：

<pre class="brush: bash"># cut -d, -f1,2,5 geo.txt | ./iprange | sed 's/"//g' | awk -F' ' '{ print $1 >> $2".subnets" }'
# ls *.subnets
A1.subnets  AX.subnets  BW.subnets  CX.subnets  FJ.subnets  GR.subnets  IR.subnets  LA.subnets  ML.subnets  NF.subnets  PR.subnets  SI.subnets  TK.subnets  VE.subnets
# cat AU.subnets 
1.0.0.0/24
1.0.4.0/22
1.1.1.0/24
</pre>

f> 把这些文件放到同一个文件夹下，我们以/etc/haproxy/conf/为例。

**2、正确配置HAProxy以这些IP段为规则正确调度；**

下面是一个haproxy.cfg的例子。配置好后重启Haproxy即可。

<pre class="brush: perl">global
    log         127.0.0.1 local2 debug

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     8000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 8000

frontend  main *:5000
    acl geo_A1 src -f /etc/haproxy/conf/A1.subnets
    acl geo_AX src -f /etc/haproxy/conf/AX.subnets
    acl geo_BW src -f /etc/haproxy/conf/BW.subnets
    acl geo_CX src -f /etc/haproxy/conf/CX.subnets
    acl geo_FJ src -f /etc/haproxy/conf/FJ.subnets

    ...

    reqrep ^([^\ ]*)\ /(.*)\ HTTP    \1\ /\2&ipfrom=A1\ HTTP if geo_A1
    reqrep ^([^\ ]*)\ /(.*)\ HTTP    \1\ /\2&ipfrom=AX\ HTTP if geo_AX
    reqrep ^([^\ ]*)\ /(.*)\ HTTP    \1\ /\2&ipfrom=BW\ HTTP if geo_BW
    reqrep ^([^\ ]*)\ /(.*)\ HTTP    \1\ /\2&ipfrom=CX\ HTTP if geo_CX
    reqrep ^([^\ ]*)\ /(.*)\ HTTP    \1\ /\2&ipfrom=FJ\ HTTP if geo_FJ
    
    ...
    
    default_backend             static

backend static
    server      static 127.0.0.1:6081 check
</pre>

## Nginx和Varnish的实现

Nginx可以在核心模块HttpGeoModule（http://wiki.nginx.org/HttpGeoModule）的配合下实现调度：

<pre class="brush: perl">http{

...

geo $useriprang {
    ranges;
    default a;
    0.0.0.1-0.8.255.255 a;
    0.9.0.0-0.255.255.255   a;
    1.0.0.0-1.0.0.255   a;
    1.0.1.0-1.0.1.255   b;
    1.0.2.0-1.0.3.255   b;
    1.0.4.0-1.0.7.255   a;
    ...
    223.255.252.0-223.255.253.255   c;
    223.255.254.0-223.255.254.255   a;
    223.255.255.0-223.255.255.255   a;
}

upstream backend {
    server 127.0.0.1:81;
}

server {
    listen       80;
    client_max_body_size 10240m;

    location / {
        proxy_redirect off;
        proxy_pass http://backend$request_uri&useriprang=$useriprang;
        proxy_next_upstream http_502 http_504 error timeout invalid_header;
        proxy_cache cache_one;
        proxy_cache_key $host:$server_port$uri$is_args$args;
        expires  5s;
    }

}

...

}
</pre>

Varnish则有两个插件可以实现调度：

https://github.com/cosimo/varnish-geoip （Last updated: 28/05/2013）

https://github.com/meetup/varnish-geoip-plugin （Last updated: 2010）

## 性能问题

如上所述，使用Haproxy、Nginx、Varnish都能快速实现这个功能。

其中Nginx和Varnish使用了二分法在IP表中定位用户IP，而Haproxy是逐条过滤。

所以在IP分得较细，IP段组较多（归类后超过1000组）时，Haproxy会出现明显的性能衰减，其余两者没有这个问题。

## 其它

本文使用的软件版本如下：
  
HAProxy1.4.22，Nginx1.2.9，Varnish3.0.4。
  
HAProxy和Varnish都是目前的最新版本。

本文有参考http://blog.exceliance.fr/2012/07/02/use-geoip-database-within-haproxy/
