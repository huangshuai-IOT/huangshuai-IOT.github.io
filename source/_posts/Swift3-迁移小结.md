---
title: Swift3 迁移小结
date: 2016-10-8 22:56:05
tags: 
- Swift 3.x
- Alamofire 4.x
- Kanna 2.x
- Eureka 2.x
categories:
- 开发
- iOS
---
## 背景

9.15号中秋节那天发布了[师大助手App](https://itunes.apple.com/us/app/shi-da-zhu-shou-hui-shi-fan/id1150983683?l=zh&ls=1&mt=8)，后来有次在 Yosemite OS 下更新到 Xcode8 ，打开后项目一片红，当时只是想修复一下 bug，还没想要迁移到 Swift3，于是默默的 重新装了 Xcode7，又继续愉快的改bug了，然后9月20号提交到商店更新也没问题。21号提示 Sierra 可以更新了，于是手贱更新了一波。然后悲剧就开始了。
更新完 Sierra 后，我添加了Widgets功能，于是准备上传到商店，然后就是 一直传不上去。试了一下几个方法：

1. Xcode7 打包，Xcode8 上传
2. Xcode7 打包，在 Sierra 下用 Application Loader 上传
3. Xcode7 打包，在同学 Yosemite 下用 Application Loader 上传

结果以上都不行，一直卡在验证那里，于是我就一狠心，说用 Xcode8 ，做一下 Swift3 迁移😂，开启了升级打怪之旅

## 步骤

### 自动代码 Convert

在这里，我只 Convert 自己写的代码，事实证明是明智的，因为第三方库自己都做了适配，不需要我们来进行 Convert，这一步做完，自己的代码大致上没什么问题了，主要错误都集中在第三方库的接口调用上

### 第三方库 Update

我有个习惯，就是 update 之前喜欢先去项目主页看一下，看看有没有什么坑。

#### CocoaPods 

1. 卸载已有的 CocoaPods (非必须)
``` swift 
$ which pod
/usr/local/bin/pod
$ sudo rm -rf /usr/local/bin/pod
```
2. 安装支持 Xcode8 的 CocoaPods
``` swift
sudo gem install cocoapods --pre
```
更新完 CocoaPods 还有更新本地库，就不说了。

#### Alamofire

去官网看了一下，真是坑爹，Alamofire 4.0 变化好大，接口都按照 Swift3 重新弄了，二话不说，先更新一下。
``` swift
pod 'Alamofire', '~> 4.0'
```
基本的代码更新看[官方文档](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Alamofire%204.0%20Migration%20Guide.md)就差不多了。下面写几个官方文档没有提到的：

##### 配置https证书
``` swift
func configureAlamofireManager() {
   let manager = SessionManager.default
   manager.delegate.sessionDidReceiveChallenge = { session, challenge in
       var disposition: URLSession.AuthChallengeDisposition = .performDefaultHandling
       var credential: URLCredential?
       
       if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust {
           disposition = URLSession.AuthChallengeDisposition.useCredential
           credential = URLCredential(trust: challenge.protectionSpace.serverTrust!)
       } else {
           if challenge.previousFailureCount > 0 {
               disposition = .cancelAuthenticationChallenge
           } else {
               credential = manager.session.configuration.urlCredentialStorage?.defaultCredential(for: challenge.protectionSpace)
               
               if credential != nil {
                   disposition = .useCredential
               }
           }
       }
       return (disposition, credential)
   }
}
```

##### 自定义Request请求

``` Swift
let urlRequest = urlRequestWithComponents(SchoolNetWorkBaseURL, parameters: parameters)
Alamofire.upload(urlRequest.1, with: urlRequest.0).response {
    response in
        
    let htmlString:String = NSString(data: response.data!, encoding: GB2312Encoding)! as String
    print(htmlString)
}

fileprivate func urlRequestWithComponents(_ urlString:String,parameters:[String: AnyObject]) -> (URLRequestConvertible,Data) {
        
   // create  url request to send
   var mutableURLRequest = URLRequest(url: URL(string: urlString)!)
   mutableURLRequest.httpMethod = HTTPMethod.post.rawValue
   let contentType = "application/x-www-form-urlencoded"
   mutableURLRequest.setValue(contentType, forHTTPHeaderField: "Content-Type")
   mutableURLRequest.setValue("****", forHTTPHeaderField: "Referer")
   
   // add parameters
   var uploadData = Data()
   for (key,value) in parameters {
       uploadData.append("\(key)=\(value)&".data(using: String.Encoding(rawValue: GB2312Encoding))!)
   }
   
   return (try! URLEncoding.default.encode(mutableURLRequest, with: nil),uploadData)
}
```

#### Eureka

Eureka 在新版本中删除了 ImageRow，我之前代码有一处无法调用，后来提了个[issue](https://github.com/xmartlabs/Eureka/issues/688)，问了作者。
![Issue](http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Swift3_1.png)
由于在 iOS10 中，app 需要对相册和相机的使用进行权限申明，不然会闪退，未避免其他 app 闪退，就删了 imageRow，作者也给了解决方法，按照他给方法，本地添加 imageRow 就可继续使用了，或者自己实现CustomRow，准备过段时间试一下自定义 CustomRow。

#### Kanna

项目中需要解析 html ，一直用的 Kanna，在 swift3 中，原来的`doc.xpath("//table/tr").count`变成了 `doc.xpath("//table/tr").nodeSet.count`。


