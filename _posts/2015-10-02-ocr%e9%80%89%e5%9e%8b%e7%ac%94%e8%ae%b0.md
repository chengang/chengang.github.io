---
id: 4763
title: OCR选型笔记
date: 2015-10-02T09:54:53+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4763
permalink: /4763.html
categories:
  - 备忘
tags:
  - ocr
  - opencv
  - tesseract
  - 机器学习
---
## OCR（按照识别率从低到高排序）

[Cuneiform for Linux](https://launchpad.net/cuneiform-linux) —— 本来是个Windows软件，这是Linux的移植，2011年4月已经停止维护。
  
[GNU Ocrad](https://www.gnu.org/software/ocrad/) —— 命令行工具。有JS移植，可用于前端。
  
[GOCR](http://jocr.sourceforge.net/) —— 命令行工具。有JS移植，可用于前端。
  
[Tesseract](https://github.com/tesseract-ocr/tesseract) —— 开源OCR引擎，也有命令行工具。HP开发Google接手。3.0之后支持训练。[Golang绑定](https://godoc.org/gopkg.in/GeertJohan/go.tesseract.v1)。[入门教程](http://www.joyofdata.de/blog/a-guide-on-ocr-with-tesseract-3-03/)。
  
[OCRopy](https://github.com/tmbdev/ocropy) —— 基于训练的OCR引擎，训练后可以达到比Tesseract更高的准确度，项目比Tesseract更年轻。包含一个叫做OCRopus的布局分析器。in Python。
  
[Microsoft OCR Library](https://blogs.windows.com/buildingapps/2014/09/18/microsoft-ocr-library-for-windows-runtime/) —— Windows8.1之后的版本内置OCR引擎，可用于桌面和WindowsPhone。
  
[Abbyy](http://www.abbyy.com/) —— 收费软件，有SDK，有Cloud版本。

## 预处理

[OpenCV](http://opencv.org/) —— 图像处理老大哥。OpenCV3中有Scene Text Detection值得一用。
  
[Libccv](http://libccv.org/) —— 现代图像处理库，被很多人推荐。实现了精选的若干个图像处理算法，干净容易移植。其中[Stroke Width Transfor](http://libccv.org/doc/doc-swt/)尤其有用。
  
[lswms](http://sourceforge.net/projects/lswms/) —— 分行检测。
  
[OCRopus](https://github.com/tmbdev/ocropy) —— 基于神经学习网络算法的布局分析库。[教程](http://www.danvk.org/2015/01/09/extracting-text-from-an-image-using-ocropus.html)。
  
[TiRG](http://sourceforge.net/projects/tirg/) —— 文字区域检测库，[效果演示](http://funkybee.narod.ru/)。
  
[unpaper](https://github.com/Flameeyes/unpaper) —— 检测文字和旋转，用的是[Hough transform](https://en.wikipedia.org/wiki/Hough_transform)算法。

## Scene Text Detection

[API](http://docs.opencv.org/3.0-beta/modules/text/doc/erfilter.html)，
  
[例子1](https://github.com/Itseez/opencv_contrib/blob/master/modules/text/samples/textdetection.cpp)，
  
[例子2](https://github.com/Itseez/opencv_contrib/blob/master/modules/text/samples/end_to_end_recognition.cpp)，
  
[Paper](http://cmp.felk.cvut.cz/~neumalu1/neumann-cvpr2012.pdf)，
  
[高层包装应用](https://github.com/tleyden/open-ocr/wiki/Stroke-Width-Transform)。

## 高层项目

[node-dv](https://github.com/creatale/node-dv) —— in Node.js，整合了OpenCV、Tesseract和一些其他项目。
  
[node-fv](https://github.com/creatale/node-fv) —— node-dv的更高层，用于证件识别。
  
[OpenOCR](https://github.com/tleyden/open-ocr) —— 包装了SWT、Tesseract、Docker、RabbitMQ，提供队列和HTTP访问服务。in Golang。
  
[openalpr](https://github.com/openalpr/openalpr) —— 包装了Tesseract和OpenCV，支持多系统build，支持Docker，有Python和Node.js绑定。

## 百度OCR

[API](http://apistore.baidu.com/apiworks/servicedetail/146.html)值得借鉴学习。

## 关于移动端

[tess-two](https://github.com/rmtheis/tess-two)，Tesseract的安卓移植，[教程](http://gaut.am/making-an-ocr-android-app-using-tesseract/)。
  
[microblink](https://microblink.com/)，免费的移动OCR-SDK。

## 新方法：机器学习

如果有够多的样本和验证能力，机器学习可以很好的处理OCR的问题。
  
http://www.danvk.org/2015/01/09/extracting-text-from-an-image-using-ocropus.html
  
http://www.danvk.org/2015/01/11/training-an-ocropus-ocr-model.html
  
https://en.wikipedia.org/wiki/Long\_short\_term_memory
  
https://github.com/nypl/map-vectorizer

一个快速深度学习的框架，和基于它构建的OCR项目。
  
https://github.com/BVLC/caffe/
  
https://github.com/pannous/caffe-ocr

JS构建的神经学习网络
  
https://github.com/mateogianolio/mlp-character-recognition

## 顺便

[ImageMagick](http://www.imagemagick.org/) —— 实现PDF、PNG、TIFF之间的格式转换。
  
[Apache Tika](https://tika.apache.org/) —— 从HTML、Word、Pdf、Excel、PPT、Zip等文档中提取内容的类库，in JAVA。