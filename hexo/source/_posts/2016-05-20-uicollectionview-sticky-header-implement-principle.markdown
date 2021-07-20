---
layout: post
title: "在UICollectionView上实现StickyHeader效果"
date: 2016-05-20 16:25:43 +0800
comments: true
categories: iOS
---

前几天在用UICollectionView实现组合式布局的时候，想到如果在CollectionView上要求sticky header（也就是header自动悬停置顶）的效果应当如何实现。
我们都知道如果是在UITableView上实现非常简单，只要设置style为group然后多个section，headerView就会自动吸附在顶部。而collectionView虽然功能强大得多，但是原生并没有支持直接设置这个效果，所以需要自己实现。


作为一个懒人，首先想到的自然是站在巨人的肩膀上，[CSStickyHeaderFlowLayout](https://github.com/jamztang/CSStickyHeaderFlowLayout)是一个比较知名的CollectionView布局控件，它支持sticky header效果以及头部的视差滚动效果，效果很赞。但是感觉稍微重了些，需要做更多的设置以及修改基类等。

然后我又找到了一篇文章，虽然已经是3年前的文章，但是写得非常简明清晰，而且阅读完有豁然开朗的感觉。


文章很短，全文如下：[http://blog.radi.ws/post/32905838158/sticky-headers-for-uicollectionview-using](http://blog.radi.ws/post/32905838158/sticky-headers-for-uicollectionview-using)

核心原理就是三句话：

- `The header should be positioned so it can never go further up than one header height above the first cell in the section.`
- `The header should be positioned so it can never go further down than one header header height above the lower bounds of the last cell in the section.`
- `The header should be positioned so it usually stays around the top edge, referencing the content offset of the collection view.`


代码也很简单，只需要在UICollectionViewFlowLayout实现`layoutAttributesForElementsInRect`和`shouldInvalidateLayoutForBoundsChange`两个方法。

接着我附上文中源代码以及自己写的注释，穿插了一些对文章初略的翻译。

```objc
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect {

    /**
     *  返回当前显示区域的所有布局信息
     */
    NSMutableArray *answer = [[super layoutAttributesForElementsInRect:rect] mutableCopy];
    UICollectionView * const cv = self.collectionView;
    CGPoint const contentOffset = cv.contentOffset;

    NSMutableIndexSet *missingSections = [NSMutableIndexSet indexSet];

    /**
     *  找出所有UICollectionElementCategoryCell类型的cell
     */
    for (UICollectionViewLayoutAttributes *layoutAttributes in answer) {
        if (layoutAttributes.representedElementCategory == UICollectionElementCategoryCell) {
            [missingSections addIndex:layoutAttributes.indexPath.section];
        }
    }
    /**
     *  再从里面删除所有UICollectionElementKindSectionHeader类型的cell
     */
    for (UICollectionViewLayoutAttributes *layoutAttributes in answer) {
        if ([layoutAttributes.representedElementKind isEqualToString:UICollectionElementKindSectionHeader]) {
            [missingSections removeIndex:layoutAttributes.indexPath.section];
        }
    }

    /**
     *  默认情况下，为missingSections手动插入attributes，应该是为rect外的section生成attributes
     */
    [missingSections enumerateIndexesUsingBlock:^(NSUInteger idx, BOOL *stop) {

        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:0 inSection:idx];

        UICollectionViewLayoutAttributes *layoutAttributes = [self layoutAttributesForSupplementaryViewOfKind:UICollectionElementKindSectionHeader atIndexPath:indexPath];

        [answer addObject:layoutAttributes];

    }];


    for (UICollectionViewLayoutAttributes *layoutAttributes in answer) {

        /**
         *  从answer中储存的布局信息中，针对UICollectionElementKindSectionHeader...
         */
        if ([layoutAttributes.representedElementKind isEqualToString:UICollectionElementKindSectionHeader]) {

            NSInteger section = layoutAttributes.indexPath.section;
            NSInteger numberOfItemsInSection = [cv numberOfItemsInSection:section];

            /**
             *  为什么需要firstCellIndexPath和lastCellIndexPath呢？
             *  header应当保持着距离本section第一个cell的最大距离，简单来说就是header在置顶之前要贴着cell
             *  同理，header应当保持着与最后一个cell的最小距离，
             *  最后，在不违反上面两则约束的前提下，通过collection view的offset与header的高度来使header处于 origin.y = 0 的状态。
             */
            NSIndexPath *firstCellIndexPath = [NSIndexPath indexPathForItem:0 inSection:section];
            NSIndexPath *lastCellIndexPath = [NSIndexPath indexPathForItem:MAX(0, (numberOfItemsInSection - 1)) inSection:section];

            /**
             *  针对当前layoutAttributes的section， 找出第一个和最后一个普通cell的位置
             */
            UICollectionViewLayoutAttributes *firstCellAttrs = [self layoutAttributesForItemAtIndexPath:firstCellIndexPath];
            UICollectionViewLayoutAttributes *lastCellAttrs = [self layoutAttributesForItemAtIndexPath:lastCellIndexPath];

            /**
             *  获取当前处理header的高度和位置，然后通过firstCellAttrs和lastCellAttrs确定header是否置顶
             */
            CGFloat headerHeight = CGRectGetHeight(layoutAttributes.frame);
            CGPoint origin = layoutAttributes.frame.origin;
            origin.y = MIN(
                           MAX(
                               contentOffset.y,
                               (CGRectGetMinY(firstCellAttrs.frame) - headerHeight)
                               ),
                           (CGRectGetMaxY(lastCellAttrs.frame) - headerHeight)
                           );

            layoutAttributes.zIndex = 1024;
            layoutAttributes.frame = (CGRect){
                .origin = origin,
                .size = layoutAttributes.frame.size
            };
        }
    }
    return answer;
}

- (BOOL) shouldInvalidateLayoutForBoundsChange:(CGRect)newBound {
    return YES;
}
```

感觉又涨了姿势，撒花。