---
layout: post
title: "我自己常常用到的第三方控件"
date: 2013-08-20 09:49:23 +0800
comments: true
categories: iOS
---

- EGOTableViewPullRefresh（下拉刷新控件)
[Github地址](https://github.com/emreberge/EGOTableViewPullRefresh)

只需要实现`PullTableViewDelegate`的这两个委托:

	- (void)pullTableViewDidTriggerRefresh:(PullTableView *)pullTableView
	{
	    [self performSelector:@selector(refreshTable) withObject:nil afterDelay:3.0f];
	}

	- (void)pullTableViewDidTriggerLoadMore:(PullTableView *)pullTableView
	{
	    [self performSelector:@selector(loadMoreDataToTable) withObject:nil afterDelay:3.0f];
	}

这里面指向的具体方法是数据请求完成后进行处理和显示的操作，比如一般在里面reloadData之类：

	- (void) refreshTable
	{
	    /*
	     Code to actually refresh goes here.
	     */
	    self.pullTableView.pullLastRefreshDate = [NSDate date];
	    self.pullTableView.pullTableIsRefreshing = NO;
	}

	- (void) loadMoreDataToTable
	{
	    /*
	     Code to actually load more data goes here.
	     */
	    self.pullTableView.pullTableIsLoadingMore = NO;
	}

几乎就这么多，在注释部分进行逻辑处理，就好了。
注:这不是原版的，但我感觉是最完善的（上拉加载更多以及更自由的定制（背景颜色）等等），而且使用也更加简化。

- MMDrawerController(抽屉视图)
[Github地址](https://github.com/mutualmobile/MMDrawerController)

抽屉式布局一直很火，应用也很广泛，相关控件也很多，感觉这个控件封装得比较好，而且功能也强大的很。
核心调用代码就几行：

	UIViewController * leftDrawer = [[UIViewController alloc] init];
	UIViewController * center = [[UIViewController alloc] init];
	UIViewController * rightDrawer = [[UIViewController alloc] init];
	MMDrawerController * drawerController = [[MMDrawerController alloc]
	                                       initWithCenterViewController:center
	                                           leftDrawerViewController:leftDrawer
	                                           rightDrawerViewController:rightDrawer];

其他的下载官方的demo。十分灰常详细。

- EGOImageView(异步加载图片)

这个控件的使用非常简单，不用介绍过多~
而关于这个控件我有一篇专门的博客，可戳[这里](http://kinoandworld.github.io/2013/08/05/egoimage-addition/)

- PZPagingScrollView(延迟加载的滚动视图)
[额……地址暂时找不到了。]

用这个控件可以很方便地做一个相册，提供的委托挺多的，有点类似UITableView委托的写法：

	- (Class)pagingScrollView:(PZPagingScrollView *)pagingScrollView classForIndex:(NSUInteger)index;
	- (NSUInteger)pagingScrollViewPagingViewCount:(PZPagingScrollView *)pagingScrollView;
	- (UIView *)pagingScrollView:(PZPagingScrollView *)pagingScrollView pageViewForIndex:(NSUInteger)index;
	- (void)pagingScrollView:(PZPagingScrollView *)pagingScrollView preparePageViewForDisplay:(UIView *)pageView forIndex:(NSUInteger)index;

对这个控件我也做了一些修改，具体使用还是看例子吧。我都集成好放进一个demo里面了~

感兴趣的可以[看看](https://github.com/KinoAndWorld/UsefulThirdsControlsDemo)