---
title: '【 Perl 】三种方式解决&#8221; Wide character in subroutine/print &#8220;'
date: 2014-08-17T16:48:07+00:00

---
## 1、binmode STDOUT, &#8220;:utf8&#8221;;

因为程序本身是用utf8编码的（可以用use utf;明示给Perl）。
  
这句话就是告诉Perl输出是utf8编码的。

## 2、use utf8::all;

当然，我们需要先安装这个模块 [utf8::all](https://metacpan.org/pod/utf8::all "utf8::all") 。
  
一劳永逸，所有涉及字符集编码的地方，此模版都会帮你设置为utf8；

## 3、encode( &#8216;utf8&#8217;, $_ );

嗯。需要先use Encode;。
  
对CPU有不小的开销。
  
这种处理方法的好处在于可以用于print之外的其它方法出现的&#8221;Wide character in subroutine&#8221;问题。管得比较宽。

## 4、原因

宽字节这事，是从windows C++发祥的。
  
我理解是懒得认真处理字符集问题的一种偷懒方式。将多个字节绑定为单一结构体内，以期达到hack式修复的目的。
  
这种做法确实使得在VS上编程时不用关心字符集了。
  
效果嘛，只能说在windows内部自己玩的时候还算蛮不错……
