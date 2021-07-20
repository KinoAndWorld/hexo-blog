---
title: iOS中使用WKWebview支持h5跳转支付客户端
date: 2018-07-23 18:24:08
tags:
categories: iOS
---

大致的业务场景是这样的：我们的客户端APP本身不包含支付SDK，但是在APP内打开的HTML5是包含了第三方支付的，而且在Safari内是可以正常调起支付宝/微信客户端进行支付的，然而在APP的webview内打开同样的URL则毫无反应。
原因大致是支付宝/微信的h5支付sdk没有对客户端支持，当然也存在一些系统的限制。
现在就来解决一下这个问题。

### 柳暗

经过稍微的查询和参考，解决方案其实非常简单，只需要在`WKWebView`的代理方法`func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void)`内监听微信/支付宝的特定前缀URL，然后使用`openUrl`方法打开这个URL就可以触发支付宝/微信的scheme。具体代码大致如下：
```swift
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    pushCurrentSnapshotViewWithRequest(request: navigationAction.request)

    guard let curUrl = navigationAction.request.url else {
        decisionHandler(.allow); return
    }

    if curUrl.absoluteString.hasPrefix("alipay://alipayclient/") || curUrl.absoluteString.hasPrefix("weixin://"){
        decisionHandler(WKNavigationActionPolicy.cancel)
        UIApplication.shared.openURL(url)
        return
    }
    decisionHandler(WKNavigationActionPolicy.allow)
}
```

开心，居然这么简单。
然后…emmmmm，跳是可以正常跳了，但是好像支付结束后无法跳回APP。
冷静分析一下，我们都知道iOS内的应用间跳转，基本都是通过scheme的方式，跳出去如此，要返回也是如此。

### 花明

先看下支付宝支付：
捕获支付宝web支付跳转链接如 `alipay://alipayclient/?{"requestType":"SafePay","fromAppUrlScheme":"alipays","dataString":"h5_route_token=\"shierRZ25\"&is_h5_route=\"true\""}`
发现其中只要将fromAppUrlScheme改为APP内配置的scheme，即可正确跳转回应用。具体代码示例如下：
```swift
fileprivate func handleAlipayUrl(url: URL) -> URL? {
    if url.absoluteString.hasPrefix("alipay://alipayclient/") {
        // 更换scheme
        var decodePar = url.query ?? ""
        decodePar.urlDecode()
        var dict = JSON(parseJSON: decodePar)
        dict["fromAppUrlScheme"] = "xproject"

        if let strData = try? JSONSerialization.data(withJSONObject: dict.dictionaryObject ?? [:], options: []) {
            var param = String(data: strData, encoding: .utf8)
            param?.urlEncode()

            let finalStr = "alipay://alipayclient/?\(param ?? "")"
            if let finalUrl = URL(string: finalStr) {
                return finalUrl
            }
        }
        return url
    }
    return nil
}
```
似乎挺顺利，再看一下微信，微信的h5支付回调应该是服务端提供的一个h5地址，因此支付完成后默认是跳转到了Safari，在APP内进行的支付，我们要换掉这个回调，变成我们自己的。

大致步骤是：
- 工程文件添加Scheme，内容为[APP本地配置的scheme]
- 捕获跳转链接 https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb，将其中其中的redirect_url参数换成[APP本地配置的scheme]
- 重新发起请求，给请求头加上Referer字段，内容为[APP本地配置的scheme]
- 使用openUrl发起微信客户端调用
这里参考了这篇文章 https://www.jianshu.com/p/c1973aacc774
需要注意的一点就是，[APP本地配置的scheme]需要是http的URL形式，而且根域名是要包含在微信支付后台填写的白名单内的，譬如白名单域名是abc.com，你可以将你的scheme设置为ios.abc.com，否则也不会生效。

具体代码大致如下：
```swift
let wxpayScheme = "ios.abc.com://"
// 去除原有的URL回调地址，换成自己的配置
if curUrl.absoluteString.hasPrefix("https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb") {
    if var comps = URLComponents(string: curUrl.absoluteString) {
        var needChange = false
        for (idx, item) in (comps.queryItems ?? []).enumerated() {
            if item.name == "redirect_url" && item.value != wxpayScheme {
                needChange = true
                comps.queryItems?.remove(at: idx)
                break
            }
        }
        if needChange {
            comps.queryItems?.append(URLQueryItem(name: "redirect_url", value: wxpayScheme))
            if let finalUrl = comps.url {
                // 给请求头加上Referer字段
                let mRequest = NSMutableURLRequest(url: finalUrl)
                mRequest.setValue(wxpayScheme, forHTTPHeaderField: "Referer")

                decisionHandler(WKNavigationActionPolicy.cancel)
                webView.load(mRequest as URLRequest)
                return
            }
        }
    }
}
```
替换的过程有一点绕，其实就是找到相应字段替换掉，有更好地写法。
尝试了一下，可以成功跳转回来了，但是新的问题又出现了→_→

### 又一村
因为替换了微信支付的回调，h5的跳转可能会出现白屏的问题，上面的文章也有提到。
我根据自己的实际情况采用了直接强制调用`webView.goBack()`，因为本身的H5页面自带了支付等待完成页，支付完成后返回APP，确认一下支付状态就好了。
需要注意的是，调用微信支付5秒后，webview会收到一个链接调整，截获然后进行后退就好：
```swift
if curUrl.absoluteString.hasPrefix(wxpayScheme) {
    // 进入空白页，强制后退
    decisionHandler(WKNavigationActionPolicy.cancel)
    webView.goBack()
    return
}
```

不过这个白屏问题可能会根据本身h5的不同而采取不同的解决方案，所以这个应该并非万全之策。

参考文章：
[iOS 解决微信h5支付无法直接返回APP的问题](https://www.jianshu.com/p/90db7dfb075c)
[iOS微信H5支付无法返回APP解决方案](https://www.jianshu.com/p/c1973aacc774)