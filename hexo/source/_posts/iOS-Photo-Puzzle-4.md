---
title: iOS突发奇想编程系列之 图片拼图(4)
date: 2021-03-23 18:28:01
tags: 
categories: iOS
---

## 第四篇 完成度判断

经过前面的努力，我们现在已经完成了整个拼图的大概框架。在这篇中，我们再稍微完善和补充一下细节，把整个流程做一个总结。

首先，我们用一个数据模型，绑定图片和对应的位置
```swift
class PieceDataModel: NSObject {
    var loc: CGPoint = .zero
    var image = UIImage()
}
```
在开始拖拽碎片的时候，我们追踪这个model，然后在移动结束后，判断当前拖拽碎片与终点位置是否一致，如果一致，那么说明这个方块已经正确移动到了目标。

这里，我们可以做一个”锁定“的操作，便于简化后续的操作。
```swift
    // 拼图摆放记录 n*n
    private var completeStates: [[Int]] = []

    // 在放置碎片完成后调用
    self.checkAdapterBindCorrect(adapter: adapter)
    // 判断游戏是否结束
    if self.checkAllComplete() {
        self.showCompleteAlert()
    }


    /// 判断当前绑定是否正确
    /// - Parameter adapter: PieceDragableAdapter
    private func checkAdapterBindCorrect(adapter: PieceDragableAdapter) {
        // 判断是否落在了正确的格子
        if let dragingLoc = self.dragingModel?.loc, dragingLoc.equalTo(adapter.bindPt) {
            adapter.bindView?.alpha = 1.0
            // 如果正确 则锁定
            completeStates[Int(adapter.bindPt.x)][Int(adapter.bindPt.y)] = 1
        } else {
            adapter.bindView?.alpha = 0.5
            completeStates[Int(adapter.bindPt.x)][Int(adapter.bindPt.y)] = 0
        }
    }
```

当我们拖拽碎片到一个已经"锁定"的格子上时，则直接return。
以及当我们在一个已经"锁定"的格子上拖拽时，也直接return。
这样可以保障当一个碎片已经正确移动时，我们便可以更好专注于未完成的碎片。

同时，得益于`completeStates`，我们只需要判断当`completeStates`的所有元素都为1时，拼图就全部完成了。

```swift
private func checkAllComplete() -> Bool {
    for list in completeStates {
        for state in list {
            if state == 0 { return false }
        }
    }
    return true
}

private func showCompleteAlert() {
    let alert = UIAlertController(title: "恭喜🎇", message: "拼图已完成", preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "👌", style: .default, handler: { [unowned self] _ in
        self.navigationController?.popViewController(animated: true)
    }))
    present(alert, animated: true, completion: nil)
}
```

我们运行一下，看一下最终的效果。

![IMG_5235.PNG](https://i.loli.net/2021/03/23/jxb1ZyGQ3nKlUEa.png)

补全最后一块拼图后：

![IMG_5236.PNG](https://i.loli.net/2021/03/23/ZSmrUbtNIJqfCy9.png)

大功告成！

最后，项目的所有代码我放会在GitHub上，作一个阶段性的成果😆
[https://github.com/KinoAndWorld/PuzzleDemo](https://github.com/KinoAndWorld/PuzzleDemo)


---

写在后面：
虽然现在项目告一段落，但是还有非常多需要完善和扩展的空间。比如UI的完善，碎片的打乱，流程的规范，逻辑的补完。

但毫无疑问的是，这是一次非常有趣的尝试。也让我学习到了许多，吾生有崖而知无涯，共勉之。