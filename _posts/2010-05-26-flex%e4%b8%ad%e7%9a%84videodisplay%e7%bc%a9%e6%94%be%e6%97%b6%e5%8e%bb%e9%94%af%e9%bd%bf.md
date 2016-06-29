---
id: 1738
title: Flex中的VideoDisplay缩放时去锯齿
date: 2010-05-26T15:48:03+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=1738
permalink: /1738.html
categories:
  - 备忘
tags:
  - flex
  - smoothing
  - VideoDisplay
  - 去锯齿
  - 视频缩放
---
flex中提供的视频播放组件VideoDisplay有一个很恶心的地方。当指定大小与视频大小不一致时画质会严重下降。其实这是因为flex没有开启flash中Smoothing属性的原因。

而且flex并没有给我们提供控制这个属性的接口。可以按照下面的方法扩展出开启Smoothing属性的视频播放组件。

1、建目录custom，在目录中新建文件SmoothVideoDisplay.as，内容如下：

<pre class="brush: as3">package custom {

	import mx.controls.VideoDisplay;
	import mx.core.mx_internal;
	import mx.events.FlexEvent;

	use namespace mx_internal;

	public class SmoothVideoDisplay extends VideoDisplay
	{
		public function SmoothVideoDisplay():void
		{
			super();
			addEventListener( FlexEvent.CREATION_COMPLETE, onCreationComplete );
		}

		private function onCreationComplete( e:FlexEvent):void
		{
			if (videoPlayer.smoothing != smoothing)
			videoPlayer.smoothing = smoothing;
		}

		private var _smoothing:Boolean = true;

		[Bindable]
		public function set smoothing( val:Boolean):void
		{
			if ( val == smoothing)
			return;
			_smoothing = val;

			if (videoPlayer)
			videoPlayer.smoothing = _smoothing;
		}

		public function get smoothing():Boolean
		{
			return _smoothing;
		}

	}
}
</pre>

2、如下使用这个新的SmoothVideoDisplay组件。

<pre class="brush: xml"><?xml version="1.0" encoding="utf-8"?>
&lt;mx:Application xmlns:mx="http://www.adobe.com/2006/mxml" xmlns:custom="custom.*">
        &lt;custom:SmoothVideoDisplay width="480" id="movPlay1" autoPlay="false"
          height="368" source="http://10.210.136.52/20100520/flv/dd_g.flv"/>
&lt;/mx:Application>
</pre>

3、再看看你的视频。