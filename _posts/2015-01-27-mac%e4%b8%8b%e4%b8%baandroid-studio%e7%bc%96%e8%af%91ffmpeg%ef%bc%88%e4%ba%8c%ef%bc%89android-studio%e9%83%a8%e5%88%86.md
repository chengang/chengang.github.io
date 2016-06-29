---
id: 4539
title: Mac下为Android Studio编译Ffmpeg（二）Android Studio部分
date: 2015-01-27T17:39:54+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4539
permalink: /4539.html
categories:
  - 备忘
  - 实践
tags:
  - Android
  - Android Studio
  - ffmpeg
  - mac
  - ndk
  - 安卓
---
Android Studio和Eclipse不太一样，它有一定的自动生成Android.mk并自动搞定JNI的能力。
  
但目前还并不足以让我们使用起来Ffmpeg库。
  
因此我们的思路是禁用掉Android Studio自动ndk-build的功能，手动编译我们的C代码达到目的。

首先当然要新建一个Android Studio项目。
  
我们使用$ROOT_DIR指代项目根目录。

## 1、Android Studio配置ndk路径

$ROOT_DIR/local.properties原先只配置了sdk。

<pre>sdk.dir=/usr/local/bin/android-sdk-macosx
</pre>

给它增加一行ndk的配置

<pre>sdk.dir=/usr/local/bin/android-sdk-macosx
ndk.dir=/usr/local/bin/android-ndk-r10d
</pre>

## 2、配置build.gradle

项目里面有两个build.gradle，一个在根目录下，一个在$ROOT_DIR/app/src下，我们要修改的是后者。
  
A>添加这一段以禁用自动ndk-build。

<pre>sourceSets.main.jni.srcDirs = []
</pre>

B>添加这一段让它知道用库

<pre>ndk {
            abiFilter "armeabi"
            moduleName "ovsplayer"
            ldLibs "log", "z", "m", "jnigraphics", "android"
        }
</pre>

修改后的build.gradle是这样的。

<pre>android {
    compileSdkVersion 21
    buildToolsVersion "21.1.1"

    sourceSets.main.jni.srcDirs = [] // 禁用自动执行ndk-build
    defaultConfig {
        applicationId "com.example.chengang.myapplication"
        minSdkVersion 15
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
        ndk {
            abiFilter "armeabi"
            moduleName "ovsplayer" // 这个是C文件的名字
            ldLibs "log", "z", "m", "jnigraphics", "android"
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
</pre>

## 3、生成头文件

执行命令，注意路径要根据自己的情况更改。

<pre>javah -d jni -classpath ..\..\build\intermediates\classes\debug com.example.nativeapp.app.MainActivity
</pre>

会生成这个文件$ROOT\_DIR/app/src/main/jni/com\_example\_chengang\_myapplication_MainActivity.h

## 4、编写C文件

$ROOT_DIR/app/src/main/jni/ovsplayer.c内容如下：

<pre>#include &lt;stdio.h>
#include &lt;stdlib.h>

#include "com_example_chengang_myapplication_MainActivity.h"
#include "libavutil/avutil.h"
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"


JNIEXPORT jstring JNICALL Java_com_example_chengang_myapplication_MainActivity_getStringFromNative
  (JNIEnv * env , jobject obj)
  {
        const char *url = "/mnt/sdcard/1.mp4";
        av_register_all();

        AVFormatContext *pFormatCtx = NULL;
        int ret = avformat_open_input(&pFormatCtx, url, NULL, NULL);

        ret = avformat_find_stream_info(input_context, NULL);
        int streamNum = input_context->nb_streams;

        char wd[512];
        sprintf(wd, "AVCODEC VERSION %u\n, streamNum[%d]"
             , avcodec_version()
             , streamNum
             );
        return (*env)->NewStringUTF(env, wd);
  }
</pre>

## 5、编写Java文件

$ROOT_DIR/app/src/main/java/com/example/chengang/myapplication/MainActivity.java内容如下。
  
其中getStringFromNative()方法是我们实现的，打印了Ffmpeg库的版本号（我编译的这个是3673444）和视频文件的信息出来。

<pre>package com.example.chengang.myapplication;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;


public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView view = (TextView) findViewById(R.id.mytext);
        view.setText(this.getStringFromNative());
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    public native String getStringFromNative();
    static {
        System.loadLibrary("swresample-1");
        System.loadLibrary("avutil-54");
        System.loadLibrary("avformat-56");
        System.loadLibrary("avcodec-56");
        System.loadLibrary("swscale-3");
        System.loadLibrary("ovsplayer");
    }
}

</pre>

附上布局文件是这样的。

<pre>&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    &lt;TextView
        android:id="@+id/mytext"
        android:text="@string/hello_world" android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

&lt;/RelativeLayout>
</pre>

## 6、编写项目Android.mk

Android.mk放到$ROOT_DIR/app/build/intermediates/ndk/debug/下。
  
注意其中的绝对路径&#8221;/Users/chengang/Code/Android/MyApplication4&#8243;是咱们的$ROOT_DIR。

<pre>LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := ovsplayer
LOCAL_LDLIBS := \
	-lm \
	-ljnigraphics \
	-landroid \
	-llog \
	-lz \

LOCAL_SHARED_LIBRARIES := libavformat libavcodec libswscale libavutil

LOCAL_SRC_FILES := \
	/Users/chengang/Code/Android/MyApplication4/app/src/main/jni/ovsplayer.c \

LOCAL_C_INCLUDES += /Users/chengang/Code/Android/MyApplication4/app/src/main/jni
LOCAL_C_INCLUDES += /Users/chengang/Code/Android/MyApplication4/app/src/debug/jni

include $(BUILD_SHARED_LIBRARY)
$(call import-module,ffmpeg-2.5.3/android/arm)

</pre>

简单说下LOCAL\_LDLIBS和LOCAL\_SHARED_LIBRARIES的区别，前者链接系统库，后者链接第三方库。
  
并不能换着用。

## 7、编译C代码

在终端下执行这个命令编译C代码。

<pre>/usr/local/bin/android-ndk-r10d/ndk-build NDK_PROJECT_PATH=null APP_BUILD_SCRIPT=/Users/chengang/Code/Android/MyApplication4/app/build/intermediates/ndk/debug/Android.mk APP_PLATFORM=android-21 NDK_OUT=/Users/chengang/Code/Android/MyApplication4/app/build/intermediates/ndk/debug/obj NDK_LIBS_OUT=/Users/chengang/Code/Android/MyApplication4/app/build/intermediates/ndk/debug/lib APP_ABI=armeabi
</pre>

## 8、运行项目

回到Android Studio中Ctrl+R运行项目。
  
会看到手机上打印出Ffmpeg库的版本号（我编译的这个是3673444）和视频文件的信息出来。
  
Ffmpeg库已经可以调用了，然后继续写自己的逻辑就好了。

## 9、另外

另外说两个C代码中可能遇见的错误：
  
A>如果你通过标准库想得到文件大小，又写错了文件名。安卓上，在不存在的文件的句柄上执行fseek会报如下错误：&#8221;could not disable core file generation.&#8221;；
  
B>如果avformat\_open\_input()返回-1330794744了。那有两种可能，一是你忘记调用av\_register\_all()了，二是编译Ffmpeg的时候没有编译进对应的DEMUXER或者指定了disable-everything。

本文前半配置ndk的部分见这个链接：[Mac下为Android Studio编译Ffmpeg（一）ndk部分](http://blog.yikuyiku.com/?p=4533 "Mac下为Android Studio编译Ffmpeg（一）ndk部分")