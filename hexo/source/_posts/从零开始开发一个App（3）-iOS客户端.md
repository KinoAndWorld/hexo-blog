---
title: 从零开始开发一个App（3）- iOS客户端
date: 2016-08-05 13:14:10
tags:
categories: 技术杂谈
---

接上文，我们已经成功地在本地跑起了一个rails服务端，并且正确返回了JSON的接口数据，可以说万事俱备只欠客户端了。

终于来到了主战场，对我来说这是最繁琐但也是最简单的一步 :)

### 工程搭建

二话不说，老规矩祭出Xcode，新建工程，新建Podfile

```ruby
platform :ios, '7.0'

pod 'Masonry'
pod 'BFKit'

pod 'ReactiveCocoa'

pod 'AFNetworking'

pod 'YYModel'
pod 'YYWebImage'
```
然后安装`pod install`
 
### 界面布局

我们可以先完成简单的页面布局，由于目前的数据非常单一，所以也就简单单页面实现，以一个CollectionView作为主控件，每个评测一个cell。


新增`ReviewListView`继承自UICollectionView，自定义UICollectionViewFlowLayout子类`ReviewListViewLayout`，然后用`Masonry`手写布局。

```ruby
ReviewListView *colview = [[ReviewListView alloc] init];
colview.delegate = self;
colview.dataSource = self;
[colview registerClass:[ReviewSummaryCell class] forCellWithReuseIdentifier:@"ReviewSummaryCell"];
	
_reviewListView = colview;
[self.view addSubview:_reviewListView];
	
[_reviewListView mas_makeConstraints:^(MASConstraintMaker *make) {
	make.edges.equalTo(self.view);
}];
```

接着实现UICollectionViewDataSource的代理方法，这个时候需要新建UICollectionViewCell的子类`ReviewSummaryCell`，然后手写添加控件，完成基本布局。

自此，一个简单的页面搭建已经完成，下面是数据请求的接入。


### 网络层

网络层是在基于*AFNetworkKit*的基础上，连接数据请求模块和ViewController的中间层，我稍微封装了一下，详细的代码可以在文末的源码连接查看。

大致步骤是新建一个`ReviewApiEngine`类，继承自`BaseApiEngine`

网络请求大致代码如下：

```objc
/**
 *  获取评测列表
 */
+ (void)fetchFeatureListWithSuccessHandler:(ResponseBlock)completeBlock
							  errorHandler:(ErrorBlock)errorBlock
								translater:(Class<DataTranslate>)translater{
	
	[[APIManager shareManager] sendRequestFromMethod:APIMethodGet
												path:API_ReviewList
											  params:nil
										  onComplete:Default_Handle
											 onError:errorBlock];
}

```

然后在controller只需要调用

```ruby
- (void)fetchData{
	useWeakSelf
	[ReviewApiEngine fetchReviewListWithSuccessHandler:^(id responseResult) {
		
		[weakSelf.reviewListView reloadData];
		
	} errorHandler:^(NSError *error) {
		
	} translater:[GameReview class]];
}
```
便可以轻松完成网络数据请求的任务。

我们可以先写测试或者在App生命周期中先调用网络请求，确保能正确无误地拿到数据。
一切准备就绪后，我们在`ReviewSummaryCell`中添加配置方法：

```
- (void)configWithIndexPath:(NSIndexPath *)indexPath model:(GameReview *)review{
	
	[_thumbImageView setImageURL:[NSURL URLWithString:review.cover_image]];
	
	_titleLabel.text = review.title;
	
	_summaryLabel.text = review.summary;
	
	_scoreView.score = review.score;
}
```

最终完成效果如图：


![模拟器截图](http://upload-images.jianshu.io/upload_images/25038-1570b079cf488d95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 尾声

这个实践对我来说是一次非常有意义的技术探索，扩展起来，我们可以发挥更多想象力做更多有趣以及有意义的事情，技术改变生活不应是一句口号，而应当是我们行动的信念。

好了，打完收工。

所有的源代码放在[这里](https://github.com/KinoAndWorld/YouxiaReview)。

→_→

---

**系列链接**

{% post_link 从零开始开发一个App（1）-Scrapy爬虫 %}
{% post_link 从零开始开发一个App（2）-简易REST-API服务端 %}
{% post_link 从零开始开发一个App（3）-iOS客户端 %}