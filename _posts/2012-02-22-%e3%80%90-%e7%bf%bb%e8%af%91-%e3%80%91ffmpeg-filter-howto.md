---
id: 3023
title: 【 翻译 】FFmpeg filter HOWTO
date: 2012-02-22T15:52:28+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=3023
permalink: /3023.html
categories:
  - 实践
  - 翻译
tags:
  - ffmpeg
  - libavfilter
  - 滤镜
---
## 定义一个滤镜

**AVFilter**

所有我们写的滤镜都要用一个AVFilter结构体讲给ffmpeg听。 这个结构体里描述了ffmpeg从哪个方法进入我们的滤镜。 这个结构体在libavfilter/avfilter.h里如下定义：

<pre>typedef struct
{
    char *name;         ///&lt; 滤镜名称

    int priv_size;      ///&lt; 给滤镜分配的内存大小

    int (*init)(AVFilterContext *ctx, const char *args, void *opaque);
    void (*uninit)(AVFilterContext *ctx);

    int (*query_formats)(AVFilterContext *ctx);

    const AVFilterPad *inputs;  ///&lt; 一系列输入 NULL terminated list of inputs. NULL if none
    const AVFilterPad *outputs; ///&lt; 一系列输出 NULL terminated list of outputs. NULL if none
} AVFilter;
</pre>

“query_formats”方法用于设置可以接受的输入图像格式和输出的图像格式（用于滤镜链分辨哪些滤镜可以组合在一起用）。

**AVFilterPad**

这个滤镜用于描述滤镜的输入输出，在libavfilter/avfilter.h中定义如下：

<pre>typedef struct AVFilterPad
{
    char *name;
    int type;

    int min_perms;
    int rej_perms;

    void (*start_frame)(AVFilterLink *link, AVFilterPicRef *picref);
    AVFilterPicRef *(*get_video_buffer)(AVFilterLink *link, int perms);
    void (*end_frame)(AVFilterLink *link);
    void (*draw_slice)(AVFilterLink *link, int y, int height);

    int (*request_frame)(AVFilterLink *link);

    int (*config_props)(AVFilterLink *link);
} AVFilterPad;
</pre>

头文件里有十分具体的描述，这里大概解释如下：

输入输出pad都可以用的元素：
  
name pad的名字，所有的输入pad名字不能重复，所有的输出名字不能重复；
  
type 此元素目前只能为“AV\_PAD\_VIDEO”值
  
config_props 链接此pad的配置方法的函数指针

仅限输入pad使用的元素：
  
min_perms 接受输入需要的最小权限
  
rej_perms 不接受的输入权限
  
start_frame 一帧传入时引用的方法的函数指针
  
draw_slice 每个slice已经传入后引用的方法的函数指针
  
end_frame 一帧完整结束后引用的方法的函数指针
  
get\_video\_buffer 前一个滤镜调用，用以为一个图像请求内存

仅限输出pad使用的元素：
  
request_frame 请求滤镜输出一帧



## 图像缓冲

**引用计数**

滤镜系统使用引用计数。意味着存在一个buffer里面存放着图像数据，而所有的滤镜都各自保有一个指向这个buffer的引用。当每一个滤镜完事，它就释放自己的那个引用。这样当所有的引用都释放以后，滤镜系统就会自动把buffer释放掉。

**权限**

由于可能有多个滤镜都保有了buffer的指针，它们可能会同时操作buffer而造成冲突，因此ffmpeg引入了权限系统。
  
大多数情况下，当一个滤镜准备输出一帧时，它调用滤镜链上下一个滤镜的一个方法来请求一个buffer。这指定了它对这个buffer需要的最低权限，不过可能实际被赋予的权限可能比要求的要高。
  
在想要把buffer输出给另一个滤镜时，会新建一个新的指向这个图像的引用，可能是一个权限标记的子集。这个新的引用属于接受buffer的滤镜。
  
举例说：一个丢帧的滤镜在输出最后一帧时，他可能在输出之后还是想要保持一个指向图像的引用，以确保没有其它滤镜同时修改这个buffer。为了达到这个目的，他可能请给自己求AV\_PERM\_READ|AV\_PERM\_WRITE|AV\_PERM\_PRESERVE权限，然后在给予其它滤镜的引用里去掉AV\_PERM\_WRITE权限。

可用的权限有：
  
AV\_PERM\_READ 可以读取图像数据
  
AV\_PERM\_WRITE 可以写入图像数据
  
AV\_PERM\_PRESERVE 保证图像数据不会被其它滤镜修改，意味着不会有其它滤镜拿到AV\_PERM\_WRITE权限
  
AV\_PERM\_REUSE 滤镜可能往一段buffer多次输出，但图像数据不得切换到不同的输出
  
AV\_PERM\_REUSE2 滤镜可能往一段buffer多次输出，可能在不同的输出之间修改图像数据



## 滤镜链

滤镜的输入输出用“AVFilterLink”结构体和其它滤镜相连接：

<pre class="brush: cpp">typedef struct AVFilterLink
{
    AVFilterContext *src;       ///&lt; source filter
    unsigned int srcpad;        ///&lt; index of the output pad on the source filter

    AVFilterContext *dst;       ///&lt; dest filter
    unsigned int dstpad;        ///&lt; index of the input pad on the dest filter

    int w;                      ///&lt; agreed upon image width
    int h;                      ///&lt; agreed upon image height
    enum PixelFormat format;    ///&lt; agreed upon image colorspace

    AVFilterFormats *in_formats;    ///&lt; formats supported by source filter
    AVFilterFormats *out_formats;   ///&lt; formats supported by destination filter

    AVFilterPicRef *srcpic;

    AVFilterPicRef *cur_pic;
    AVFilterPicRef *outpic;
};
</pre>

成员“src”和“dst”分别指出滤镜在链上的输入和输出的结束。“srcpad”指向链条上一个“源滤镜”的输出面的索引；类似的，“dstpad”指向目标滤镜的输入面的索引。
  
成员“in\_formats”指向“源滤镜”定义的它支持的格式，“out\_formats”指向“目标滤镜”支持的格式。结构体“AVFilterFormats”用于存储支持格式的列表，它使用引用计数，跟踪它的引用（参见libavfilter/avfilter.h中关于AVFilterFormats结构体的注释，了解色度空间的协商机制是怎么工作的，以及为什么协商是必要的）。结果就是一个滤镜如果为它之前和之后的滤镜提供了指向相同支持格式的列表的指针，就意味着这个链条上的滤镜就只能使用相同的格式了。
  
两个滤镜相连时，它们需要在它们处理的图像数据的尺寸和图像格式上达成一致。达成一致后，这些会作为参数存储在link结构体中。
  
成员“srcpic”是滤镜系统内部使用的，不应该直接存取。
  
成员“cur\_pic”是给目标滤镜用的。当一个帧正在通过滤镜链时（开始于start\_frame()，结束于end_frame），这个成员包含了目标滤镜对此帧的引用。
  
成员“outpic”会在接下来一个小教程中详细介绍。



## 写一个简单的滤镜

**默认的滤镜入口点**

因为大多数滤镜都只有一个输入一个输出，且每接受一个帧只输出一个帧，ffmpeg的滤镜系统提供了一系列默认的切入点以简化这种滤镜的开发，以下是切入点和默认实现的作用：
  
request_frame() 从滤镜链中前一个滤镜那请求一个帧
  
query_formats() 设置所有面上都支持的格式列表，这样别人就要按照这个列表来。默认包含大多数的YUV和RGB／BGR格式
  
start\_frame() 请求一个buffer来保存输出帧。一个指向此buffer的引用存储在hook到滤镜的输出的link的“outpic”成员中。下一个滤镜的start\_frame()回调会被调用，传入一个此buffer的引用。
  
end\_frame() 调用下一个滤镜的end\_frame()回调函数。释放指向输出link的“outpic”成员的引用，如果那成员被设置了（比如说使用了默认的start\_frame()方法）。释放输入link的“cur\_pic”引用
  
get\_video\_buffer() 返回一个在要求的权限上加一个AV\_PERM\_READ权限的buffer
  
config_props() on output pad 把输出的图像尺寸设置成和输入一样

**“vf_negate”滤镜**

介绍了数据结构和回调函数，让我们来看一个真实的滤镜。vf_negate滤镜的效果是反转视频中的色彩。它就一个输入一个输出，并且每个输入帧都输出一个帧。非常典型，可以使用滤镜系统那些默认的回调实现。

首先，让我们看一眼在“libavfilter/vf_negate.c”文件最下面的“AVFilter”结构体：

{% raw %}
<pre>AVFilter avfilter_vf_negate =
    {
        .name      = "negate",
    
        .priv_size = sizeof(NegContext),
    
        .query_formats = query_formats,
    
        .inputs    = (AVFilterPad[]) {{ .name            = "default",
                                        .type            = AV_PAD_VIDEO,
                                        .draw_slice      = draw_slice,
                                        .config_props    = config_props,
                                        .min_perms       = AV_PERM_READ, },
                                      { .name = NULL}},
        .outputs   = (AVFilterPad[]) {{ .name            = "default",
                                        .type            = AV_PAD_VIDEO, },
                                      { .name = NULL}},
    };
    
</pre>
{% endraw %}

可以看到滤镜的名字是“negate”，需要sizeof(NegContext)字节的空间存储上下文。在input和output列表的最后，都有一个name设置为NULL的pad。可以看出这个滤镜确实只有一个输入一个输出。如果你仔细观察pad的定义，你会发现好多回调函数已经被指定好了。因为我们这个滤镜很简单，所以大多数保持默认的就可以。

让我们看看它自己定义的回调函数。

**query_formats()**

<pre class="brush: cpp">static int query_formats(AVFilterContext *ctx)
{
    avfilter_set_common_formats(ctx,
        avfilter_make_format_list(10,
                PIX_FMT_YUV444P,  PIX_FMT_YUV422P,  PIX_FMT_YUV420P,
                PIX_FMT_YUV411P,  PIX_FMT_YUV410P,
                PIX_FMT_YUVJ444P, PIX_FMT_YUVJ422P, PIX_FMT_YUVJ420P,
                PIX_FMT_YUV440P,  PIX_FMT_YUVJ440P));
    return 0;
}
</pre>

这个函数调用了avfilter\_make\_format\_list()。这个方法第一个函数指定后面要列举多少个格式，后面就把格式列出来。返回是一个包含指定格式的AVFilterFormats结构体。把这个结构体传给avfilter\_set\_common\_formats()方法把格式给设置上。如同你看到的，这个滤镜支持一堆YUV平面的色彩空间格式，包括JPEG YUV色彩空间（其中那些包含字母J的）。

config_props() on an input pad
  
input填充的config_props()负责验证是否支持输入pad的属性，也负责更新滤镜的属性上下文。
  
TODO: 快速解释一下YUV色彩空间，色读采样，YUV和JEPG YUV范围的不同。

让我们看看滤镜是怎么存储它的上下文的：

<pre class="brush: cpp">typedef struct
{
    int offY, offUV;
    int hsub, vsub;
} NegContext;
</pre>

AVFilter结构体中的成员“priv\_size”告诉滤镜系统它需要多少字节来存储这个结构体。成员“hsub”和“vsub”用于色度采样，成员“offY”和“offUV”用于YUV和JPEG间范围的不同。让我们看看这些在输入pad的config\_props中是咋设置的：

<pre class="brush: cpp">static int config_props(AVFilterLink *link)
{
    NegContext *neg = link->dst->priv;

    avcodec_get_chroma_sub_sample(link->format, &neg->hsub, &neg->vsub);

    switch(link->format) {
    case PIX_FMT_YUVJ444P:
    case PIX_FMT_YUVJ422P:
    case PIX_FMT_YUVJ420P:
    case PIX_FMT_YUVJ440P:
        neg->offY  =
        neg->offUV = 0;
        break;
    default:
        neg->offY  = -4;
        neg->offUV = 1;
    }

    return 0;
}
</pre>

它只是简单调用了avcodec\_get\_chroma\_sub\_sample()方法去得到色度采样的位移因子，然后把它们存到上下文中。然后它存储了一些JPEG YUV的亮度／色度范围不同的偏移补偿。返回0表明成功，因为没有这个滤镜无法处理的输入格式。

**draw_slice()**

最后，滤镜中最重要的方法，它实际处理图像，draw_slice()：

<pre class="brush: cpp">static void draw_slice(AVFilterLink *link, int y, int h)
{
    NegContext *neg = link->dst->priv;
    AVFilterPicRef *in  = link->cur_pic;
    AVFilterPicRef *out = link->dst->outputs[0]->outpic;
    uint8_t *inrow, *outrow;
    int i, j, plane;

    /* luma plane */
    inrow  = in-> data[0] + y * in-> linesize[0];
    outrow = out->data[0] + y * out->linesize[0];
    for(i = 0; i &lt; h; i ++) {
        for(j = 0; j &lt; link->w; j ++)
            outrow[j] = 255 - inrow[j] + neg->offY;
        inrow  += in-> linesize[0];
        outrow += out->linesize[0];
    }

    /* chroma planes */
    for(plane = 1; plane &lt; 3; plane ++) {
        inrow  = in-> data[plane] + (y >> neg->vsub) * in-> linesize[plane];
        outrow = out->data[plane] + (y >> neg->vsub) * out->linesize[plane];

        for(i = 0; i &lt; h >> neg->vsub; i ++) {
            for(j = 0; j &lt; link->w >> neg->hsub; j ++)
                outrow[j] = 255 - inrow[j] + neg->offUV;
            inrow  += in-> linesize[plane];
            outrow += out->linesize[plane];
        }
    }

    avfilter_draw_slice(link->dst->outputs[0], y, h);
}
</pre>

“y”参数是当前slice的顶部，“h”参数是slice的高度。在这个区域以外的图像被假设为是无意义的（可能在未来的一些滤镜中这个假设会被打破）。

变量“inrow”指向输入slice的第一行，“outrow”指向输出的第一行。然后，它先遍历每一行，然后在每一行中遍历每一个像素，用255去减像素值，加上在config_props()中为不同格式的范围做出的修正值。

然后它在色度平面上做了同样的事。注意宽度和高度是调整到合适色度采样的。

在图像修改结束后，调用calling avfilter\_draw\_slice()方法把slice送给下一个滤镜去处理。

## 翻译自

http://wiki.multimedia.cx/index.php?title=FFmpeg\_filter\_HOWTO
