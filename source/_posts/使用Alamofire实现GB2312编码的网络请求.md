---
title: Swift 使用 Alamofire 实现 GB2312 编码的网络请求
date: 2016-08-02 14:04:58
tags:
- iOS
- Swift
- Alamofire
- GB2312编码
categories:
- 开发
- iOS
---

# 背景

最近在做一个校园工具，其中有个功能是查看自己校园网账户的剩余流量。学校的网络管理中心没有开放接口，于是只能利用Alamofire来模拟登录解析数据。一开始通过GET请求获取用户账户余额都是没有问题的，但是通过POST请求的话，就无法正确访问数据。

# 原因

之前知道该网站是GB2312编码的，于是在解析data为HTML的时候就用GB2312解码的。但在POST数据时，也需要GB2312编码。我开始时将post的string参数用GB2312编码，发现不行。于是就查看了Alamofire的源码，发现其默认的参数编码是UTF-8，而且只有这一个选择😂。

``` swift
if mutableURLRequest.valueForHTTPHeaderField("Content-Type") == nil {
    mutableURLRequest.setValue("application/x-www-form-urlencoded; charset=utf-8",forHTTPHeaderField: "Content-Type")
}
```

这样就很尴尬了。在Google搜了半天，也没找到类似的问题。于是问了我一个经常抓包的哥们，他说GB2312编码的服务器遇到的次数非常少。再加上本来用swift模拟请求的场景就不多了，难怪没同样的问题呢。试了很久，我都准备鸣金收兵了。

# 解决

后来问了学长这个问题，他说可以自定义Alamofire的request，在request中进行参数的GB2312编码或许可以。于是带着最后一丝希望去试了一下，发现学长真行🤗。

自定义Alamofire的request代码如下：
```swift
let GB2312Encoding = CFStringConvertEncodingToNSStringEncoding(0x0632)
private func urlRequestWithComponents(urlString:String,parameters:[String: AnyObject]) ->(URLRequestConvertible,NSData) {
        
    // create  url request to send
    let mutableURLRequest = NSMutableURLRequest(URL: NSURL(string: urlString)!)
    mutableURLRequest.HTTPMethod = Alamofire.Method.POST.rawValue
    let contentType = "application/x-www-form-urlencoded"
    mutableURLRequest.setValue(contentType, forHTTPHeaderField: "Content-Type")
    mutableURLRequest.setValue("*******", forHTTPHeaderField: "Referer")
        
    // add parameters
    let uploadData = NSMutableData()
    for (key,value) in parameters {
        uploadData.appendData("\(key)=\(value)&".dataUsingEncoding(GB2312Encoding)!)
    }
        
    return (Alamofire.ParameterEncoding.URL.encode(mutableURLRequest, parameters: nil).0,uploadData)
}
```

调用该自定义Request的方法如下：

``` swift
let urlRequest = urlRequestWithComponents(SchoolNetWorkBaseURL, parameters: parameters)
Alamofire.upload(urlRequest.0, data: urlRequest.1).response {
    request, response, data, error in
}
```

通过这个，就可以实现使用Alamofire实现GB2312编码的请求了。

# 番外篇：关于使用Alamofire实现gzip压缩参数的网络请求

在解决上面的问题时，我一度偏失了方向，以为是参数没有压缩的问题。下面记录一下gzip压缩的参数问题。

下面是对ParameterEncoding的扩展，Alamofire在请求的request的encoding中，选择ParameterEncoding.gzipped即可。

``` swift
//
//  ParameterEncoding-Extension.swift
//  AHNUer
//
//  Created by 黄帅 on 16/7/28.
//  Copyright © 2016年 帅帅. All rights reserved.
//

import Foundation
import Alamofire

// Actual gzipping from https://github.com/1024jp/NSData-GZIP

// Example: ParameterEncoding.JSON.gzipped

infix operator • { associativity left }
func • <A, B, C>(f: B -> C, g: A -> B) -> A -> C {
    return { x in f(g(x)) }
}

extension ParameterEncoding {
    
    var gzipped:ParameterEncoding {
        
        return gzip(self)
    }
    
    private func gzip(encoding:ParameterEncoding) -> ParameterEncoding {
        
        let gzipEncoding = self.gzipOrError • encoding.encode
        
        return ParameterEncoding.Custom(gzipEncoding)
    }
    
    private func gzipOrError(request:NSURLRequest, error:NSError?) -> (NSMutableURLRequest, NSError?) {
        
        let mutableRequest = request.mutableCopy() as! NSMutableURLRequest
        
        if error != nil {
            return (mutableRequest, error)
        }
        
        var gzipEncodingError: NSError? = nil
        
        do {
            let gzippedData = try mutableRequest.HTTPBody?.gzippedData()
            mutableRequest.HTTPBody = gzippedData
            
            if mutableRequest.HTTPBody != nil {
                mutableRequest.setValue("gzip", forHTTPHeaderField: "Content-Encoding")
            }
        } catch {
            gzipEncodingError = error as NSError
        }
        
        return (mutableRequest, gzipEncodingError)
    }
}

```

# 参考

* [Swift - 使用gzip压缩NSData数据](http://www.hangge.com/blog/cache/detail_1032.html)
* [blender/ParameterEncodingExt](https://gist.github.com/blender/923f1c1de2f00514ed12)


