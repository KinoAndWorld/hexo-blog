---
layout: post
title: "打造一个直观易复用的iOS设置菜单页"
date: 2016-04-05 16:09:14 +0800
comments: true
categories: iOS
---

几乎每一个App都有一个设置菜单页，而且他们几乎都长这样：

![img](https://upload-images.jianshu.io/upload_images/25038-d7705c74260a493e.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/320/format/webp)

##反面教材

最简单最原始的做法是这样的：

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
	UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"
	                                                        forIndexPath:indexPath];

	switch (indexPath.row) {
	    case 0:
	        // configure cell
	        break;
	    case 1:
	        // configure cell
	        break;
	    case 2:
	        // configure cell
	        break;
	}
	return cell;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    
    switch (indexPath.row) {
        case 0:
            // click cell 0
            break;
        case 1:
            // click cell 1
            break;
        case 2:
            // click cell 2
            break;
    }
}
```

这样做不仅违反了DRY原则，在菜单项比较多的情况下烦不胜烦，而且一旦需要修改某个菜单项的内容，或者插入或者删除，修改起来都是非常容易出错的，换句话说，可维护性非常差。


##抽象、依赖转移

通过观察我们可以发现，这些列表的结构非常相似，那么很自然地我们可以想到建立一个通用的模型来表示一个菜单.

```objc KOSettingItem.h

@interface KOSettingItem : NSObject

@property (copy, nonatomic) NSString *title;
@property (strong, nonatomic) UIImage *imageIcon;
@property (assign, nonatomic) UITableViewCellAccessoryType accessoryType;
@property (copy, nonatomic) void(^handleCallback)();

+ (instancetype) itemWithTitle:(NSString *)title
                          icon:(UIImage *)image;

+ (instancetype) itemWithTitle:(NSString *)title
                          icon:(UIImage *)image
                         block:(void(^)())handle;

- (id)initWithTitle:(NSString *)title
               icon:(UIImage *)image
              block:(void(^)())handle;

	
@end	
```

```objc KOSettingItem.m

	@implementation KOSettingItem

	+ (instancetype) itemWithTitle:(NSString *)title
	                          icon:(UIImage *)image{
	    return [[self alloc] itemWithTitle:title icon:image];
	}
	
	+ (instancetype) itemWithTitle:(NSString *)title
	                          icon:(UIImage *)image
	                          block:(void(^)())handle{
	    KOSettingItem *item = [[self alloc] initWithTitle:title icon:image block:handle];
	    return item;
	}
	
	- (id)initWithTitle:(NSString *)title
	               icon:(UIImage *)image
	              block:(void(^)())handle{
	    if (self = [super init]) {
	        self.title = title;
	        self.imageIcon = image;
	        self.handleCallback = handle;
	        self.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
	    }
	    return self;
	}
	
	@end
	
```

一个简单的模型就这样建好了，其中我们用block写了一个handleCallback用来处理表格的点击事件，接着我们来应用到TableView中，首先新增一个property `@property (strong, nonatomic) NSArray *settingItems;`

然后在ViewDidLoad中

```objc
	KOSettingItem *item1 = [KOSettingItem itemWithTitle:@"菜单1" icon:[UIImage imageNamed:@"Carrot"]];
    [item1 setHandleCallback:^{
        NSLog(@"点击菜单1");
    }];
    
    KOSettingItem *item2 = [KOSettingItem itemWithTitle:@"菜单2" icon:[UIImage imageNamed:@"Owl"]];
    [item1 setHandleCallback:^{
        NSLog(@"点击菜单2");
    }];
    
    KOSettingItem *item3 = [KOSettingItem itemWithTitle:@"菜单3" icon:[UIImage imageNamed:@"Rubber-Duck"]];
    [item1 setHandleCallback:^{
        NSLog(@"点击菜单3");
    }];
    
    KOSettingItem *item4 = [KOSettingItem itemWithTitle:@"菜单4" icon:[UIImage imageNamed:@"Snowman"]];
    [item1 setHandleCallback:^{
        NSLog(@"点击菜单4");
    }];

    self.settingItems = @[item1, item2, item3, item4];
```

最后修改一下TableView的Datasource和Delegate

```objc
	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
	    return self.settingItems.count;
	}
	
	- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
	    return 44;
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
	    
	    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"
	                                                            forIndexPath:indexPath];
	    
	    KOSettingItem *item = self.settingItems[indexPath.row];
	    cell.textLabel.text = item.title;
	    cell.imageView.image = item.imageIcon;
	    cell.accessoryType = item.accessoryType;
	    
	    return cell;
	}
	
	- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
	    [tableView deselectRowAtIndexPath:indexPath animated:YES];
	    
	    KOSettingItem *item = self.settingItems[indexPath.row];
	    if (item.handleCallback) {
	        item.handleCallback();
	    }
	}
```

我们发现再也不需要冗长的switch语句，如果需要增删菜单项，只需要修改settingItems里面的item顺序，并且由于点击事件绑定到KOSettingItem里面，也不需要担心事件与item不对应的情况，除非有特殊的样式需求，我们可以几乎不用再更改UITableViewDataSource与UITableViewDelegate的实现。


##尾声

文章只是提供了一个思路，而且出于简明的目的KOSettingItem的模型非常简单，实际上你可以通过添加变量达到更多的可定制化效果，比如分割线的offset，自定义的accessoryView等等，完整的demo可以戳[这里](https://github.com/KinoAndWorld/MenuSettingViewDemo)。

运行一下，最后的结果如图，虽然还是平淡无奇 ：）
 
![img](https://upload-images.jianshu.io/upload_images/25038-9ac9bdb88316c0b9.png?imageMogr2/auto-orient/strip|imageView2/2/w/320/format/webp)


最后吐槽一下自己，真的好久没写博客了_(:з」∠)_