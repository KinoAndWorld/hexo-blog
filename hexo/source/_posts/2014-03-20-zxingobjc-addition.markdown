---
layout: post
title: "ZXingObjC-addition"
date: 2014-03-20 16:10:20 +0800
comments: true
categories: 技术杂谈
---

##二维码扫描库ZXingObjC功能完善

对于做过二维码扫描的大概都对ZXing这个库不陌生，然而这个库由于跨平台，代码文件过于臃肿，而且各种步骤非常麻烦。。。
还有不知道什么原因，ZXing的iOS库在最新发布版本不见了。 `还是我找不到？` 
总之因为种种原因，[ZXingObjC]("https://github.com/TheLevelUp/ZXingObjC") 作为良好的替代方案粗线了。

顺带一提，支持 cocoapods => 'ZXingObjC', '~> 2.2.6'

好吧，现在进入主题，这个库很好用，但是我们在做二维码的时候一般会有一个需求，让二维码进入一个区域方才扫描，即是`扫描区域`
让我比较郁闷的是，这么完善的库居然貌似没有实现这个很常见的功能。

然后上GitHub上看了一下这个[Issue]("https://github.com/TheLevelUp/ZXingObjC/issues/100") , 以为终于找到解决方案……却发现是个坑。具体怎么坑就不说了……也许是我不会用吧。

然后重新阅读了源码，发现了一个更简单的设置裁剪区域的方法。

--

###Let's go

首先，进入`ZXCapture.h` ，添加一个property

``` objc
	@property (nonatomic, assign) CGRect scanCrop;
```


然后进入`ZXCapture.m` 的这个方法：


``` objc
  - (void)captureOutput:(ZXCaptureOutput *)captureOutput
ZXQT(didOutputVideoFrame:(CVImageBufferRef)videoFrame
     withSampleBuffer:(QTSampleBuffer *)sampleBuffer)
ZXAV(didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer)
       fromConnection:(ZXCaptureConnection *)connection;
```

将大概570行位置的代码修改为

``` objc
	CGImageRef videoFrameImage = NULL ;
      if(CGRectEqualToRect(self.scanCrop,CGRectZero)){
          videoFrameImage = [ZXCGImageLuminanceSource createImageFromBuffer:videoFrame];
      }else{
          videoFrameImage = [ZXCGImageLuminanceSource createImageFromBuffer:videoFrame
                                                                       left:_scanCrop.origin.x
                                                                        top:_scanCrop.origin.y
                                                                      width:_scanCrop.size.width
                                                                     height:_scanCrop.size.height];
      }
```

就这么简单。。。最后只要在客户端做类似这样的设置就好

``` objc
	self.capture.scanCrop = CGRectMake(_scanImageView.frame.origin.y,
                                       _scanImageView.frame.origin.x,
                                       _scanImageView.frame.size.width,
                                       _scanImageView.frame.size.height);
```

唯一需要注意的是，这个frame的 x 和 y 需要颠倒，具体原因应该跟capture的rotation有关。

暂时这么多吧- -虽然没人看~默默记录。