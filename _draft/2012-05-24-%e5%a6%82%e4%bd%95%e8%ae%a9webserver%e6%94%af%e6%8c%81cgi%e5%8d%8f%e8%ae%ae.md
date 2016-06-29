---
title: 如何让WebServer支持CGI
date: 2012-05-24T18:16:16+00:00

---
想着让自己写的WebServer支持CGI协议，这样就不用总是给选用我的server的同学讲解怎么用怎么方便了。

其实我这个server的用法比CGI的语法简单多啦⋯⋯

说正事，WebServer是如何支持CGI的。



## 首先，一个什么样的程序可被称为CGI程序呢？

第一，它必须可以输出符合HTTP协议的Head和Body；

第二，它还必须可以解析HTTP请求。

所以作为WebServer，HTTP协议都不大用管了，那么要做的事情一下子就变少啦。



## 然后，WebServer到底是怎么与一个CGI程序沟通，最终完成用户请求呢？

大概要做以下几件事：

1、把各种要告诉CGI程序的环境变量（如：QUERY\_STRING、CONTENT\_LENGTH），设置到shell的环境变量中（Perl来说直接往%ENV里加就成），这样CGI程序就能通过读环境变量拿到它们。

2、环境变量弄好了以后，fork一个新进程来执行启动CGI程序。

3、如果来的请求有Body部分的话，现在从STDIN输入给CGI程序。

4、在STDOUT和STDERR等着收CGI程序吐出来的内容。别忘了wait结束了的CGI程序的pid。

5、把STDOUT的部分吐回给WebClient，如果吐出来的头不完整的话，要帮忙补全一下。

另：环境变量的说明在http://tools.ietf.org/html/rfc3875的4.1部分。

然后这个WebServer就算是支持CGI了。就可以用C、Perl、PHP或者任意能执行的语言给它写动态内容了。



## 顺便记一下FastCGI与CGI实现上的不同

我看来FastCGI就是一个CGI的网络升级版，一旦搬到了网络上，那自然天生就可以搞进程池和分布式等特性啦。

FastCGI和CGI的方式，server与后台程序沟通往复的内容是完全一样的。FastCGI就是把CGI方式沟通的内容都给封装到网络流里面了。没有方便的环境变量和STDIN了，那么自然也要加一些便于网络流读取的长度头啊sessionID啊什么的进去。

这就是FastCGI了。要是说CGI是一个让WebServer与后台程序沟通的小技巧，那么FastCGI就是一个让WebServer和后台程序沟通的网络协议。

那么接下去的事情就是实现这个协议的细节了。关于这些细节，除了可以上官网看，这篇文章讲得很好：
  
http://xiaoxia.org/2009/10/05/fastcgi-protocol-analysis/



## PSGI和WSGI

其它还有一些WebServer和后台程序沟通的方式，比如Perl的PSGI、Python的WSGI。这两个都跟语言绑得比较死，不能通用所有的语言。

因为它们的大意是这样，既然WebServer是拿Perl写的，那么WebServer起来的时候Perl的解释器肯定已经起起来了，那么就用它去解释后台的Perl脚本好了，不要再新起解释器进程了。



## mod\_perl和mod\_php为什么没有PSGI和WSGI那么高效？

越扯越远了，那么mod\_perl和mod\_php不也一样内置了perl和php的解释器么，为什么它就没有Perl的PSGI、Python的WSGI或者Ruby的Rack那么高的效率呢？

那是因为它内置了太多了解释器，每个进程都有一个。（吐槽一下：其实在mod_php下用oo形态的php有点蛋疼）

而PSGI和WSGI就起来了一个解释器，每一个脚本其实只是一个新的lib而已，都没有跨出这个解释器的进程。

那么相比多个孤立的解释器进程来说，这个掌控着全局的解释器它自然就多了很多小魔法可以做啦。
