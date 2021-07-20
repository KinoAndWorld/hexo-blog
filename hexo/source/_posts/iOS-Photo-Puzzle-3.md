---
title: iOS突发奇想编程系列之 图片拼图(3)
date: 2021-03-23 10:46:55
tags:
categories: iOS
---

## 第三篇 拼接和吸附（手势处理
有了碎片之后，我们可以尝试构造一个新的页面，在这个页面中，我们分为两个部分，碎片放置区，和拼图框区。
琐碎的页面布局和数据粗略完成以下，变成了图下，我们在拼图框直接放个原图的浅透明，当作提示和区域说明。
![piece-arrange.png](https://i.loli.net/2021/03/23/c1CISfVKGpRsPlj.png)

然后，我们要做的，就是把下方区域的碎片通过手势拖拽到上方区域中。
首先想到的是组合UILongPressGesture和PanGesture等组合手势，但是突然发现或许可以直接使用iOS11新推出的Drag&Drog手势来处理。正好学习一下新的技术应用。

![1501122-dc430560f58ee325.png](https://i.loli.net/2021/03/23/AGqg9of75LQSC8H.png)

虽然其实这个手势更大的用途是在于APP之内的数据传输，但是我们可以稍微利用一下，

简单地，我们给下方的listview（这里我用collectionView）添加一下手势
```swift
collectionView.dragInteractionEnabled = true
collectionView.dragDelegate = self
```

然后简单添加处理delegate

```swift
extension PuzzlePlayController: UICollectionViewDragDelegate {
    func collectionView(_ collectionView: UICollectionView, itemsForBeginning session: UIDragSession, at indexPath: IndexPath) -> [UIDragItem] {
        let itemProvider = NSItemProvider(object: pieceImages[indexPath.item])
        let item = UIDragItem(itemProvider: itemProvider)
        self.dragIndexPath = indexPath
        return [item]
    }
    
    func collectionView(_ collectionView: UICollectionView, dragSessionIsRestrictedToDraggingApplication session: UIDragSession) -> Bool {
        return true
    }
    
    func collectionView(_ collectionView: UICollectionView, dragSessionAllowsMoveOperation session: UIDragSession) -> Bool {
        return true
    }
}
```

这个时候，长按collection中的拼图碎片，就可以成功抓取碎片，系统也会自动加上一些效果
![piece-drag.png](https://i.loli.net/2021/03/23/xV9qhBfToHsc8bA.png)

接着，我们直接实现view接收drog手势的处理

```swift
extension PuzzlePlayController: UIDropInteractionDelegate {
    func dropInteraction(_ interaction: UIDropInteraction, canHandle session: UIDropSession) -> Bool {
        return true
    }
    
    func dropInteraction(_ interaction: UIDropInteraction, performDrop session: UIDropSession) {
        let pt = session.location(in: self.view)
        session.loadObjects(ofClass: UIImage.self) { (image) in
            guard let img = image.first as? UIImage else { return }
            
            if self.puzzleContainerView.frame.contains(pt) {
                print("在区域内")
                let imageV = UIImageView(image: img)
                imageV.frame = self.currentHighlightRect(oriPt: pt)
                self.puzzleContainerView.addSubview(imageV)
                
                let adapter = PieceDragableAdapter(view: imageV, image: img)
                self.pieceAdapters.append(adapter)
                
                if let dragIdx = self.dragIndexPath?.item {
                    self.collectionView.performBatchUpdates {
                        self.pieceImages.remove(at: dragIdx)
                        self.collectionView.deleteItems(at: [IndexPath(item: dragIdx, section: 0)])
                    } completion: { (done) in
                        
                    }
                }
            } else {
                print("不在区域内")
            }
        }
    }
    
    func dropInteraction(_ interaction: UIDropInteraction, sessionDidUpdate session: UIDropSession) -> UIDropProposal {
        // Propose to the system to copy the item from the source app
        let pt = session.location(in: self.view)
        print("ori pt = \(pt)")
        if self.puzzleContainerView.frame.contains(pt) {
            let retPt = view.convert(pt, to: self.puzzleContainerView)
            print(retPt)
            
            // 确定在哪个方格
            let rectWidth = puzzleContainerView.frame.width / CGFloat(rowCount)
            let column = floor(retPt.x / rectWidth)
            let row = floor(retPt.y / rectWidth)
            
            let idx = Int(column + row * CGFloat(rowCount))
            
            print("当前落在格子\(column)-\(row)， 对应方格-\(idx)")
            
            lastHightlightFV?.isHidden = true
            tipsFrames[idx].isHidden = false
            lastHightlightFV = tipsFrames[idx]
        }
        
        return UIDropProposal(operation: .move)
    }
}
```

这里省略了`一点点`细节，为了方便处理和提升交互，我们在框内添加了N*N个"边框View"，当拼图移动到对应的区域内，则显示边框，否则隐藏。松开后，同样判断当前移动坐落在哪个格子内，然后放置。

初始化边框代码：
```swift
private func setupPuzzleRect() {
    let pieceSize: CGFloat = 300.0 / CGFloat(rowCount)
    for r in 0..<rowCount {
        for c in 0..<rowCount {
            let frameV = UIView(frame: CGRect(x: CGFloat(c) * pieceSize,
                                                y: CGFloat(r) * pieceSize,
                                                width: pieceSize, height: pieceSize))
            frameV.layer.borderWidth = 4
            frameV.layer.borderColor = UIColor.lightGray.cgColor
            
            frameV.isHidden = true
            tipsFrames.append(frameV)
            
            puzzleContainerView.addSubview(frameV)
        }
    }
}
```

移动到对应方块上的高亮边框：
![piece-drag-in.png](https://i.loli.net/2021/03/23/JLyXhkcfAeFlr9T.png)

同时，我们在创建碎片摆在"棋盘"时，也要同时给碎片View增加拖拽手势，方便二次移动。我们新建一个`PieceDragableAdapter`来管理：

```swift
class PieceDragableAdapter: NSObject {
    var bindView: UIView?
    var bindImage = UIImage()
    var bindPt: CGPoint = .zero
    
    var isVaild: Bool = true
    
    
    init(view: UIView, image: UIImage, loc: CGPoint) {
        super.init()
        
        self.bindView = view
        self.bindImage = image
        self.bindPt = loc
        
        commonInit()
    }
    
    private func commonInit() {
        bindView?.isUserInteractionEnabled = true
        
        let dragInteract = UIDragInteraction(delegate: self)
        bindView?.addInteraction(dragInteract)
        dragInteract.isEnabled = true
    }
}

extension PieceDragableAdapter: UIDragInteractionDelegate {
    func dragInteraction(_ interaction: UIDragInteraction, itemsForBeginning session: UIDragSession) -> [UIDragItem] {
        if !isVaild { return [] }
        let itemProvider = NSItemProvider(object: bindImage)
        
        let item = UIDragItem(itemProvider: itemProvider)
        session.localContext = ("Piece-Frame", self.bindPt)
        
        // 半透明化原图
        self.bindView?.alpha = 0.5
        
        return [item]
    }
    
    func dragInteraction(_ interaction: UIDragInteraction, sessionAllowsMoveOperation session: UIDragSession) -> Bool {
        return true
    }
    
    func dragInteraction(_ interaction: UIDragInteraction, session: UIDragSession, willEndWith operation: UIDropOperation) {
        
    }
}
```

我们再扩展一下`func dropInteraction(_ interaction: UIDropInteraction, performDrop session: UIDropSession)`方法，让其同时处理来自棋盘内部的拖拽

```swift
// 判断来源 如果是移动铺面的
    if let context = session.localDragSession?.localContext as? (String, CGPoint) {
        let fromPt = context.1
        // 移动到新方块
        let newFrame = self.currentHighlightRect(oriPt: pt)
        if let adapter = self.adapterByLoc(pt: fromPt) {
            adapter.bindView?.frame = newFrame
            
            // 更新pt
            adapter.bindPt = self.lastHighlightPt
            // 隐藏方块
            self.lastHightlightFV?.isHidden = true
            
            self.checkAdapterBindCorrect(adapter: adapter)
        }
        return
    }
```