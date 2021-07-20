---
layout: post
title: "亲历的iOS面试题"
date: 2013-12-03 09:49:23 +0800
comments: true
categories: 林中笔记
---

上个星期在V2EX上看到一个酷工作然后挺心动的，俗话说心动不如行动，所以我二话不说当基(机)立断地把简历投了去~
然后马不停蹄地跑去广州面试了~

说实话面试官江哥给我感觉很亲切……完全没有紧张的氛围~当然内心还是有点小紧张的~
好吧进入正题，主要是有几个问题我感觉自己理解的还是不够透彻，趁着面试暴露出来，现在就加深一下了解。

##block到底是什么东西
按照我的理解，block类似一个函数指针，主要应用场景总结下就是：
- 当调用其他的功能模块，或者自己写一个通用的功能模块的时候，代替delegate去做回调，这样的好处是不用再去声明回调函数，再去函数里面处理，而是直接把处理结果传进去，直接而优雅。
- 使用多线程（GCD）的时候，因为GCD的语法是直接用block的，所以这两个好搭档可谓是天衣无缝，举个栗子：

	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    	//进行异步操作
    	dispatch_async(dispatch_get_main_queue(), ^{
        	//回到主线程
    	});
	});


短短几行代码就可以线程安全地玩转在新创建的线程和主线程之间，这种便利的愉悦是不言而喻的~

因为我自己理解也不是很深~所以还需要慢慢学习

关于语法描述 [看这里](http://fuckingblocksyntax.com/)
还有个很好地资料 [看这里](http://onevcat.com/2011/11/objective-c%E4%B8%AD%E7%9A%84block/)


##多线程的几种实现方式

- NSTheard 

- NSOperation

- GCD

具体使用栗子：[http://www.uml.org.cn/mobiledev/201210262.asp](http://www.uml.org.cn/mobiledev/201210262.asp)


##页面动画的几种实现方式

- UIView animation 动画

- CoreAnimation 动画  参考：[iOS核心动画编程指南](http://www.dreamingwish.com/dream-2012/the-concept-of-coreanimation-programming-guide.html)

- 简单粗暴地用 OpenGL ES实现

这是我的答案……不知道是否正确或者全面
目前我也就第一个用得比较多，CoreAnimation还在慢慢学 ╮(╯▽╰)╭
