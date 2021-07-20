---
layout: post
title: "iOS常用第三方库的备忘和介绍"
date: 2013-10-21 09:49:23 +0800
comments: true
categories: 林中笔记
---

俗话说不要重复造轮子~身为一个IOS开发者我们有必要知道一些先进好用的第三方开源库，
不仅可以在项目中加快开发进度，更可以通过研究这些开源库提高自己的水平~

下面介绍自己用过和流行的一些开源库

##视图
- [MBProgressHUD](https://github.com/jdg/MBProgressHUD) 很常用的模态等待窗口~使用简单明了
- [PZPhotoView](https://github.com/brennanMKE/PhotoZoom) 一个封装的比较好的滚动视图
- [SDWebImage](https://github.com/rs/SDWebImage) 替代UIImageView，支持异步加载网络图片和自动缓存
- [EGOImageView](https://github.com/enormego/EGOImageLoading) 同上
- [ViewDeck](https://github.com/Inferis/ViewDeck) 仿Path和Facebook的左右侧滑视图
- [FlatUIKit](https://github.com/Grouper/FlatUIKit) 看起来很棒的UI组件 正如名字一样 扁平风
- [EGOTableViewPullRefresh](https://github.com/enormego/EGOTableViewPullRefresh) 下拉刷新 上拉加载更多的功能集成 很常用

##网络
- [AFNetworking](https://github.com/AFNetworking/AFNetworking) 最受欢迎的网络框架 集成得很好而且易于扩展 
- [MKNetworkKit](https://github.com/MugunthKumar/MKNetworkKit) 这个我比较常用 因为使用简单方便~


##数据
- [JSONKit](https://github.com/johnezang/JSONKit) 简单易用的JSON解析库
- [FMDB](https://github.com/ccgus/fmdb) 很知名的SQLITE操作封装库
- [CocoaSecurity](https://github.com/kelp404/CocoaSecurity) 提供了 AES加解密和Hash(MD5, HmacMD5, SHA1~SHA512, HmacSHA1~HmacSHA512)的加密

##综合
- [nimbus](https://github.com/jverkoey/nimbus) 一个替代THREE20的框架 功能集成了蛮多的
- [tapkulibrary](https://github.com/devinross/tapkulibrary) 封装了常用的UI控件和功能
- [sstoolkit](https://github.com/soffes/sstoolkit) 这个也比较知名 提供了很多常用的扩展和UI控件

先想到这么多……有空再补充吧~