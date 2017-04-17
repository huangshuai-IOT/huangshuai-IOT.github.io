---
title: 阿里百川用户反馈Swift开发小结
date: 2016-12-29 19:41:25
tags:
- Swift  
- 阿里百川
- 用户反馈
categories:
- 开发
- iOS
---

## 背景

所有的App都需要给用户一个反馈意见的功能，这样才能让用户使用的更好更方便。不想自己写这一套组件，因此调研了几款第三方的反馈组件。

### Instabug 
九月份上架的师大助手App用的就是这个反馈组件，知乎日报用的也是这个，不管在App任何界面摇一摇就可以触发，Instabug会把当前界面进行截图并且支持用户编辑，非常方便好用的组件，这个用来反馈bug我觉得非常好用，但是作为反馈意见的话，感觉还是缺少一个入口。另外，Instabug采集Bug还是非常准确的。

### LeanCloud
LeanCloud的反馈组件号称两行代码接入，非常方便，它做成一个对话框的形式，客服与用户可以进行交流，并且用户的反馈可以及时发给开发人员的LeanCloud 手机客户端，实时性非常高。

### 阿里百川
一年之前开发的app用了阿里百川（阿里巴巴无限开放平台）的即时通讯功能，当时阿里巴巴刚把友盟这些公司收购，阿里巴巴无线开放平台还没有现在这么多功能，一年过去了，现在的功能已经非常多了。今天看用户反馈功能时，看到了他们的2.0版本的反馈组件，觉得这种加入了常见反馈问题这一页特别好，用户在反馈之前，可以先自行查看相似问题的解答。并且，相似问题的解答是可以实时在后台添加修改，这极大方便了用户。

于是就暂定了使用阿里百川，于是进行了接入的技术验证。先看一下效果图：
<div id="tt" style="display: inline-block;">
       <img src="http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Baichuan_1.PNG" width="50%" height="25%" />
       <img src="http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Baichuan_2.PNG" width="50%" height="25%" />
</div>

## 步骤
### Part1 阿里百川平台注册
这里按照官方的接入流程就可以很容易完成了，就不赘述了。

### Part2 用户反馈SDK导入
我选择使用的是专业版2.0，SDK用的是Cocoapods安装方式，在Podfile添加阿里的私有源。

```swift
source 'http://repo.baichuan-ios.taobao.com/baichuanSDK/AliBCSpecs.git'
source 'http://repo.baichuan-ios.taobao.com/baichuanSDK/AliBCSpecsMirror.git'
target 'YourTargetName' do
    pod 'YWFeedbackFMWK', '~> 2.0.3.1'
end
```

接着使用 `pod install` 安装出现了一个问题，就是我之前用的 Alamofire 等其他很多组件安装不了，出现需要更新的问题，我这里猜测 AliBCSpecsMirror 和 CocoaPods/Specs 有一些差异，弄了一会索性直接在上面再添加 CocoaPods 的仓库，`source 'https://github.com/CocoaPods/Specs.git'`, 这样就能同时安装YWFeedbackFMWK和我之前的其他组件了，但是又会出现repo仓库有两份的warning，这个还没解决好，我考完试再找找解决方法。

注意： Cocoapods集成方式不需要手动添加依赖系统库

修改编译选项：
在Target->Linking->Other Linker Flags中添加-ObjC选项。

iOS 10中隐私权限设置：
如果不做设置，可能会导致崩溃、审核不通过等情况。需要在info plist中增加字段：
``` xml
<key>NSCameraUsageDescription</key>
<string>访问相机</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>访问相册</string>
```
### Part3 初始化SDK，打开反馈页面
1. 在桥接头文件中加入：

``` swift
#import <YWFeedbackFMWK/YWFeedbackKit.h>
#import <YWFeedbackFMWK/YWFeedbackViewController.h>
```

2. 调用初始化方法：

``` swift
var bcKit = BCFeedbackKit(appKey: "自己的appkey")
```
**注意：**这里需要将在阿里百川后台生成的安全图片添加进项目目录中

3. 设置业务方扩展的反馈数据：
相当于给反馈信息中加入反馈用户的个人信息，可设置任意字段，在创建反馈页面前设置，可在后台扩展信息中查看。

``` swift
bcKit.extInfo = ["username":"自己app用户体系中的用户id"]
```

4. 打开反馈页面：
5. 
``` swift
bcKit.makeFeedbackViewController(completionBlock: { (feedbackVC, error) in
    if let vc = feedbackVC {
    // 当用户关闭用户反馈，将调用该block进行dismiss或pop
        vc.closeBlock = {(aParentController) in
            aParentController?.navigationController?.popViewController(animated: false)
        }
        
        self.navigationController?.isNavigationBarHidden = false        
        self.navigationController?.pushViewController(vc, animated: false)
    }
})
```

这样就完成了在app中加入反馈页面的添加。

### Part4 阿里百川控制台配置反馈组件
在阿里百川用户反馈后台，可以在App内反馈中查看用户发送的反馈信息，还可以在Mobile客户端配置中自定义配置app中反馈页面的UI。这是我觉得的该产品的亮点所在，可以实时配置热门问题，减去了在app开发时需要考虑的动态获取生成热门问题的过程，而且整个appUI全都实时动态设置。具体操作大家用一下就很容易了解了。

## 思考
通过使用阿里百川用户反馈组件，我又一次看到了H5的用途。对于这类弱使用强度、高数据变更、低页面逻辑的页面，我觉得H5的用途大大体现了，虽然讲这样的一个反馈的原生页面以及一套后台处理系统没有什么技术实现上的困难点，但是对于大部分app开发来讲，还是需要消耗时间精力来做的，更重要的是这不是用户使用app的刚需，因此这种即插即用可服务端实时配置的H5反馈页面的出现恰到好处。

## 番外篇
push 百川 feedbackVC 的时候，因此 NavigationBar 部分会黑一下再显示。很奇怪的是我如果在 push之前 navigationController 隐藏了 NavigationBar ，这个 feedbackVC 就没有了 NavigationBar。反馈页面其实就是一个 webView，因此我没有搞懂为什么 navigationController 的NavigationBar 显示与否会影响一个 webView 里面 H5 页面的 NavigationBar 显示。这个问题我得再研究研究。



