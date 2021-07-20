---
layout: post
title: "一些iOS面试题的探索"
date: 2013-12-17 09:49:23 +0800
comments: true
categories: 林中笔记
---

其实就是因为cocosachina的[一篇文章](http://www.cocoachina.com/gamedev/misc/2013/1209/7499.html)
看了之后发现自己好多objective-C的细节都没弄清楚，所以记录一下，会的不会的都写一下~

### ==题目==

1、假设有三个对象，一个父类的父类，一个父类和一个子类。父类的父类持有父类的引用（retain），父类持有子类的引用（retain），子类持有父类的引用（retain）。父类的父类释放（release）父类，解释下会发生什么。
	`父类被父类的父类释放掉以后，父类的引用计数-1，而因为子类拥有父类的计数，父类并未销毁。`

2、当一个空指针（nil pointer）调用了一个方法会发生什么？ 
	`安然无恙 —— 这是oc自带的消息机制，nil也能发送消息，而不会报错`

3、为什么retainCount绝对不能用在发布的代码中？请给出两个相对独立的解释。

	- retainCount受到时间和framework的影响太大，不能准确反映内存的引用计数
	- retainCount很容易迷惑人，采取规范的内存管理才是正道

4、请说明一下你查找或者解决内存泄露的处理过程。
	`使用instruments作为动态分析的手段，还有Xcode的静态内存分析`

5、解释下自动回收池(autorelease pool)在程序运行时是如何运作的.
	`xcode为开发者写的代码外层包了一层NSAutoreleasePool。建立一个回收池堆栈(Stack)每次对象发送autorelease消息时，对象的引用计数并不真正变化，而是向pool中添加一条记录，记下对象的这种要求。最后当pool发送drain或release消息时，池中的所有对象的这种要求一一被执行。顺便说下使用场景：`

- 应用不是基于”Application Kit”，像”Command-line tool”，因为它并没有内置的”autorelease pools”的支持。
- 创建线程，你必需在线程开始时创建一个”Autorelease Pool”实例。反之，会造成内存池露（会在以后的文章详细说明线程与池的技巧）。
- 一个循环内创建了太多的临时对象，你应该为他们创建一个”Autorelease Pool”对象，并在下次循还前销毁它们。

6、当处理属性申明的时候，原子（atomic）跟 非原子（non-atomic）属性有什么区别？
	`事关多线程，原子（atomic）可以说是线程安全的，也就是在读取这个属性的变量的时候，会进行一些额外的操作（比如锁）.所以说，atomic会比较安全但是比较耗时。`

7、在C语言中，你如何能用尽可能短的时间来倒转一个字符串？
	`这个我真不知道有什么高效的方法~`  [有篇文章可参考](http://www.cnblogs.com/JCSU/articles/1305401.html)

8、遍历一个NSArray和一个NSSet，哪一个更快？
	`其实意思是问，遍历一个链表和哈希表，哪个更快？`
	[参考一下](http://maccrazy.diandian.com/post/2011-10-09/5671451) 结论在最后

9、Objective-C中的posing指的是什么？
	[参考](http://blog.csdn.net/tenfyguo/article/details/9220535)

10、copy跟retain有什么区别？
	`一个是复制内容，一个引用计数+1，（nsstring比较特殊，两个的功能几乎一样）需要注意的是，自定义的类需要重写一个方法以实现自己的深复制：`

		-(id) copyWithZone:(NSZone *) Zone{
    		grandSuper * scCopy;
    		scCopy=[[[self class] allocWithZone:Zone]init];
    		return scCopy;
		}

11、执行如下的代码会发生什么情况？

		Ball *ball = [[[[Ball alloc] init] autorelease] autorelease]; 

------崩溃，因为重复释放，在自动回收池下一次进行回收时崩溃


很多地方理解可能还是有问题~继续努力吧 ↖(^ω^)↗
