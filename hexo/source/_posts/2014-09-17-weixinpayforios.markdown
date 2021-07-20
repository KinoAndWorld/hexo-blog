---
layout: post
title: "微信支付的小坑"
date: 2014-09-17 00:08:59 +0800
comments: true
categories: iOS
---

## 先说点题外话

貌似很久很久没更新博客，有半年了吧，这半年忙着毕业的各种事情，忙着小或者比较大的项目，感受着初入职场的艰辛与喜悦，以及过着平淡无奇的小日子————好吧虽然这些都可以成为懒惰的理由，但是回顾一些自己这半年真的过于放松，许多学习的时间被游戏动漫日剧所占据，如果平时不用上班恐怕要变成废宅了吧╮(╯_╰)╭，so，确定一下今后的目标：`多写代码、多锻炼、多阅读、多写博客`，写博客主要为了思考，因为要在脑中整理语言，沉淀思想，一方面可以巩固记忆，另一方面也算是[热爱生活]的一种方式吧。


## 好吧主题其实是

其实有点为了写博客而写博客的意思，因为要分享的东西挺简单的，更像是吐槽……好吧其实我就是吐槽。

- 吐槽1：微信支付为啥没有iOS的官方demo……实在想不通，服务端有，Android甚至WindowsPhone都有，微信这是对iOS平台歧视的节奏- -
幸好我大天朝人才辈出，早有人根据`渣成翔`的官方文档写了微信支付的[非官方demo](https://github.com/gbammc/WechatPayDemo)

里面用了cocoapods， 需要下载的话好好看下说明。

- 吐槽2：微信官方文档
虽然我知道，也理解 写代码的人一般比较讨厌写文档，但是毕竟你是一个支付组件，总有很多人需要仔细阅读的甚至copy文档里边的代码的，你可以写得很简单，但是不要犯一些低级错误好吗 (#‵′)。说明含糊不清我就不提了（其实上面的github主页有吐槽），实例代码都写错这就有点。。。
贴两段：


``` objc
	PayReq *request = [[[PayReq alloc] init] autorelease]; 
	request.partnerId = _pactnerid;
	request.prepayId= _prapayid;
	Request.package = _package;
	request.nonceStr= _noncestr;
``` 

第三行好好的request突然变成Request了……当时看到整个人都不太好

And

``` objc
	// 构造参数列表
	NSMutableDictionary params = [NSMutableDictionary dictionary]; 
	[params setObject:@"1234567" forKey:@"appid"];
```

NSMutableDictionary params。。。原来NSMutableDictionary不是引用类型啊T.T

有些文档看了真的会哭。

- 吐槽3：微信支付SDK

处于安全便捷考虑，几乎所有的操作都在服务端完成，然后今天服务端给出API，也跟Android调通了。我本以为既然Android端都OK了，那iOS端应该也没多大问题……但是，我错了，我发现iOS端调用微信支付不成功，马上弹回原应用，拿到的是`errCode = -1`的错误。这些我就思密达了，然后一直在找是不是自己哪里调用不对。弄了一个多小时，未果。
然后很郁闷地吃了个饭，回去之后我跟服务端一步步联调……才发现问题出在PayReq的`sign`变量和`package`变量不对应。

再深入纠结原因，发现原来Android的文档或者SDK和iOS的处理是不一样的，而服务端是参考了Android的文档来做，我当场就呵呵了。
如果两端的处理方法不一样，至少也该说明一下吧。

- - -





