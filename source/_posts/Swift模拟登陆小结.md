---
title: Swift模拟登陆小结
date: 2016-08-02 18:27:36
tags: 
- Swift 
- Alamofire 
- 模拟登陆
categories:
- 开发
- iOS
---

元旦到现在都没有写博客了，就拿这篇当做重新开始的起点吧。

# 背景

过完年在学校度过大学的最后一学期，期间加深学习了Python的Django和iOS开发。在学习的过程中，就想着做一个校园iOS应用。由于我们的需要的信息分散在学校各个部门的系统中，此时就需要通过模拟登陆来获取我们想要的信息。一开始是用python来模拟登陆解析数据的，后期由于我对架构设计的改变，我把很多解析放在手机端来处理，所以就应运而出本篇博文。

# 准备工作

要模拟浏览器登陆，首先得分析浏览器登陆的步骤，再用代码来实现。

## 工具

### Alamofire

1. Alamofire 的前身是 AFNetworking。AFNetworking 是 iOS 和 OS X 上很受欢迎的第三方HTTP网络基础库。
2. 其实 AFNetwork 的前缀 AF 便是 Alamofire 的缩写。
3. Swift发布后，AFNetworking的作者又用Swift语言写了个相同功能的库，这便是 Alamofire。
4. Alamofire 本质是基于`NSURLSession`，并做了封装。使用 Alamofire 可以让我们网络请求相关代码（如获取数据，提交数据，上传文件，下载文件等）更加简洁易用。

**关于Cookie:**
Alamofire是基于NSURLRequest封装的，所以Cookie会自动保存，就和浏览器请求是一个效果。而且网站Set_cookie多久，本地的Cookie就多久，每次请求的时候都会自动带上cookie，直到过期。（所以像登陆session这些的都不用我们手动去处理）。

### Charles

这是一款常用的网络封包截取工具，在做开发时，我们为了调试与服务器端的网络通讯协议，常常需要截取网络封包来分析。Mac和PC都可以使用。今天好像发布了新版了。这个还可以作为iPhone的http代理，来抓包手机的网络包。关于此软件的使用详情，可以阅读[唐巧老师的博客]("http://blog.devtang.com/2015/11/14/charles-introduction/")。

### Postman

这是Mac上Chrome的一个网络调试的工具插件，PC上不知道有没有，可以通过该工具来进行各种http请求。

### 编码工具

这是一个在线的[站长工具](http://tool.chinaz.com/tools/urlencode.aspx)，可以进行各类编码间的转换，最常用的应该就是utf-8的UrlDecode和UrlEncode。这个工具还非常贴心的提供了gb2312格式的UrlDecode和UrlEncode。

## 基础知识

POST请求和GET请求应该就不用解释了，这里提醒一下编码格式，现在大部分服务器都是utf-8编码格式的，但不排除少量用的GB2312。所以在发现服务器响应数据乱码时要检查返回数据的编码格式。

## 几个例子

### 登陆1

做校园APP想到的第一个功能就是查成绩查课表，于是第一个就是拿教务系统动刀。

教务系统登陆的时候，其实是跳转到一个check页验证账号密码，再跳转回教务系统首页。该网页返回的就是JSON数据，所以用的`responseJSON`。

``` swift
Alamofire.request(Method.POST, SchoolBaseURL+"login/check.shtml"
, parameters:["user":"*****","pass":"*****","usertype":"stu"]
, encoding: ParameterEncoding.URL
, headers: nil).responseJSON {
                response in
            
            guard response.result.isSuccess  else {
                self.callback?.managerApiCallBackFailed(self)
                Hud.showError("网络错误了")
                return
            }
            // 登陆成功
}
```

### 登陆2

想到的第二个功能就是查询校园网流量使用情况，于是瞄上了信息管理中心的校园网系统。

这里请求时必须带上Content-Type和Referer。否则就会跳到登录页。这个系统就是我讲的GB2312编码的坑货系统。返回的data需要经过GB2312编码。

``` swift
Alamofire.request(Method.POST, SchoolNetWorkBaseURL
, parameters:["username":"*****","password":"*****"]
, encoding: ParameterEncoding.URL
, headers: ["Content-Type":"application/x-www-form-urlencoded","Referer":SchoolNetWorkBaseURL]).response {
                request, response, data, error in 
                
                // 业务处理
                let GB2312Encoding = CFStringConvertEncodingToNSStringEncoding(0x0632)
                let selfHTMLString:String = NSString(data: data!, encoding: GB2312Encoding)! as String
}

```
 
弄到这里感觉还比较轻松，但是最坑爹的是，这个网站的请求参数必须是经过GB2312的URLEncode。而Alamofire只能将参数进行utf-8编码。我开始时以为将请求参数用GB2312编码后传入即可，或者利用GB2312编码的URLEncode后再传入。但是都相当于Alamofire在最外层还是用UTF-8又URLEncode了一遍。

这是Alamofire的源码，这里可以由`charst=utf-8`看出它默认将数据进行utf-8编码。
``` swift
if let method = Method(rawValue: mutableURLRequest.HTTPMethod) where encodesParametersInURL(method) {
    if let
        URLComponents = NSURLComponents(URL: mutableURLRequest.URL!, resolvingAgainstBaseURL: false)
        where !parameters.isEmpty
    {
        let percentEncodedQuery = (URLComponents.percentEncodedQuery.map { $0 + "&" } ?? "") + query(parameters)
        URLComponents.percentEncodedQuery = percentEncodedQuery
        mutableURLRequest.URL = URLComponents.URL
    }
} else {
    if mutableURLRequest.valueForHTTPHeaderField("Content-Type") == nil {
        mutableURLRequest.setValue(
            "application/x-www-form-urlencoded; charset=utf-8",
            forHTTPHeaderField: "Content-Type"
        )
    }

    mutableURLRequest.HTTPBody = query(parameters).dataUsingEncoding(
        NSUTF8StringEncoding,
        allowLossyConversion: false
    )
}
```

这里参考学长给的建议，将request先自己编码后再进行传输。首先封装了自定义编码request方法。

``` swift
private func urlRequestWithComponents(urlString:String,parameters:[String: AnyObject]) -> (URLRequestConvertible,NSData) {
    
    // create  url request to send
    let mutableURLRequest = NSMutableURLRequest(URL: NSURL(string: urlString)!)
    mutableURLRequest.HTTPMethod = Alamofire.Method.POST.rawValue
    let contentType = "application/x-www-form-urlencoded"
    mutableURLRequest.setValue(contentType, forHTTPHeaderField: "Content-Type")
    mutableURLRequest.setValue("http://nic.ahnu.edu.cn/cgi-bin/service", forHTTPHeaderField: "Referer")
    
    // add parameters
    let uploadData = NSMutableData()
    for (key,value) in parameters {
        uploadData.appendData("\(key)=\(value)&".dataUsingEncoding(GB2312Encoding)!)
    }
    
    return (Alamofire.ParameterEncoding.URL.encode(mutableURLRequest, parameters: nil).0,uploadData)
}
```


``` swift

let urlRequest = urlRequestWithComponents(SchoolNetWorkBaseURL
, parameters:["username":"****","credential":self.credential,"logtbl":"int201607","echo":"查询","func":"计费网关"])
 
Alamofire.upload(urlRequest.0, data: urlRequest.1).response {
    request, response, data, error in
    
    let htmlString:String = NSString(data: data!, encoding: GB2312Encoding)! as String
    print(htmlString)
}
```

// 未完待续



