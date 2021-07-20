---
layout: post
title: "Runtime实践之打造易复用的iOS公共页面"
date: 2016-05-03 18:04:23 +0800
comments: true
categories: iOS
---

相信我们开发的项目中，只要涉及到网络交互，都会遇到一个再普遍的不过的需求，那就是出于用户体验的需要，在请求开始的时候显示`加载页`，请求到空数据的时候显示`空内容页`，以及请求出错的时候显示的`错误或重试页`。这三类页面在一个项目中通常是一致的（至多会有图标和文案的变化），但却要求可能在每一个涉及网络请求的页面呈现。

##(；′⌒`)
如果没有大局观，一开始接到需求就开始在某ViewController里面添加几个View用来展现。

举个例子，如果要添加一个loading view，

```objc
@property (strong, nonatomic) UIView *loadingView;
@property (strong, nonatomic) UIImageView *loadingImageView;
```
然后增加方法，视图的初始化和配置就省略了
```objc
/**
 *  加载视图
 */
- (void)startLoading{
    [self.view addSubview:self.loadingView];
    [self.loadingImageView startAnimating];
}

/**
 *  停止加载并消失
 */
- (void)stopLoading{
    [self.loadingImageView stopAnimating];
    [self.loadingView removeFromSuperview];
}
```


OK，这样做实现上没问题，但是遇到下一个需要展示这些页面的ViewController，只能使用copy&paste大法，把property和方法实现都搬到另一个ViewController，倘若有10个以上的页面，再加上万一需要修改页面的视图结构，你就会深刻的体会到

![img](https://upload-images.jianshu.io/upload_images/25038-62e32ccb70ef2db5.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/220/format/webp)


总所周知有个大原则叫做Don't repeat yourself。再运用上我们不为什么就很熟练的面向对象思维，自然而然可以想到，使用继承大法。

##╮(╯_╰)╭

实现方法很简单

- 首先创建一个BaseViewController，将几个property转移过来，并且展现方法也照搬，在.h文件暴露出来。

- 然后把所有用到的Controller都继承自BaseViewController，调用的地方可以保持不变，会自动调用父类的方法。

但是这样做还是不够好，因为这需要我们把所有的Viewcontroller的头文件都改一遍，引入BaseViewController并集成，就是所谓这是带有`侵入性`的。更致命的是，如果你的ViewController本身集成了另外的BaseController，由于Objective-C不支持多继承，你只能去修改另一个BaseController……有点悲伤。


##╭(′▽`)╯

通过标题的剧透，我们知道最后的实现跟runtime有关，那么主角也该出场了。
其实就是使用Category + runtime的对象关联。在上面的方案中，解决集成BaseViewController的侵入性的方案就是使用category为UIViewController添加方法，但是category是不能直接使用property保存私有变量的，于是引入runtime的AssociatedObject系列方法，可以动态为对象添加成员变量，这几乎是runtime最基础的应用。

非常简单的，只用到两个方法，其实就是一个Setter和Getter

`id objc_getAssociatedObject(id object, const void *key)`

`void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)`

如果想深入探究一下，有人已经写得挺全面了，可以点击[这篇文章](http://blog.leichunfeng.com/blog/2015/06/26/objective-c-associated-objects-implementation-principle/)

在本例中，用法大概是这样

```objc
#pragma mark - Getter

- (UIView *)loadingView{
    UIView *loadingView = objc_getAssociatedObject(self, &PresnterLoadingViewKey);
    if (!loadingView) {
        loadingView = [[UIView alloc] initWithFrame:self.view.bounds];
        objc_setAssociatedObject(self, &PresnterLoadingViewKey, loadingView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        
        loadingView.backgroundColor = [UIColor whiteColor];
        [loadingView addSubview:self.loadingImageView];
    }
    return loadingView;
}

- (UIImageView *)loadingImageView{
    UIImageView *imageView = objc_getAssociatedObject(self, &PresnterLoadingImageViewKey);
    
    if (!imageView) {
        imageView = [[UIImageView alloc] initWithFrame:
                         CGRectMake(self.view.bounds.size.width / 2 - 100, self.view.bounds.size.height/2 - 80, 200, 150)];
        
        NSMutableArray *tmpArr = [NSMutableArray array];
        for (int i = 0; i <= 80; i++) {
            UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"01-progress00%02d.jpg",i]];
            [tmpArr addObject:image];
        }
        [imageView setAnimationImages:[NSArray arrayWithArray:tmpArr]];
        imageView.animationDuration = 2.0;
        
        objc_setAssociatedObject(self, &PresnterLoadingImageViewKey, imageView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    
    return imageView;
}
```

把实例化放进Getter，这样实现方法可以同上保持不变

```objc
/**
 *  加载视图
 */
- (void)startLoading{
    [self.view addSubview:self.loadingView];
    [self.loadingImageView startAnimating];
}

/**
 *  停止加载并消失
 */
- (void)stopLoading{
    [self.loadingImageView stopAnimating];
    [self.loadingView removeFromSuperview];
}
```

最后只需要在你需要用到这些页面的ViewController引入UIViewController+Presenter.h
然后展示就好
```objc
[self startLoading]; //加载完成后调用 [self stopLoading];
```


具体的代码我写了个demo放在github上，地址在[这里](https://github.com/KinoAndWorld/RuntimePracticeDemo)。

另外实现了空白视图和失败重试视图的功能，跟loading页大同小异。
如此一来，这些公共页面的展示逻辑基本被封装进了category中，而且当我们需要修改展示的页面时，也只需修改文件里面的实现，然后暴露出方法，简单高效。


