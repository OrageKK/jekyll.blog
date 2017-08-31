---
layout:     post
title:      "ijkPlayer 集成"
subtitle:   "iOS，前端开发，杂货铺"
date:       2017-08-30
author:     "oragekk"
header-img: "img/post-bg-2016-11-3.jpg"
tags:

    - iOS
    - 前端开发
    - 杂货铺 
---
> [参考地址](http://www.jianshu.com/p/b7a646a6c80e)
> ijkplayer 是一款做视频直播的框架，基于FFmpeg，支持Android和iOS。这里介绍一下iOS中集成ijkplayer

## 一、FFmpeg
FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。它包括了领先的音/视频编码库libavcodec等。

**libavformat**：用于各种音视频封装格式的生成和解析，包括获取解码所需信息以生成解码上下文结构
和读取音视频帧等功能；
**libavcodec**：用于各种类型声音/图像编解码；
**libavutil**：包含一些公共的工具函数；
**libswscale**：用于视频场景比例缩放、色彩映射转换；
**libpostproc**：用于后期效果处理；
**ffmpeg**：该项目提供的一个工具，可用于格式转换、解码或电视卡即时编码等；
**ffsever**：一个 HTTP 多媒体即时广播串流服务器；
**ffplay**：是一个简单的播放器，使用ffmpeg 库解析和解码，通过SDL显示；

### 支持的编码
源自FFmpeg项目组的两个视频编码：

Snow

FFV1
### 支持的格式
ASF

AVI

BFI[7]

IFF[8]

RL2[9]

FLV

MXF, Material eXchange Format, SMPTE 377M

Matroska

Maxis XA[10]

MSN Webcam stream[11]

MPEG transport stream

TXD[6]

OMA[12]

GXF, General eXchange Format, SMPTE 360M

mov,mp4,m4a,3gp,

### 支持的协议

HTTP

RTP

RTSP

RealMedia RTSP/RDT

TCP

UDP

Gopher

RTMP

RTMPT, RTMPE, RTMPTE, RTMPS (via librtmp)

SDP

MMS over TCP

## 二、下载ijkplayer
ijkplayer下载地址:[https://github.com/Bilibili/ijkplayer](https://github.com/Bilibili/ijkplayer)

下载完成后解压, 解压后文件夹内部目录如下图:
![ijkplayer.png](https://storage2.cuntuku.com/2017/08/31/ijkplayer.png)

## 三、编译
其实这里主要是编译FFmpeg，因为他是一个C语言的跨平台库，需要sh脚本来进行编译

1. 打开终端, cd 到jkplayer-master文件夹中, 也就是下载完解压后的文件夹, 如下图:![1.png](https://storage1.cuntuku.com/2017/08/31/1.png)
2. 执行命令行**./init-ios.sh**, 这一步是去下载 ffmpeg 的, 时间会久一点, 耐心等一下.如下图:![2.png](https://storage2.cuntuku.com/2017/08/31/2.png)
3. cd 到ios目录中
4. 执行**./compile-ffmpeg.sh clean**![3.png](https://storage1.cuntuku.com/2017/08/31/3.png)
4. 执行**./compile-ffmpeg.sh all**进行FFmpeg的编译，时间较久![4.png](https://storage2.cuntuku.com/2017/08/31/4.png)

## 四、打包IJKMediaFramework.framework框架
其实集成ijkplayer有两种方法，一种是按照Demo中的导入IJKMediaPlayer.xcodeproj，此方法不是很推荐

下面主要说另一种把 ijkplayer 打包成framework导入工程中使用. 
首先打开工程IJKMediaPlayer.xcodeproj,![IJKMediaPlayer.xcodeproj](http://upload-images.jianshu.io/upload_images/1803339-607cc84c212faf90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择IJKMediaFramework点击EditScheme![](http://upload-images.jianshu.io/upload_images/1803339-bbc0adc479cebb69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择release![](http://upload-images.jianshu.io/upload_images/1803339-daa4498f7e0746d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置好 scheme 后, 分别选择真机和模拟器进行编译, 编译完成后, 进入 Finder,![](http://upload-images.jianshu.io/upload_images/1803339-344cda905745ff39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面开始合并真机和模拟器版本的 framework, 注意不要合并错了, 合并的是这个文件, 如下图:![](http://upload-images.jianshu.io/upload_images/1803339-ec00ef4cb15c66d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开终端, 进行合并, 命令行具体格式为:
lipo -create 真机版本路径 模拟器版本路径 -output 合并后的文件路径

合并后如下图![](http://upload-images.jianshu.io/upload_images/1803339-d025e12bf804ee05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用合并后的IJKMediaFramework把原来的IJKMediaFramework替换掉![](http://upload-images.jianshu.io/upload_images/1803339-8d228ab56eb77f43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 五、在iOS项目中集成ijkplayer
新建工程, 导入合并后的IJKMediaFramework.framework以及相关依赖框架以及相关依赖框架,如下图:
![5.png](https://storage1.cuntuku.com/2017/08/31/5.png)

导入框架后在ViewController.m中进行buid，如果成功，说明集成成功。然后可以在控制器中写一个Demo测试![Snip20170831_10.png](https://storage1.cuntuku.com/2017/08/31/Snip20170831_10.png)
可以是mp4格式，也可以是m3u8,rtmp,hls等流媒体

>[demo地址](https://github.com/OrageKK/ijkPlayerDemo)