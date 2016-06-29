---
title: DASH标准阅读笔记
date: 2015-05-06T11:38:49+00:00
layout: post
---
## 功能上和HLS相比的强处

1、支持inband事件流；
  
2、切片可以用模版描述，避免轮询索引；

## 功能上和HLS相比的弱处

1、没有冗余线路的支持；

## 3个无感切流的实践

1、关键帧；
  
2、时间上不相互重叠的切片；
  
3、切片拼装起来后和原始流数据一致（注意时间戳）；

## 关于MPD

1、MPD是XML；
  
2、MPD支持GZIP压缩；
  
3、加密和完整性内容保护都不是DASH标准的一部分，但是可以通过XML的W3C标准中相关内容来实现；

## 数据分层概览

1、Period -> Adaptation Sets -> Representations -> Sub-Representations-> Segments -> Sub-Segments；
  
2、Adaptation Set、Representation、Segment都可以指定码率、sar等视频相关的属性，按照级别一级一级覆盖，低级优先；

## Period

是数据分层中最高的一层，内含一段时间的内容；

## 先发Period

先发Period是Period的一种，用于表示流的一些信息，类似头信息。
  
同时满足以下条件的Period就是“先发Period”。
  
1、直播；
  
2、不是MPD里的第一片；
  
3、不包含@start属性；
  
4、前一片没有@Duration属性；

## Adaptation Set

1、是Period的下一级；
  
2、是Representation的上一级；
  
3、包含多个可以无感切换的Representation；
  
4、可以声明下面所有Repersentation的码率总范围；
  
5、可以声明关键帧是否是对齐的；

## Media Content Compontent

1、是Adaptation Set的下一级；
  
2、如果一个Adaptation Set中只包含一个Media Content Compontent，可以不单独出现这个元素，直接在Adaptation Set的属性中声明；
  
3、可以指定par、lang、contentType、role、viewpoint等属性；

## Representation

1、是Adaptation Set的下一级；
  
2、是Segment的上一级；
  
3、时长与Period相同；
  
4、可以是自启动的，也可以通过指定@dependencyId属性靠依赖启动；
  
5、直播时的时钟补偿操作在这一层做；
  
6、Segmentlist属性只能指定一个；
  
7、要声明自己的@qualituRanking；

## Sub-Representations

1、用于音频、字幕、低码率视频；
  
2、提供同内容的低质量流，比如说只包含声音或者文字的；
  
3、是包含在Repersentation之中的，而不是与之平级；
  
4、如果指定了@level，就必须指定@bandwidth；

## Subsets

1、是Period的下一级；
  
2、负责聚集多个Adaptation Set；
  
3、不可以用一个Subsets聚集所有的Adaptation Sets；
  
4、主要作用就是聚集Adaptation Sets；

## Segments

1、可以指定这个片的“可见时间”，也就是不会404的时间；
  
2、三种指定Segment Url的方法：
    
a> 1个SegmentList；
    
b> 1个SegmentTemplate；
    
c> 1个SegmentBase加多个BaseUrl；
  
3、1个representation只包含1个Segment时才用到SegmentBase；
  
4、1个representation包含多个Segment时用SegmentList或者SegmentTemplate；
  
5、多个Segment时必须指定@Duration属性或者SegmentTimeline元素其中之一，但不能都指定；
  
6、SegmentURL支持指定HTTP Range；
  
7、可以通过SegmentList.href关联远端SegmentList；
  
8、SegmentTemplate模版使用示例： %05d。printf的格式都支持，还可以关联representation中的@Id和@bandwidth等属性；
  
9、多个Segment时，以下4项可以帮助定位：
    
a> URL+HTTP Range；
    
b> representation中的序号；
    
c> 开始时间；
    
d> 时长；
  
10、模版中$time变量是指之前所有片的@Duration之和；
  
11、SegmentUrl尝试解析顺序如下：
    
a> SegmentTemplate + Number；
    
b> SegmentTemplate + Time；
    
c> SegmentList；
    
d> SingleSegment；
  
12、Segment的“可见时间”可以等于：
    
a> representation的结束时间；
    
b> MPD的“可见时间” —— Period.@miniumUpdate属性；
