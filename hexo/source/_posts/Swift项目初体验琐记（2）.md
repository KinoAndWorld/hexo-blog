---
title: Swift项目初体验琐记（2）
date: 2017-04-09 00:04:35
tags: Swift
categories: iOS
---

### Swift上的常用第三方框架

- Alamofire 
	相当于OC上的AFNetWorking，毕竟作者也是同一个人😂。不过在Swift上，一般还会搭配其他框架封装在一起。这个等会一起介绍。

- RxSwift
	相对于ReactiveCocoa的swift版本来说，RxSwift似乎更“血统纯正”一些，就我个人的使用而言，与ReactiveCocoa的基本用法还是非常相似的。RxSwift的教程也很多，就不展开了。
	
- SnapKit
对应Objective-C上的masorny，同一家写的，语法几乎一样~
	
- Kingfisher 
喵神写的，对应于SDWebImage 或者 YYWebImage

- HandyJSON：
阿里巴巴团队开源的Json自动转model框架，现有的Json转model的第三方库基本都是用到了上篇文章提到的`Mirror`（反射）把model所有字段映射出来然后匹配，但是似乎都不能支持像Objective-C的一些库可以不用写映射自动用变量名匹配字段名。而HandyJson用了一个很高端的`内存位移映射`的办法来实现变量名匹配和赋值，可以让你不用重载任何mapping方法就能自动转成Model。（对原理感兴趣的可以戳[这里](http://www.jianshu.com/p/eac4a92b44ef)）

### Moya + RxSwift + HandyJSON

在App中我们最常打交道的或许就是网络模块了，虽然Alomfire写得非常棒，但是我们在项目中一般不会直接使用，而是再通过一个网络层封装，得以对接我们的具体业务和配置，懒惰如我，四处搜寻一个好的网络层解决方案，然后发现[Moya](https://github.com/Moya/Moya)这个第三方网络中间件，我将在本文着重介绍。
	
首先，他的基础用法和扩展[这篇文章](https://blog.callmewhy.com/2015/11/01/moya-rxswift-argo-lets-go/)已经写得很清楚了，不过我们这里没有用argo，而是用了上面提到的HandyJSON

OK，还是用实例说话，首先我们新增一个可重用的extension
	
```swift
extension ObservableType where E == Moya.Response {
	
	func preOrderCheck(response: Response) throws -> JSON {
		// check http status
		guard ((200...209) ~= response.statusCode) else {
			throw ServiceError.NotSuccessfulHTTP
		}
		// unwrap biz json shell
		let json = JSON.init(data: response.data)
		
		// check biz status
		let code = json[RESULT_CODE].stringValue
		if code == BizStatus.BizSuccess.rawValue {
			return json[RESULT_DATA]
		}else {
			throw ServiceError.OtherError(resultCode: json[RESULT_CODE].stringValue, resultMsg: json[RESULT_MSG].stringValue)
		}
	}
	
	func mapResponseToModel<T: HandyJSON>(type: T.Type) -> Observable<T> {
		return map{ response in
				do {
					let respContent = try self.preOrderCheck(response: response)
					if let obj = T.deserialize(from: respContent.rawString()) {
						return obj
					}else {
						throw ServiceError.CouldNotMakeObjectError
					}
				} catch let err as ServiceError {
					throw err
				}
		}
	}
	
	func mapResponseToModelList<T: HandyJSON>(type: T.Type) -> Observable<[T]> {
		return map{ response in
			do {
				let respContent = try self.preOrderCheck(response: response)
				if let objs = [T].deserialize(from: respContent.rawString()) as? [T] {
					return objs
				}else {
					throw ServiceError.CouldNotMakeObjectError
				}
			} catch let err as ServiceError {
				throw err
			}
		}
	}
}
```
	
然后我们就可以很轻松地写出这样的请求：
	
```swift
MovieProvider.request(MovieApi.cityList)
	.mapResponseToModelList(type: MovieCity.self)
	.subscribe(onNext: { movieCitys in
		
	}, onError: { error in
		print(error)
	}).addDisposableTo(disposebag)
```
非常简洁且高度可扩展，而且因为用了泛型，甚至可以不用做过多转换，得到的model list是直接可用的。
	
另外，值得一提的是，得益于Moya的灵活和扩展性，我们可以给Moya的provider做很多定制化，比如我这里的MovieProvider其实是这样的：
	
```swift
let MovieProvider = RxMoyaProvider<MovieApi>(
	endpointClosure: normalEndpointClosure(),
	plugins: [NetworkLoggerPlugin(verbose: true, cURL: true,responseDataFormatter: JSONResponseDataFormatter)]
)
```
	
`plugins`中Moya提供了打印网络请求参数的`NetworkLoggerPlugin`和网络状态控制的`NetworkActivityPlugin`，当然你也可以自己写一个Plugin
	
`endpointClosure`这里可以去定制化一些http参数，比如header和common param，我写了一个比较共用的方法大概这样：
	
```swift
let headerFields: Dictionary<String, String> = [
	"platform": "iOS",
	"Auth-Token": "get some token from your code"
]

let headerFieldsWithoutToken: Dictionary<String, String> = [
	"platform": "iOS"
]
	
func normalEndpointClosure<T: TargetType>() -> ( (_ target: T) -> Endpoint<T>) {
	return { (target: T) -> Endpoint<T> in
		let appendedParams: Dictionary<String, AnyObject> = ["key" : "somekey" as AnyObject]
		let fields:Dictionary<String, String> = headerFields
		
		return Endpoint<T>(url: url(target), sampleResponseClosure: {.networkResponse(200, target.sampleData)}, method: target.method, parameters: target.parameters)
			.adding(newParameters: appendedParams)
			.adding(newHTTPHeaderFields: fields)
	}
}
```
	
比如登录之后，可能需要在请求头加token或者每个请求参数带个token，就可以在这里统一添加。
	
嗯~大概就这么多。我会把总结的这部分内容写成一个[小demo](https://github.com/KinoAndWorld/KONetworkLayerDemo)，算是一个小小的学习成果吧。