---
title: iOS:webApp客户端开发小记
date: 2015-11-05 17:00:15
tags:
- iOS
- Swift
- UIWebView
- 二维码扫描
categories:
- 开发
- iOS
---

>这篇文章，现在是我写的第二遍，因为就在我刚才准备commit的时候，手贱点了sync，结果白写了一个小时。

## 前言
暑期以来我一直在一家物联网公司实习，刚来的时候开发了一款物联网云服务的iOS客户端，也许你会问为啥一个实习生刚来就开发产品，主要原因是我能力超强，还有一个很小的原因是全公司就我用Mac，并且会点点Swift，哈哈。

刚来是开发一款物联网应用的iOS客户端（[1.0版本](https://itunes.apple.com/us/app/iotcloud/id1045360550?l=zh&ls=1&mt=8)现在已上线），后期开发了一段时间这个产品的后台，主要是MongoDB和Java Web。之后，我又被叫去开发另一款云教育应用的iOS客户端和Android客户端。这篇文章主要介绍的是开发这两款iOS应用的总结，后期会总结一下在Android上实现同样功能的经验。

我们公司的产品都是前端是采用HTML5的，这样每个平台的客户端只要套个UIWebView的壳子就好。但是问题不是全部网页都是从服务器端加载的，因为不能保证网络一直畅通，以及服务器没有错，要是网络不通，那整个应用就是一片空白，这样的体验太差了。

所以我们的想法是欢迎页、登陆页都是本地的资源，这样应用打开，不管服务器啥情况，用户都是可以看见东西的。

***

## 应用逻辑

1. 打开应用，首先加载welcome.htm(原本是jsp文件，在Android平台上可以加载，但是Safari不行，所以直接改了后缀)，welcome.htm的js会从sqlite中取出用户名和密码，若不为空，则直接表单提交登录；如为空，则跳转到本地login.htm界面进行登录

2. 用户在login.htm进行登录，若登录成功，则其中的js文件会将用户名、密码存入sqlite中

3. 应用中有二维码扫描功能，若点击该按钮，则跳转回本地，使用相机进行二维码扫描，然后将结果又返回给网页应用

4. 应用中有退出功能，若点击该按钮，则跳转回本地login.htm界面，其中的js会清空sqlite中数据

***

## 遇到的问题以及解决方法

1. Xcode7导入本地资源
之前用的是Xcode6，项目下面有一个Supporting Files文件夹，将本地资源导入后都在此文件夹中，升级到Xcode7后，没有这个文件夹了，可以直接将本地资源直接导入项目中。

2. iOS9网络连接问题
刚开始用Xcode6开发第一项目的时候，UIWebView联网是没有问题的，第二个项目使用的是Xcode7和iOS9，就连打开baidu都无法实现，控制台会报错:
当时在Stack Overflow找到原因了:

	```swift
	Application Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app's Info.plist file.
	```

	**在iOS9 beta1中，苹果将原http协议改成了https协议，使用 TLS1.2 SSL加密请求数据。**
解决方法就是：在info.plist中添加

	```swift
	<key>NSAppTransportSecurity</key>
	<dict><key>NSAllowsArbitraryLoads</key><true/></dict>
	```
	后来在[《微信在适配iOS9上遇到的问题和解决方案》](http://www.infoq.com/cn/articles/wechat-ios9-adaptation)也看到了该问题的解决方法。

3. UIWebView加载本地资源

	我做的项目中，主要是UIWebView加载本地的网页，代码如下：
	
	```swift
	let path = NSBundle.mainBundle().pathForResource("index", ofType: "htm")
	let urlobj = NSURL.fileURLWithPath(path!)
	let request = NSURLRequest(URL: urlobj)
	myWebView.loadRequest(request)
	```

4. 从网页跳转回本地

	前面说到项目的两个功能：一是网页调用本地二维码；二是在网页里退出，本地能清除登录数据。

	我的解决思路是：web应用中，需要跳转本地的功能，都把链接写成固定格式的

	```html
	<a href="m://erweima.com/" class="link_btn link_btn_t">二维码扫描</a>
	```
	UIWebView中有一个方法，可以在链接地址改变时触发，我就是利用该方法，判断UIWebView即将要跳转的链接地址，如果是事先确定的，则执行本地相关操作，如上所述的二维码操作：

    ```swift
func webView(webView: UIWebView,shouldStartLoadWithRequest request: NSURLRequest,navigationType: UIWebViewNavigationType) -> Bool {
    let rurl = request.URL
    if(rurl!.scheme == "m" && rurl!.host == "erweima.com") { 
    //启动二维码扫描
    tiaozhuan()
    }
    //返回true就会跳转新地址
}
```

5. 二维码扫描功能

	这里用到一个用Swift2和AVFoundation写的二维码扫描的类[github](https://github.com/Recursion0210/QRCode)，类里用的是UIStoryBoard进行跳转的，我稍微改动了一下，用方法调用的：

    ```swift
func tiaozhuan() {
    let scannerVC:ScannerViewController = ScannerViewController()
    scannerVC.delegate = self
    self.presentViewController(scannerVC, animated: false, completion: nil)
}
```

    ViewController继承了`ScannerViewControllerDelegate`协议，实现了：
    
    ```swift
func barcodeObtained(viewController:ScannerViewController, data: String) {
    //打印二维码扫描的结果
    print(data)
}
```

***
项目本来已经做完了，经理又加了一些功能，因此又来补充了。

今天周三，是光棍节，周一做的是Android和iOS的判断网络状态的功能，周二和今天上午做的是Android的即时通讯和实时音视频功能，今天下午做的是iOS的即时通讯功能，由于SDK用的是OC，而我用的是Swift，所以到现在也没搞出来啥，还好项目不急，所以来更新一下博客。

2015-11-11 17:55

***

6. 判断网络状态

   ##### 问题：

   是这样的，由于产品登陆完之后，全部内容都在UIWebView中，要是网络突然断了，就会显示无法连接界面，这样用户体验不好，如果后面能联网了，又不能返回到断网之前的界面。

   ##### 需求：

   在尽量不改后台的情况下，解决上面的问题

   ##### 解决思路：

   在每次需要跳转之前，先判断一下网络状态，若是可以联网则不处理；若不能联网，则存储当前链接，并跳转到本地一个不能联网的界面，该界面上有刷新按钮，通过不断点击刷新按钮来重复跳转该界面，然后可以判断网络情况，若可以联网了，则加载之前存储的链接。

   上面的逻辑代码就不贴了，没有参考意义，反正就是在上面第四点的webView方法中进行处理。这里我放一个判断网络状态的项目地址[Reachability.swift-github](https://github.com/ashleymills/Reachability.swift)

7. 即时通讯

   ##### 问题：

   由于我们的项目是做教育类的，需要用户之间可以进行类似于微信那样的通信功能，并且可以进行实时音、视频通话功能。

   ##### 需求：

   实现上述功能，还要尽可能的保证服务质量

   ##### 解决思路：

   我想到的方法是集成第三方的服务，目前用的是[容联云服务](http://www.yuntongxun.com/)。还没做完，未完待续。  


