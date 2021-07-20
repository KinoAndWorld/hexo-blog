---
layout: post
title: "how to use delegate in cocos2d x"
date: 2013-12-13 09:49:23 +0800
comments: true
categories: 技术杂谈
---

近来弄了下Coco2d-X，这个框架真的很棒，让游戏开发变得简单多了，以至于让我这样搞应用的都能很快上手。
熟悉objective-C编程的肯定都知道COCOA中的`protocol（协议）-delegate（委托）模式`
习惯了这种编程方式，然后在写cocos2d-x的时候发现没相关语法，感觉不太爽儿~

然后转念一想，原理其实很简单，可以自己去实现一个简陋的`protocol（协议）-delegate（委托）模式`。

无非就是定义一个接口，然后给delegate类这个接口的约束。

### C++的接口就是抽象类

		/**
		 *  协议
		 */
		class IBoardNumberClickDelegate {
		    
		public:
		    virtual ~IBoardNumberClickDelegate(){};
		    virtual void numberDidClick(int number) = 0;
		};

就是介么简单！！！

### C++通过多继承使用接口

		class MathQuestionScene : public CCScene , public IBoardNumberClickDelegate{
			//…… 为了节省篇幅我直接把实现写了
			void numberDidClick(int number){
        		CCLOG("点击%d",number);
    		}
		}

### 然后另一个类调用

		class NumberKeyBoard : public CCNode{
		public:
		    IBoardNumberClickDelegate *delegate;
		private:
		    void numberSelected(int number){
		        if (delegate) {
		            delegate->numberDidClick(number);
		        }
		    }
		}

用的时候在MathQuestionScene内设置

		_numberKeyBoard = NumberKeyBoard::create();
    	_numberKeyBoard->delegate = this;

就OK了。