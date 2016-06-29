---
title: Dropbox如何使用Python来开发Windows客户端
date: 2013-11-19T11:07:51+00:00
layout: post
---
Dropbox的第三位工程师Rian Hunter在PyCon 2011上做了一个分享，介绍了Python在整个Dropbox架构中的重要位置。

国外很卡，我把视频转到了国内：

<div>
</div>

分享中段的时候，他简短地提到了他们使用Python开发Windows客户端的情况。

事实上他们用同一套代码编译出了Windows和Mac的客户端，他们使用了下面这些库来帮助他们开发：

PyObjC —— 帮助Python调用外部ObjC类库，比如Cocoa；
  
WxPython —— 跨平台窗口库wxWindows的Python版；
  
ctypes —— 帮助Python调用Windows的COM API；
  
py2exe —— 将python程序和Python环境打包成windows平台的独立可执行程序；
  
py2app —— 将python程序和Python环境打包成Mac平台的独立可执行程序；
  
PyWin32 —— 帮助Python调用Windows的Win32 API和COM；

当然过程不是用了这些类库就一帆风顺的，这些类库肯定或多或少有它们的BUG，比如这里就有一个[Dropbox在使用ctypes时遇到的困难](http://blog.yikuyiku.com/?p=3899)。
