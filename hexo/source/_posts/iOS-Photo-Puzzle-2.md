---
title: iOS突发奇想编程系列之 图片拼图(2)
date: 2021-03-23 10:45:08
categories: iOS
tags:
---

## 第二篇 分析拼图分布
接上文，我们的拼图切割已经初步实现了。不过我们注意到，我们现在是需要一个一个去描述拼图碎片的状态，这样非常容易出错而且不灵活，更重要的是，如果我们需要构造一个10x10的拼图，那我们需要创建100个PieceSlicer并且分别描述好4个边的情况，So，这是非常不人道的。

因此我们需要写一个简单的碎片生成算法。
我们首先需要明确一下，一个拼图的碎片，需要遵循的约束，然后大致提炼出以下几条规则：
- 规则1：最外层的边都是平面
- 规则2：没有约束的情况下随机凹凸
- 规则3：一个碎片凹或凸的边不能超过3个
- 规则4：相邻碎片的凹凸需要契合
最后，遇事不决，暴力穷举。
我们遍历NxN个碎片，然后逐一检查边界与契合情况，分别赋值，剩余的可凹可凸我们随机赋值即可。

```swift
// n x n 的方阵
var map: [[PieceSlicer]] = []

// 规则1：最外层的边都是平面
// 规则2：没有约束的情况下随机凹凸
// 规则3：一个碎片凹或凸的边不能超过2个
// 规则4：相邻碎片的凹凸需要契合
mutating func construct() -> [PieceSlicer] {
    var pieceList: [PieceSlicer] = []
    
    let raduis: CGFloat = pieceFillRadius
    
    for i in 0..<row {
        for j in 0..<row {
            var slicer = PieceSlicer(left: .none, top: .none, right: .none, bottom: .none)
            slicer.holeRadius = raduis
            var lineBox: [PieceSlicer.PathDrawType] = [.inner, .outside]
            
            // 根据前置位 匹配凹凸类型
            if j == 0 {
                slicer.leftDraw = .line
            } else {
                // 左边有方块 直接取反
                let leftSilce = map[i][j-1]
                slicer.leftDraw = leftSilce.rightDraw.oppose
            }
            
            if j == row - 1 { slicer.rightDraw = .line }
            if i == 0 {
                slicer.topDraw = .line
            } else {
                let topSilce = map[i-1][j]
                slicer.topDraw = topSilce.bottomDraw.oppose
            }
            
            if i == row - 1 { slicer.bottomDraw = .line }
            
            // 填充剩余边
            slicer.randomSetDirections(box: &lineBox)
            
            print(slicer.description())
            pieceList.append(slicer)
            map[i][j] = slicer
        }
    }
    return pieceList
}

```

另外需要注意的是，我们的拼图碎片大小也与碎片数量呈反比，所以我们前面提到的凹凸半径，需要根据碎片数量或者碎片大小进行动态调整。

好了，我们调整一下布局代码，重新生成。
我们分别试一试4x4个碎片 和 6x6个碎片的效果：

N=4
![piece-n4.png](https://i.loli.net/2021/03/23/oCJ4wQNTAbs8Sgp.png)

N=6
![piece-n6.png](https://i.loli.net/2021/03/23/iKda1wpg2FeA5Nt.png)

鹅妹子嘤！看起来还不错~
拼图的碎片生成到此暂时告一段落，接下来我们尝试把生成的碎片打乱并且添加手势，实现游戏的拼图过程。

`另外，由于当前碎片生成算法显而易见地低效（O(n²)），在生成的碎片越多的情况下效率越低，因此如果需要大量生成拼图碎片，需要优化算法或者选择其他算法。`