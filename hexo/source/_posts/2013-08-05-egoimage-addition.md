---
layout: post
title: "给EGOImageView增加下载进度功能"
date: 2013-08-05 09:49:23 +0800
comments: true
categories: iOS
---

### 给EGOImageView异步加载图片增加下载进度功能

--

做网络图片相关的项目时，离不开缓存和异步加载。于是找到了一个很好用的第三方控件
 [EGOImageView](https://github.com/enormego/EGOImageLoading)

灰常简单好用~

但是呢，随着项目的进一步展开，弱弱地发现这个控件并没有`获取图片加载进度`这么个功能，而且一般我们等待图片加载时需要有一个显示进度的进度条，这样就无法实现了。

于是我上网找到了获取进度的方法，然后自己改造了一下EGOImageView，写这篇博客，仅做一下记录。

### 首先，找到EGOImageLoadConnection这个类
在它的.h文件加上下面两个自定义的委托

	@protocol EGOImageLoadConnectionDelegate<NSObject>
	- (void)imageLoadConnectionDidFinishLoading:(EGOImageLoadConnection *)connection;
	- (void)imageLoadConnection:(EGOImageLoadConnection *)connection didFailWithError:(NSError *)error;
	//进度委托
	- (void)imageLoadConnection:(EGOImageLoadConnection *)connection didDownLoadData:(long long)btyes;
	- (void)imageLoadConnection:(EGOImageLoadConnection *)connection countDownLoadData:(long long)btyes;
	@end

紧接着，进入.m文件，找到这两个方法，这两个方法是关键。
	didReceiveResponse是得到一个有效图片URL后返回的信息，里面就包含了图片的大小
	didReceiveData则是在一张图片下载过程中，不断传输过来的数据，因此这个方法可能会调用不止一次

	- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
		if(connection != _connection) return;
		[_responseData appendData:data];
	}

	- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
		if(connection != _connection) return;
		self.response = response;
	}

网上找了方法，加进去，变成下面这样

	- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
		if(connection != _connection) return;
		[_responseData appendData:data];
	    //NSLog(@"%lld",(long long)[data length]);
	    if([self.delegate respondsToSelector:@selector(imageLoadConnection:didDownLoadData:)]) {
	        [self.delegate imageLoadConnection:self didDownLoadData:(long long)[data length]];
	    }
	}

	- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
	    
	    long long total_;
	    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
	    if(httpResponse && [httpResponse respondsToSelector:@selector(allHeaderFields)]){
	        NSDictionary *httpResponseHeaderFields = [httpResponse allHeaderFields];
	        
	        total_ = [[httpResponseHeaderFields objectForKey:@"Content-Length"] longLongValue];
	        if([self.delegate respondsToSelector:@selector(imageLoadConnection:countDownLoadData:)]) {
	            [self.delegate imageLoadConnection:self countDownLoadData:total_];
	        }
	        //NSLog(@"total:%lld", total_);
	    }
		if(connection != _connection) return;
		self.response = response;
	}

理论上，现在已经获取到了数据的大小和每次传输的数据量，接着进入EGOImageLoader.m文件。
然后我遵循它原本的模式（观察者），添加两个宏定义:

	#if __EGOIL_USE_NOTIF
		#define kImageNotificationLoaded(s) [@"kEGOImageLoaderNotificationLoaded-" stringByAppendingString:keyForURL(s, nil)]
		#define kImageNotificationLoadFailed(s) [@"kEGOImageLoaderNotificationLoadFailed-" stringByAppendingString:keyForURL(s, nil)]
		//下载进度
		#define kImageNotificationLoadData(s) [@"kEGOImageLoaderNotificationLoadedData-" stringByAppendingString:keyForURL(s, nil)]
		#define kImageNotificationCountLoadData(s) [@"kEGOImageLoaderNotificationCountLoadData-" stringByAppendingString:keyForURL(s, nil)]
	#endif

接着同样在 `-(void)loadImageForURL:(NSURL*)aURL observer:(id<EGOImageLoaderObserver>)observer;`
方法添加观察者

	//进度的观察
    if([observer respondsToSelector:@selector(imageLoaderDidLoadData:)]) {
		[[NSNotificationCenter defaultCenter] addObserver:observer selector:@selector(imageLoaderDidLoadData:) name:kImageNotificationLoadData(aURL) object:self];
	}
    if([observer respondsToSelector:@selector(imageLoaderDidCountLoadData:)]) {
		[[NSNotificationCenter defaultCenter] addObserver:observer selector:@selector(imageLoaderDidCountLoadData:) name:kImageNotificationCountLoadData(aURL) object:self];
	}

和在 `-(void)removeObserver:(id<EGOImageLoaderObserver>)observer forURL:(NSURL*)aURL;`
方法删除观察者

	// 移除下载进度观察者
    [[NSNotificationCenter defaultCenter] removeObserver:observer name:kImageNotificationCountLoadData(aURL) object:self];
    [[NSNotificationCenter defaultCenter] removeObserver:observer name:kImageNotificationLoadData(aURL) object:self];


接着，该实现刚刚写的那两个委托了

	#pragma mark URL Connection delegate methods
	//下载进度
	- (void)imageLoadConnection:(EGOImageLoadConnection *)connection didDownLoadData:(long long)btyes{
	#if __EGOIL_USE_NOTIF
	    NSNotification* notification = [NSNotification notificationWithName:kImageNotificationLoadData(connection.imageURL)
	                                                                 object:self
	                                                               userInfo:[NSDictionary dictionaryWithObjectsAndKeys:
	                                                                         [NSNumber numberWithLongLong:btyes],@"btyes",
	                                                                         connection.imageURL,@"imageURL",nil]];
	    
	    [[NSNotificationCenter defaultCenter] performSelectorOnMainThread:@selector(postNotification:) withObject:notification waitUntilDone:YES];
	#endif
	}

	- (void)imageLoadConnection:(EGOImageLoadConnection *)connection countDownLoadData:(long long)btyes{
	#if __EGOIL_USE_NOTIF
	    NSNotification* notification = [NSNotification notificationWithName:kImageNotificationCountLoadData(connection.imageURL)
	                                                                 object:self
	                                                               userInfo:[NSDictionary dictionaryWithObjectsAndKeys:
	                                                                         [NSNumber numberWithLongLong:btyes],@"btyes",
	                                                                         connection.imageURL,@"imageURL",nil]];
	    
	    [[NSNotificationCenter defaultCenter] performSelectorOnMainThread:@selector(postNotification:) withObject:notification waitUntilDone:YES];
	#endif
	}

最后呢，我们再给EGOImageView.h文件加两个委托

	//图片进度
	- (void)imageViewImageProgress:(long long)bytes;
	- (void)imageViewImageProgressCount:(long long)bytes;

还记得我刚刚加的两个通知么，现在派上用场了，在.m文件加上如下代码

	//进度处理
	- (void)imageLoaderDidLoadData:(NSNotification*)notification {
		if(![[[notification userInfo] objectForKey:@"imageURL"] isEqual:self.imageURL]) return;
	    long long bytes = [[[notification userInfo] objectForKey:@"btyes"] longLongValue];
	    
		//NSLog(@"sub:%lld",bytes);
	    if([self.delegate respondsToSelector:@selector(imageViewImageProgress:)]) {
			[self.delegate imageViewImageProgress:bytes];
		}
	}

	- (void)imageLoaderDidCountLoadData:(NSNotification*)notification {
	    
		if(![[[notification userInfo] objectForKey:@"imageURL"] isEqual:self.imageURL]) return;
		long long bytes = [[[notification userInfo] objectForKey:@"btyes"] longLongValue];
	    
	    if([self.delegate respondsToSelector:@selector(imageViewImageProgressCount:)]) {
			[self.delegate imageViewImageProgressCount:bytes];
		}
	}

大功告成。
<img src="{{ page.image }}" />