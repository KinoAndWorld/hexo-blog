---
layout: post
title: "iOS开发中一些常量数值"
date: 2015-03-18 14:41:02 +0800
comments: true
categories: iOS
---

###先说CGFloat

*几乎众所周知CGFloat其实是float和double的大一统，32位与64位浮点的联姻*

那么具体有什么不同？先来看看  CGFloat的一些常量，
    CGFLOAT_TYPE
    CGFLOAT_MAX
    CGFLOAT_MIN
    CGFLOAT_DEFINED
    CGFLOAT_IS_DOUBLE
好奇心使然，看了下源码

```objc
/* Definition of `CGFLOAT_TYPE', `CGFLOAT_IS_DOUBLE', `CGFLOAT_MIN', and
   `CGFLOAT_MAX'. */
#if defined(__LP64__) && __LP64__
# define CGFLOAT_TYPE double
# define CGFLOAT_IS_DOUBLE 1
# define CGFLOAT_MIN DBL_MIN
# define CGFLOAT_MAX DBL_MAX
#else
# define CGFLOAT_TYPE float
# define CGFLOAT_IS_DOUBLE 0
# define CGFLOAT_MIN FLT_MIN
# define CGFLOAT_MAX FLT_MAX
#endif

/* Definition of the `CGFloat' type and `CGFLOAT_DEFINED'. */

typedef CGFLOAT_TYPE CGFloat;
#define CGFLOAT_DEFINED 1
```

然后很清晰地看到
    CGFLOAT_TYPE         64位下是double 否则float
    CGFLOAT_MAX          64位下是double的max 否则float的max
    CGFLOAT_MIN          64位下是double的min 否则float的min
    CGFLOAT_DEFINED      就是1 为定义而定义，暂时不知道用来判断什么
    CGFLOAT_IS_DOUBLE    64位下是1 否则0

_(:з」∠)_我是不是很无聊

还有一个，当一个浮点型除以0的时候，有的语言会抛出异常，而oc会返回一个“非正常”的值+INF，我们可以用INFINITY常量判断。ex:(aFloat== INFINITY)

###NSInteger
类似的， NSInteger也是囊括了int和long的大一统（还有NSUInteger，多个unsigned），定义跟上面的浮点型是一个模子的。
下面说说我踩过的一个坑。

在很久很久以前，我还是习惯用C语言的变量类型int，在5S出来以前大家相安无事，直到有一天一个bug袭击了我。

代码是这样的：

```
    NSArray *array = [NSArray array];
    int foundIndex = [array indexOfObject:@""];
    if (NSNotFound == foundIndex) {
        NSLog(@"not found,handle it");
    }
```
为什么iPhone4s，5之流都没问题，到5s上就不对了呢。

刚准备烧柱香拜一拜的时候，我点开了NSNotFound的定义，发现它==NSIntegerMax，而NSIntegerMax==LONG_MAX。

64位下，foundIndex因为超过表示范围会被截断成-1，因此它!=NSNotFound。
 
类似的还有很多，我们应该多多jump到源码，即使只是头文件，也能发现关于Objective-C的一些底层细节。