---
title: Swift项目初体验琐记（1）
date: 2017-04-04 00:54:50
tags: Swift
categories: iOS
---

虽说从Swift刚出生就一直有关注，但是由于ABI不稳定以及第三方支持还不够好等等原因，一直没有用于实际的真实项目中，直到3.0之后感觉时机成熟了，刚好最近有一个比较小的项目，于是有了一次实际项目的初体验，也顺便写下一些总结。（总体内容会比较浅显，惭愧了

### Swift与Objective-C常见写法差异

1. 判断字符串相等、==与===的区别
	在oc中，我们已经习惯了使用`isEqualToString`方法来判断两个字符串是否相等，似乎也不觉得有什么问题，这是因为oc使用`==`来统一引用的比较与值类型的比较，但是NSString在内存模型上比较特殊，以至于oc中使用了一个专门的比较方法，而在其他许多语言中只需要用`==`对比字符串就好。Swift改进了这一点，顺带一提，Swift中使用`==`来比较值类型是否相等，而使用`===`来判断引用类型是否相等。
	
2. 数组遍历
	在oc中，我喜欢使用enumerateObjectsUsingBlock方法遍历数组并进行操作，大概是这个样子的：
	
	```objc
	NSArray *list = @[@1,@2,@3,@4,@5];
	[list enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
		NSLog(@"obj = %@, idx = %d", obj , (int)idx);
	}];
	```
	但是Swift自然不会还存在如此oc模样的方法，相替代的，Swift可以用foreach方法：
	
	```swift
	let list = [1,2,3,4,5]
	list.forEach { number in
		print(number)
	}
	```
	但是如果我们想要像oc一样同时获取元素和下标的话，就要换一种写法了：
	
	```swift
	let list = [1,2,3,4,5]
	for (idx, number) in list.enumerated() {
		print("idx is \(idx) and number is \(number)")
	}
	```
	无论如何，至少看起来比oc是要简洁多了。
	
	另外，Swift的一个令人激动的特色就是内嵌一些函数式编程范式，因此如果我们要对数组逐个进行操作的话用`map`方法是再方便不过。配合上一些语法糖，非常之简单而且直观，比如我们把上面的list的每个元素乘以2：
	
	```swift
	print(list.map { $0 * 2 })
	```
	还有更多的基本用法如filter和reduce等等这里就不赘述了，很多[相关教程]("https://yrq110.gitbooks.io/some_ios_tutorials_with_swift/content/Introduction%20to%20Functional%20Programming%20in%20Swift.html")。
	
3. 判断或获取类名字符串
	在oc中，获取一个对象的类型是很简单的，只需要调用class类型方法就好，比如`[AClass class]`，如果判断某个对象是否属于这个类型，则使用`isKindOfClass`方法：`[self isKindOfClass:[AClass class]]`，如果需要获取一个实例类名的字符串，则`NSStringFromClass([AClass class])`。而在Swift中，获取一个类的class很简单，只需要`AClass.self`就可以了，但是，如果你想获取一个实例对象的类名字符串，则需要这样写`String(describing: type(of: aInstance))`，额，总觉得有点奇怪。
	

### Swift特性的一些实践经验总结

1. 要说在Swift中最有特色体验最明显的，应该算可选类型了，诚然，可选类型可以帮助我们构建更安全更可靠的app，但是毫无疑问，可选类型中各种转换和拆包让人觉得略烦，特别是我这样的新手，稍不注意就是报错或者没有得到符合预期的值。然后在stackoverflow上看相关的讨论的时候，发现一个很有趣的小方法：

	```swift
	extension Optional {
		func valueOrDefault<T>(defaultValue: T) -> T {
			switch(self) {
			case .none:
				return defaultValue
			case .some(let value):
				return value as! T
			}
		}
	}
	```
	这是一个Optional类的扩展，用了泛型，代码就不解释了，很清晰。用上这个类别之后，我们可以方便的、安全的写这样的代码：
	
	```swift
	var a: Int?
	//a = 1
	let b = a.valueOrDefault(defaultValue: 2)
	print("b = \(b)")
	```
	
	在不知道a是否有确切值的情况下，就可以将a安全的赋值给b，通过提供默认值的方式，简直赞。在这个例子中，如果把注释的那行取消注释，b则为1，在大量使用可选类型的基础上，使用这个小技巧可以避免使用一堆堆的诸如`if let b = a as? Int`的代码。
	
2. Swift与强大的枚举
	在oc或者说c中，枚举的用途也许仅仅是一个*同数据类型的常量集合*，但是在Swift中，枚举的能力被大大扩充，甚至完全可以替代简单的class。而且在一些方面的最佳实践中，枚举的出场率也相当高，比如notification.name。先不说有没有必要这样做，在我的实践中，发现Swift的枚举确实是一个很方便而且简化逻辑的玩意儿，还是举个栗子吧，比如我们有一个item，包含一个有3种状态的state变量，我们还需要根据这个state输出对应的颜色和说明文字等：

	```swift
	struct Item {
		enum ItemState {
			case success
			case fail
			case ready
			
			//inner function
			func showColor()->UIColor {
				switch self {
					case .success: return UIColor.green
					case .fail: return UIColor.red
					case .ready: return UIColor.gray
				}
			}
			
			func descStr()->String {
				return "[output: \(self)]"
			}
		}
		var name = "example"
		var state = ItemState.ready
	}
	```
	直接在enum中写function有什么好处呢？可以免去很多的判断，从而直接绑定输出，同时也减少了出错的机会，在这个例子中，我们可以如下调用：
	
	```swift
	var item = Item()
	print("now state is = \(item.state.descStr()) , color is \(item.state.showColor())")
		
	item.state = .success
	print("now state is = \(item.state.descStr()) , color is \(item.state.showColor())")
		
	item.state = .fail
	print("now state is = \(item.state.descStr()) , color is \(item.state.showColor())")
	```
	我们可以任意改变state，然后输出当前state对应的各种绑定变量，把逻辑内聚在enum中，外部的调用会非常顺畅。
	当然，这其实算是最基础的用法了，关于enum还有更多高级且高端的用法，可以参考[Swift 中枚举高级用法及实践]("http://swift.gg/2015/11/20/advanced-practical-enum-examples/")

3. Swift中的反射
	从现状来看，Swift的动态性似乎是不如Objective-C的，某些诸如方法交换和动态绑定似乎底层用的也还是Objective-C的runtime。但是好在Swift还是有一个反射的替代方案，比较常用的一个地方就是，在需要保存一个NSObject对象到本地时，必须实现coder的相关方法，默认的做法是需要你把class内每个属性都分别写一个转换，非常麻烦而且容易出错，在Objective-C中可以通过运行时获取所有成员变量和类型来做这个事情，相对应的，Swift可以用Mirror：
	
	```swift
	convenience required init?(coder aDecoder: NSCoder) {
        self.init()

        for child in Mirror(reflecting: self).children {
            if let key = child.label {
                setValue(aDecoder.decodeObject(forKey: key), forKey: key)
            }
        }
    }
    
    func encode(with aCoder: NSCoder) {
        for child in Mirror(reflecting: self).children {
            if let key = child.label {
                aCoder.encode(value(forKey: key), forKey: key)
            }
        }
    }
	```
	还是比较方便快捷的，我们可以再优化一下，把这个放进NSObject的extension里，这样所有需要序列化的object就可以不用再繁复地一个个写了  :)
	
下一篇我会总结一下目前用到的常用第三方框架和组合用法。