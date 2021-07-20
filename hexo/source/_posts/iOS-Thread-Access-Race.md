---
layout: title
title: Swift多线程读写中的内存竞争问题
date: 2021-04-02 15:58:51
tags:
categories: iOS
---

# Swift多线程读写中的内存竞争问题

## 内存竞争
首先，我们有一段这样的代码：
```swift
private var progressMap: [String: Double] = [:]
private let accessQueue = DispatchQueue(label: "com.ThreadRWTest", qos: .default, attributes: .concurrent, autoreleaseFrequency: .inherit, target: .none)

override func viewDidLoad() {
    super.viewDidLoad()
    
    for idx in 0..<1000 {
        accessQueue.async(group: .none, qos: .default, flags: .assignCurrentContext) { [self] in
            let str = "\(idx)"
            progressMap[str] = 0.0
            DispatchQueue.main.async {
                print(str)
            }
        }
    }
}
```

这是一段很常见的代码，我们创建1000个任务（可以粗略理解为创建1000个子线程，当然系统似乎会控制最大线程数量），异步地给progressMap这个字典赋值（写操作），然后切换到主线程展示（这里直接简化为print）。
稍微有点经验的可能一眼就看出了代码的异样：在异步多线程写同一个变量。这是内存异常的罪魁祸首之一。

我们执行以下，以身试法

![WX20210401-174903@2x.png](https://i.loli.net/2021/04/01/25yf1hAsrXDRHKI.png)

噢，果然崩溃了。

但是为什么会崩溃呢？或者说如果想知道崩溃发生的细节该如何调试？

万幸的是，苹果Xcode已经为我们提供了一个非常简单而强大的工具*Thread Sanitizer*
在`edit scheme`选项卡中，勾选中`Diagnostics`中的`Thread Sanitizer`

![1_kBqRpigF7nwCMBK3H3cGKw.jpeg](https://i.loli.net/2021/04/01/RFK1p3dDnaOG75v.jpg)

然后重新运行，可以发现，Xcode会直接在控制台打印出一堆分析日志
```
WARNING: ThreadSanitizer: Swift access race (pid=76516)
  Modifying access of Swift variable at 0x7b5c00000350 by thread T9:
    #0 ViewController.progressMap.modify ViewController.swift (ThreadRWTest:x86_64+0x100002f34)
    #1 closure #1 in ViewController.viewDidLoad() ViewController.swift:28 (ThreadRWTest:x86_64+0x1000041ce)
    #2 partial apply for closure #1 in ViewController.viewDidLoad() <compiler-generated> (ThreadRWTest:x86_64+0x1000044e5)
    #3 thunk for @escaping @callee_guaranteed () -> () <compiler-generated> (ThreadRWTest:x86_64+0x1000047a3)
    #4 _dispatch_client_callout <null>:2 (libdispatch.dylib:x86_64+0x39c7)
    #5 _dispatch_client_callout <null>:2 (libdispatch.dylib:x86_64+0x39c7)

  Previous modifying access of Swift variable at 0x7b5c00000350 by thread T5:
    #0 ViewController.progressMap.modify ViewController.swift (ThreadRWTest:x86_64+0x100002f34)
    #1 closure #1 in ViewController.viewDidLoad() ViewController.swift:28 (ThreadRWTest:x86_64+0x1000041ce)
    #2 partial apply for closure #1 in ViewController.viewDidLoad() <compiler-generated> (ThreadRWTest:x86_64+0x1000044e5)
    #3 thunk for @escaping @callee_guaranteed () -> () <compiler-generated> (ThreadRWTest:x86_64+0x1000047a3)
    #4 _dispatch_client_callout <null>:2 (libdispatch.dylib:x86_64+0x39c7)
    #5 _dispatch_client_callout <null>:2 (libdispatch.dylib:x86_64+0x39c7)
```
我们直接打开issue navigation的tab，然后选中runtime，也可以看到相关的线程信息

![WX20210401-173616@2x.png](https://i.loli.net/2021/04/01/YyU36nKwQhiDsfS.png)

从这张图我们明细可以看到线程3和线程9同时调用了progressMap的modify方法，然后产生了`access race`问题。


## 线程同步

我们现在大概知道问题发生的原因。
那么，遇到这种或者类似的问题应该如何解决呢。

最简单的做法，我们把写操作隔离开，使用栅栏。

```swift
// 将 progressMap[str] = 0.0 修改为
accessQueue.async(group: .none, qos: .default, flags: .barrier, execute: {
    progressMap[str] = 0.0
})
```

重新运行，确实不会崩溃了
想一下barrier的作用，它阻塞了下一个任务，确保之前的任务执行完毕再执行下一个任务。
这里就起到了一个确保写操作是独占的，避免了多个线程同时修改一块内存的隐患。

## 变量加锁
值得注意的是，如果打算通过直接给map变量加锁的话，我们可以想到这样的方式：
```swift
objc_sync_enter(progressMap)
progressMap[str] = 0.0
objc_sync_exit(progressMap)
```

然而遗憾的是，这是不可行的，实际运行还是会崩溃。
通过一些[资料](https://stackoverflow.com/questions/35084754/objc-sync-enter-objc-sync-exit-not-working-with-dispatch-queue-priority-low)，我注意到这里的关键是因为，在swift中，map是值类型的。
因此，objc_sync_enter(progressMap)传递的只是progressMap的拷贝，相当于这些代码完全没生效！
（如果是引用类型应该是可行的，但是objc_sync_enter使用递归锁性能相对较差，建议也少用）

如果想使用锁可以尝试直接使用`pthread_mutex_t`:

```swift
var mutex = pthread_mutex_t()
pthread_mutex_init(&(self.mutex), nil)

let start = CACurrentMediaTime()
for idx in 0..<500 {
    accessQueue.async(group: .none, qos: .default, flags: .assignCurrentContext) { [self] in
        let str = "\(idx)"
        let retCode = pthread_mutex_trylock(&mutex)
        if retCode == 0 {
            progressMap[str] = 0.0
            pthread_mutex_unlock(&mutex)
        }
    }
}
```

或者使用一个自己创建的串行队列也可以达到线程同步的效果。

## 总结

- 多线程同时读写同一变量可能导致内存问题
- 使用`Thread Sanitizer`排查和分析
- 使用`barrier`或者`pthread_mutex_t`线程锁或者串行队列都可以解锁或避免这个问题