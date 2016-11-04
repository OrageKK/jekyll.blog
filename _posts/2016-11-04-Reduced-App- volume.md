---
layout:     post
title:      "减小iOS-App或者静态库体积"
subtitle:   "CommoniOS，RequireiOS, 归纳笔记"
date:       2016-11-04 18:11:56
author:     "Oragekk"
header-img: "img/post-bg-js-module.jpg"
tags:

    - 前端开发
    - iOS
---
![](https://zero-space.s3.amazonaws.com/photos/2e6cc210-e9a6-4a0f-9c05-2c4056a982acx840.jpg)


>把打包好的.ipa文件的后缀改为.zip并解压。右键.appbundle 选择显示包内容。有些情况下，大一点的文件压缩后反而比小一点的文件压缩后的体积小，而我们真正关心的时候解压后的真实体积，所以一定要解压里面的资源文件，看解压后的size。从APP Store下载的.ipa文件要比自己本地打包的要大，因为APP Store对ipa包又做了加密处理。Xcode的Organizer window的Estimate Size功能能估计本地打包文件从APP Store下载时的大小。根据优化的28定律和常识，首先当然是多媒体资源的体积啦。

## 图片
压缩图片 不重要的图片可适当采用 8bit PNG图片
1.什么是矢量图 矢量图是由计算机的算法产生的，可以无限放大或缩小，不会有任何损失，通常由矢量软件制作。
2.什么是位图 位图是由一个一个的小色块组成，放大后会看到那些小色块，同一面积内小色块越多，分辨率就越高。
3.矢量图的优缺点 可以无限放大或缩小，不会影响图像素质，文件体积较小，编辑灵活。缺点是表达的色彩层次不清，整体观感效果不如位图
4.位图的优缺点 不能放太大，减少文件分辨率后会影响图片质量，图片战胜空间较大，优点是能很细腻地表达图片的效果，图片表达效果非常好
5.什么情况下用位图，什么情况下用矢量图 一些对图片要求高的用位图，例如照片。其他的尽量用矢量图。例如文字、表格、卡通图片等

- 去掉无用的图片

- 用代码绘制简单的纯色图片 [用Sketch和PaintCode快速得到绘制代码](http://www.jianshu.com/p/d01110c80495)

- 如果不需要使用透明，可以用jpeg代替PNG。jpeg减少了些效率但更加小。需权衡性能，大小。

- 对32位的图片，尽肯能的使用高压缩率，使用PS的“Save For Web”功能，可以有效的减小JPEG和PNG图片的尺寸。 默认情况下，在build时，PNG图像就被[pngcrush](https://developer.apple.com/library/ios/qa/qa1681/_index.html)压缩。

## 音频
- 压缩音频，[尽可能使用AAC或者MP3格式，并且使用一个较低的码率。通常44.1khz的码率有点浪费，降低一定的码率也不会丢失多少音质](https://developer.apple.com/videos/wwdc/2011/?id=404)

## 视频
- 视频也可以使用类似于音频的处理方法，音视频的压缩可以很大程度的压缩，但是要注意压缩的格式，是不是会增加编解码的负担，这要权衡考虑。

## Assets
- 检查bundle中的无用文件，不要打包到app或者静态库中。可以点击文件，在右侧的file inspector里面的target membership中取消勾选；或者在build phase里面的Copy Bundle Resources中去掉。
- 确定 dead code（代码被定义但从未被调用）被剥离，build setting 里 DEAD_CODE_STRIPPING = YES。 去掉冗余的代码，即使一点冗余代码，编译后体积也是很可观的。

## 编译设置
- ``Optimization Level``设置为``Fastest, Smallest [-Os]``，``Strip Debug``
 ``Symbols During Copy``设置为``YES (COPY_PHASE_STRIP = YES)``
这样会减小接近一半的体积，但是在release版本，这些貌似是默认的配置，但是不妨也检查一下。 此外在debug版本最好在完成开发测试后，设置成这种模式，重新测试一遍，因为不同的编译设置可能会掩盖一些bug。

- 设置``IOS_DEPLOYMENT_TARGET`` 为想要运行系统的最低版本

- 设置需要的arm [架构](http://lib.csdn.net/base/16)，设置 ``ARCHS = arm64``可以消除armv6架构，潜在的减少近一半的容量。
[iOS Devices: ARM，尺寸，像素一览表](https://www.innerfence.com/howto/apple-ios-devices-dates-versions-instruction-sets)
1，如果想自己的app在各个机器都能够最高效率的运行，则需要将Build Active Architecture Only改为NO,Valid architectures选择对应的指令集：armv7 armv7s arm64。这个会为各个指令集编译对应的代码，因此最后的 ipa体积基本翻了3倍,Release版本必须NO。
2，如果想让app体积保持最小，则现阶段应该选择Valid architectures为armv7,这样Build Active Architecture Only选YES或NO就无所谓了

## 其他

- 将应用的中一些数据，如长字符串、表格等移到外部文件中，不要放在代码里面，这样能减小一些体积，因为外部文件的压缩率要比应用中的数据压缩率高。

## 编译选项
1. 编译器优化级别
Build Settings->Optimization Level有几个编译优化选项，release版应该选择Fastest, Smalllest，这个选项会开启那些不增加代码大小的全部优化，并让可执行文件尽可能小。
2. 去除符号信息
Strip Linked Product / Deployment Postprocessing / Symbols Hidden by Default 在release版本应该设为yes，可以去除不必要的调试符号。Symbols Hidden by Default会把所有符号都定义成”private extern”，详细信息见[官方文档](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/executing_files.html#//apple_ref/doc/uid/TP40001829-97021-TPXREF121)。
这些选项目前都是XCode里release的默认选项，但旧版XCode生成的项目可能不是，可以检查一下。其他优化还可以参考官方文档—[CodeFootprint.pdf](https://developer.apple.com/legacy/library/documentation/Performance/Conceptual/CodeFootprint/CodeFootprint.pdf)
### 第三方库统计
项目里会引入很多第三方静态库，如果能知道这些第三方库在可执行文件里占用的大小，就可以评估是否值得去找替代方案去掉这个第三方库。我们可以从linkmap中统计出这个信息，对此写了个node.js脚本，可以通过linkmap统计每个.o目标文件占用的体积和每个.a静态库占用的体积，[详见这里](https://gist.github.com/bang590/8f3e9704f1c2661836cd)(需翻墙)。
### ARC->MRC
有人提出用ARC写的代码编译出来的可执行文件是会比用MRC大的，原因大致是ARC代码会在某些情况多出一些retain和release的指令，例如调用一个方法，它返回的对象会被retain，退出作用域后会被release，MRC就不需要，汇编指令变多，机器码变多，可执行文件就变大了。还有其他细节实现的区别，先不纠结了。
那用ARC究竟会增大多少体积？我觉得从汇编指令的增多减少去算是很难算准确的，这东西涉及细节太多，还是得从统计的角度计算。做了几个对比试验，统计了几个同时支持ARC/MRC的开源项目在开启/关闭ARC的情况下__TEXT代码段的大小对比。只对比__TEXT代码段是因为：
ARC对可执行文件大小的影响几乎都是在代码段
可执行文件会进行某种对齐，例如有些段在不足32K的时候填充0直到对齐32K，若用可执行文件大小对比结果可能是对齐后的，不准确。

实验数据：

-------|MBProgressHUD|SDWebImage|FMDB
-------|:-----------:|:--------:|:-----:
开启ARC|19532|24424|29056
关闭ARC|17648|22575|25848
对比|↓9.6%|↓7.5%|↓11%


结果是ARC大概会使代码段增加10%的size，考虑代码段占可执行文件大约有80%，估计对整个可执行文件的影响会是8%。
可以评估一下8%的体积下降是不是值得把项目里某些模块改成MRC，这样程序的维护成本上升了，一般不到特殊情况不建议这么做。
### 无用代码
在项目里新建一个类，给它添加几个方法，但不要在任何地方import它，build完项目后观察linkmap，你会发现这个类还是被编译进可执行文件了。
按C++的经验，没有被使用到的类和方法编译器都会优化掉，不会编进最终的可执行文件，但object-c不一样，因为object-c的动态特性，它可以通过类和方法名反射获得这个类和方法进行调用，所以就算在代码里某个类没被使用到，编译器也没法保证这个类不会在运行时通过反射去调用，所以只要是在项目里的文件，无论是否又被使用到都会被编译进可执行文件。
对此我们可以通过脚本，遍历整个项目的文件，找出所有没有被引用的类文件和没有被调用的方法，在保证没有其他地方动态调用的情况下把它们去掉。如果整个项目历时很长，历时代码遗留较多，这个清理对可执行文件省出的空间还是挺可观的。
### 类/方法名长度
观察linkmap可以发现每个类和方法名都在__cstring段里都存了相应的字符串值，所以类和方法名的长短也是对可执行文件大小是有影响的，原因还是object-c的动态特性，因为需要通过类/方法名反射找到这个类/方法进行调用，object-c对象模型会把类/方法名字符串都保存下来。
对此我们可以考虑在编译前把所有类和方法名进行混淆，跟压缩js一样，把长名字替换成短名字，这样做的好处除了缩小体积外，还对安全性有很大提升，别人拿到可执行文件对它class-dump出来的结果都是混淆后的类和方法名，就无法从类和方法名中猜出某个方法是做什么的，就难以挂钩子进行hack。不过这样做有个缺点，就是crash堆栈反解出来的堆栈方法名会是混淆后的，需要再加一层混淆->原名的转换，实现和使用成本有点高。
实际上这部分占用的长度比较小，中型项目也就几百K，对安全性要求高的情况可以试试。
### 冗余字符串
代码上定义的所有静态字符串都会记录在在可执行文件的__cstring段，如果项目里Log非常多，这个空间占用也是可观的，也有几百K的大小，可以考虑清理所有冗余的字符串。另外如果有特别长的字符串，建议抽离保存成静态文件，因为AppStore对可执行文件加密导致压缩率低，特别长的字符串抽离成静态资源文件后压缩率会比在可执行文件里高很多。
### CheckList
最后把缩减[iOS](http://lib.csdn.net/base/1)安装包大小的各种方法列出来做了张CheckList图：
![](http://upload-images.jianshu.io/upload_images/2076247-dde8e96bdd40396b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考文献：http://blog.cnbang.net/tech/2296/
