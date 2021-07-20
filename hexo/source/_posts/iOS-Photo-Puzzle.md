---
title: iOS突发奇想编程系列之 图片拼图(1)
date: 2021-03-22 11:36:36
categories: iOS
tags: 
---

## 第一篇 分析碎片切图

拼图是一个老少咸宜的益智游戏，相信大家都玩过。有一天我突然看到街上的自定义照片的拼图，想到我能不能也做一个能自动把一张图片分割成拼图碎片，甚至还能手动拼回来的APP呢。
听起来有点意思，闲暇之余，我也稍微认真思考了一下……

### 分析
首先，我们看下一个完整的拼图长啥样

![stock-vector-jigsaw-icon-241085644.jpg](https://i.loli.net/2021/03/22/iTBjAXPytmWKv1p.jpg)


可以看到，即使在这个相当简单的拼图中，拼图碎片的种类也非常多，我找了另一张图，可稍管中窥豹。
![stock-vector-set-of-black-and-white-puzzle-pieces-isolated-on-white-background-vector-illustration-1171424941.jpg](https://i.loli.net/2021/03/22/r1KGBezn2Y7PmaM.jpg)


从这张图中我们发现，一个拼图碎片其实就是一张大图通过比例均等切割后的矩形，再构造【凹】与【凸】两种形态，由此形成的排列组合。

因为是矩形，所以是4个面，而每个面都有3种状态，`凸`、`凹`和`平`，
一共是3的4次方。

当然，在实际拼图中，我们会制订一些规则，实际的碎片样式会少很多。这个是后话，暂且不表，我们先来研究一下，在iOS中如何切出不规则的拼图碎片。

#### 拼图切割
第一步，我们先将大图平均分成多个小图。
这一步非常简单，我们只需要写一个扩展，就能将任意的一个view通过给定的frame裁剪出一个新的UIImage。
```swift
extension UIView {
    func asImage(rect: CGRect) -> UIImage {
        let renderer = UIGraphicsImageRenderer(bounds: rect)
        return renderer.image { rendererContext in
            layer.render(in: rendererContext.cgContext)
        }
    }
}
```
看起来很简单，但是，这里有一个点我们需要注意的是，例如我们需要切割2x2的拼图方块，我们想当然地将图片切成宽高/2的4张图片，但是不要忘记我们的拼图是包括`凸`的形态，倘若使用这种均分的切割，无法很好地对碎片进行统一的处理。因此，我们会加大裁切的区域，我们以'凸出的长度'为半径，扩展截图区域，大概如图所示：
![WX20210119-170425@2x.png](https://i.loli.net/2021/03/22/q7h3guvDPxpTURM.png)

示例代码：
```swift
let canverWidth: CGFloat = imageView.frame.width / 2.0
        
for idx in 0...3 {
    let row: Int = idx / 2
    let column: Int = idx % 2
    
    let image = imageView.asImage(rect: CGRect(x: canverWidth * CGFloat(column) - PieceConfigure.holeRadius * 2,
                                                y: canverWidth * CGFloat(row) - PieceConfigure.holeRadius * 2,
                                                width: canverWidth + PieceConfigure.holeRadius * 4,
                                                height: canverWidth + PieceConfigure.holeRadius * 4))
    
    let imageV = UIImageView(image: image)
    imageV.backgroundColor = .lightGray
    imageV.contentMode = .scaleAspectFill
    view.addSubview(imageV)
    
    imageV.frame = CGRect(x: (canverWidth + 40) * CGFloat(column) + 30, y: (canverWidth + 40) * CGFloat(row) + 500,
                            width: imageView.frame.width / 2.0 + PieceConfigure.holeRadius * 2,
                            height: imageView.frame.height / 2.0 + PieceConfigure.holeRadius * 2)
}
```
![rect-slice.png](https://i.loli.net/2021/03/22/4E76krn3TGDlfhe.png)

可以看到，这个时候我们切的4张图，是相互重叠且带有边距的。这为我们下一步的拼图抠图，做足了准备。

#### 凹凸不规则切割
根据前文所说我们看到，我们已经成功实现了一张大图的多图分割，并且应对各种情况留了裁切区域。前面我们也说过，每个面只有3种情况，平面、凹和凸，平面就是什么也不用做，我们主要看下凹和凸的实现。

我们可以简单地用一个圆形的图形运算来模拟这个过程：

![WX20210120-164821@2x.png](https://i.loli.net/2021/03/22/lIWCvdJZNjLEczT.png)

凹：通过对矩形和圆形进行顶层交集即可

![WX20210120-164831@2x.png](https://i.loli.net/2021/03/22/yj7KF6NC5sbzWwt.png)

凸：通过图形并集可得

![WX20210120-164849@2x.png](https://i.loli.net/2021/03/22/kYuUiLnbaSpTWAK.png)

在iOS的实现中，我通过直接使用UIBezierPath编辑路径，使用绘制圆弧的方式达到效果。
以一条边为例，绘制的代码如下：
```swift
let path = UIBezierPath()
path.move(to: CGPoint(x: startX, y: startY))

// top
path.addLine(to: CGPoint(x: (width + startX) / 2  - PieceConfigure.holeRadius, y: startY))
if topDraw == .outside {
    path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: startY - PieceConfigure.holeRadius),
                radius: PieceConfigure.holeRadius,
                startAngle: CGFloat(180.0).toRadians(),
                endAngle: CGFloat(0.0).toRadians(),
                clockwise: true)
} else if topDraw == .inner {
    path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: startY + PieceConfigure.holeRadius),
                radius: PieceConfigure.holeRadius,
                startAngle: CGFloat(180.0).toRadians(),
                endAngle: CGFloat(0.0).toRadians(),
                clockwise: false)
}
path.addLine(to: CGPoint(x: (width+startX) / 2 + PieceConfigure.holeRadius, y: startY))
path.addLine(to: CGPoint(x: width, y: startY))
```


### 封装一下
OK，让我们整理一下，新建一个struct，命名为PieceSlicer，我们可以把切片的过程封装一下
```swift
/// 拼图切片机
struct PieceSlicer {
    enum PathDrawType {
        case line
        case outside
        case inner
    }
    
    enum Direction {
        case left
        case right
        case top
        case bottom
    }
    
    var leftDraw: PathDrawType = .line
    var topDraw: PathDrawType = .line
    var rightDraw: PathDrawType = .line
    var bottomDraw: PathDrawType = .line

    var holeRadius: CGFloat = 6.0
    
    init(left: PathDrawType, top: PathDrawType, right: PathDrawType, bottom: PathDrawType) {
        leftDraw = left
        topDraw = top
        rightDraw = right
        bottomDraw = bottom
    }
    
    func draw(in rect: CGRect) -> UIBezierPath? {
        let startX = rect.origin.x
        let startY = rect.origin.y
        let width = rect.size.width
        let height = rect.size.height
        
        let path = UIBezierPath()
        path.move(to: CGPoint(x: startX, y: startY))
        
        // 因为绘图区域上下颠倒，这里我们直接把上和下的配置交换赋值
        // top
        path.addLine(to: CGPoint(x: (width + startX) / 2  - holeRadius, y: startY))
        if bottomDraw == .outside {
            path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: startY - holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(180.0).toRadians(),
                        endAngle: CGFloat(0.0).toRadians(),
                        clockwise: true)
        } else if bottomDraw == .inner {
            path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: startY + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(180.0).toRadians(),
                        endAngle: CGFloat(0.0).toRadians(),
                        clockwise: false)
        }
        path.addLine(to: CGPoint(x: (width+startX) / 2 + holeRadius, y: startY))
        path.addLine(to: CGPoint(x: width, y: startY))
        
        // right
        path.addLine(to: CGPoint(x: width, y: height / 2))
        if rightDraw == .outside {
            path.addArc(withCenter: CGPoint(x: width + holeRadius, y: height / 2 + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(270.0).toRadians(),
                        endAngle: CGFloat(90.0).toRadians(),
                        clockwise: true)
        } else if rightDraw == .inner {
            path.addArc(withCenter: CGPoint(x: width - holeRadius, y: height / 2 + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(270.0).toRadians(),
                        endAngle: CGFloat(90.0).toRadians(),
                        clockwise: false)
        }
        path.addLine(to: CGPoint(x: width, y: height / 2 + holeRadius * 2))
        path.addLine(to: CGPoint(x: width, y: height))
        
        
        // bottom
        path.addLine(to: CGPoint(x: (width+startX) / 2 + holeRadius, y: height))
        if topDraw == .outside {
            path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: height + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(0.0).toRadians(),
                        endAngle: CGFloat(180.0).toRadians(),
                        clockwise: true)
        } else if topDraw == .inner {
            path.addArc(withCenter: CGPoint(x: (width+startX) / 2, y: height - holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(0.0).toRadians(),
                        endAngle: CGFloat(180.0).toRadians(),
                        clockwise: false)
        }
        path.addLine(to: CGPoint(x: (width+startX) / 2.0 - holeRadius, y: height))
        path.addLine(to: CGPoint(x: startX, y: height))
        
        
        // left
        path.addLine(to: CGPoint(x: startX, y: height / 2 + startY))
        if leftDraw == .outside {
            path.addArc(withCenter: CGPoint(x: startX - holeRadius, y: height / 2 + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(90.0).toRadians(),
                        endAngle: CGFloat(270.0).toRadians(),
                        clockwise: true)
        } else if leftDraw == .inner {
            path.addArc(withCenter: CGPoint(x: startX + holeRadius, y: height / 2 + holeRadius),
                        radius: holeRadius,
                        startAngle: CGFloat(90.0).toRadians(),
                        endAngle: CGFloat(270.0).toRadians(),
                        clockwise: false)
        }
        path.addLine(to: CGPoint(x: startX, y: height / 2))
        // done
        path.close()
        path.usesEvenOddFillRule = true
        
        return path
    }
}
```

通过简单的封装，我们已经完成了拼图切片状态的绘制代码，PathDrawType和Direction分别对应我们前文提到的切面类型和四个边，并且也是一一对应的。我们可以试一下：

现在构造一个简单的2x2的拼图，我们需要描述4个碎片的结构
```swift
PieceSlicer(left: .line, top: .line, right: .outside, bottom: .inner),
PieceSlicer(left: .inner, top: .line, right: .line, bottom: .outside),
PieceSlicer(left: .line, top: .outside, right: .inner, bottom: .line),
PieceSlicer(left: .outside, top: .inner, right: .line, bottom: .line),
```
运行效果如下：

![piece-slice.png](https://i.loli.net/2021/03/22/n2govAhsTMw6c9e.png)

喔，有点像模像样了。就这样，我们的拼图切割已经初具雏形了。不过这样还很不灵活，接下来我们再看下，如何更自动和动态地生成拼图碎片。