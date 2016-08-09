---
title: iOS:阿里百川即时通讯功能
date: 2015-12-02 20:55:15
tags:
- iOS
- Swift
- 即时通讯
- 阿里百川
categories:
- 开发
- iOS
---

## 前言
   
在[iOS-webApp客户端开发小记](../../../../2015/11/06/iOS-webApp客户端开发-小记/) 的最后，我提到我们公司的产品需要在现有产品结构中加入即时通讯功能。经过对现有第三方即时通讯产品的试用和分析，我们最终选择了阿里百川提供的及时通讯功能。

## 项目需求

1.学生与老师之间要能沟通（单聊）

2.在同一门课程中，老师与学生们在一个聊天群里（群聊）

3.学生和老师要可以和我们公司的技术支持人员能沟通（客服）


## 技术选型

前期调研了容联云、LeanCloud、环信、Bmob、阿里百川、友盟等一些做即时通信服务的第三方服务。首先我们需要其提供iOS、Android、h5、Java的SDK，接着试用了他们的Demo，然后看他们的开发文档是否详尽（公司与公司之间的开发文档真的差距好大），最后看他们的技术实力与公司实力（总不能比我们的产品先倒闭吧，哈哈）。

最后选中了阿里百川，首先其是阿里无线事业部下属的，他们的技术实力和公司实力不用怀疑。其实我第一眼看中的还是他们的开发文档，非常的完善，其次是这个项目应该开始不久，他们的技术支持非常给力，一线的开发人员在充当客服，在开发的时候还跟他们打了好几通电话，技术支持非常的棒。

## 项目开发

这里讲的是iOS端如何整合阿里百川的SDK。我采用的是Swift开发，而阿里百川提供的官方文档是针对Objective-C项目的，所以在此记录一下Swift项目是怎样整合的。下面主要介绍快速集成和APNS推送，相信通过这两点，就会对如何在Swift中调用IMSDK相关功能有个了解，那么其他功能调用也会容易实现。

### 1.快速集成

#### Step1-Step5 引入IMSDK
按照阿里百川的即时通信服务的快速集成文档，完成Step1-Step5都是没有什么问题的。百川提供了非常好的胶水代码（将我们的App和IMSDK粘合起来的中间代码），其中包含对IMSDK主流程接口的调用代码，例如初始化、登录准备、登录、注销、打开会话列表、打开聊天页面。
通过调用、学习胶水代码可以为我们后期自己直接操作IMSDK来定制功能提供方向。

在拖入胶水代码后就是初始化IMSDK，由于胶水代码是OC文件，在Swift项目中需要建立桥接头文件，这里可以偷个懒，直接在项目中新建一个OC文件，然后会自动生成一个桥接头文件，然后再把刚才这个OC文件删除即可，这样可以省去在项目中配置桥接头文件这一步。接着，在桥接头文件中，引入胶水代码的头文件：`#import "SPKitExample.h"`

#### Step6 初始化IMSDK

在AppDelegate.swift调用基础入口胶水函数：`callThisInDidFinishLaunching`

``` swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    // Override point for customization after application launch.
    //函数中初始化IMSDK
        SPKitExample.sharedInstance().callThisInDidFinishLaunching()
    return true
}
```
注意：要到SPKitExample.m里exampleInit登录函数这把key换成我们自己申请的AppKey

#### Step7 登录IMSDK

在用户登录我们自己App的账号成功后，调用基础入口胶水函数callThisAfterISVAccountLoginSuccessWithYWLoginId，使其登录IMSDK。在应用里，我用Swift稍微封装了一下登录的代码:

```swift
func loginBaichuan(userid:String,password:String) {
    //应用登陆成功后，调用SDK
        SPKitExample.sharedInstance().callThisAfterISVAccountLoginSuccessWithYWLoginId(userid, passWord:password, preloginedBlock: nil, successBlock: {() -> Void in (
print("哈哈哈，登录成功！"))}, failedBlock: nil)
}
```

注意：在用户退出我们的App之后，还需要使其退出IMSDK，不然App在后台，用户没有登录我们的账号体系，但是还会受到来自即时通讯功能的新消息推送。这是退出的代码：

`SPKitExample.sharedInstance().callThisBeforeISVAccountLogout()`

#### Step8 打开会话列表和聊天页面

1. 打开会话列表

	我们app采用的是UINavigationController方式，所以打开会话列表的代码如下：

	```swift
func openTalkList(){
    let result:YWConversationListViewController = SPKitExample.sharedInstance().ywIMKit.makeConversationListViewController()
    result.title = "最近联系人"
    result.setDidSelectItemBlock { (YWConversation) -> Void in (
        SPKitExample.sharedInstance().exampleOpenConversationViewControllerWithConversation(YWConversation, fromNavigationController: self.navigationController))
    }
    self.navigationController?.navigationBarHidden = false
    self.navigationController?.pushViewController(result, animated: true)
}
	```

2. 打开单聊页面

	我直接调用是胶水函数里面的`exampleOpenConversationViewControllerWithPerson`方法
	
	```swift
func talk2Teacher(teacherId:String) { 
    SPKitExample.sharedInstance().exampleOpenConversationViewControllerWithPerson(YWPerson(personId:teacherId),fromNavigationController:self.navigationController)
}
	```

3. 打开群聊界面

    阿里百川的文档里面写的是调用胶水函数里面的`exampleOpenConversationViewControllerWithTribe`方法，但是该方法要求传入一个`YWTribe`类型的群组聊天变量。由于阿里百川即时通讯的架构是自建账号体系，所以我们的App只能从服务器处获取当前用户所在群的群ID，但是胶水函数里面没有现成的通过群ID打开群聊的功能，所以我通过看胶水代码以及IMSDK的群服务相关的头文件`IYWTribeService.h`，找到`IMCore.getTribeService().requestTribeFromServer`函数，它可以返回一个`YWTribe`类型的群变量。所以我先通过该函数根据群ID查询到群信息，并且根据YWTribe类型的群变量再调用`exampleOpenConversationViewControllerWithTribe`方法。
 
	```swift
func talk2Group(groudId:String) { 
    SPKitExample.sharedInstance().ywIMKit.IMCore.getTribeService().requestTribeFromServer(groudId, completion: { (YWTribe, NSError) -> Void in SPKitExample.sharedInstance().exampleOpenConversationViewControllerWithTribe(YWTribe,fromNavigationController:self.navigationController))
}
```

	我觉得，胶水函数里既然有根据对方的用户ID来打开单聊界面的封装函数，就应该有这样一个根据群ID来打开群聊界面的封装函数。当时，由于对IMSDK结构的不熟悉，这个功能我还花了一些时间来找接口实现。

### 2.APNS推送

我在另外一篇博文里介绍了在iOS9-Swift2-Xcode7环境下的APNS推送相关的内容，这里介绍一下具体的在阿里百川环境下的实现。

#### Step1 制作并上传证书

申请推送证书这个百度有很多教程，这里不在赘述了，反正要注意阿里百川的后台需要的生产环境的证书。

#### Step2 申请DeviceToken

首先需要在`AppDelegate.swift`文件的`func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool`函数里添加以下两句来向Apple的APNS服务器注册申请DeviceToken，这里可以设置新消息推送来App的展现形式，有Sound、Alert、Badge三种。

```swift
UIApplication.sharedApplication().registerUserNotificationSettings(UIUserNotificationSettings(forTypes: [UIUserNotificationType.Sound , UIUserNotificationType.Alert , UIUserNotificationType.Badge], categories: nil))
UIApplication.sharedApplication().registerForRemoteNotifications()
```

接着添加注册申请DeviceToken成功的回调函数：


```swift
func application(application:UIApplication,didRegisterForRemoteNotificationsWithDeviceToken deviceToken:NSData) {
    let token:String = deviceToken.description.stringByTrimmingCharactersInSet(NSCharacterSet(charactersInString: "<>"))
    print("token==\(token)")
    //将token发送到服务器
}
```
注意：IMSDK会自动得到该DeviceToken，你无须手动传给IMSDK

#### Step3 处理APNS消息

由于初始化SDK的时候就调用了`SPKitExample.sharedInstance().callThisInDidFinishLaunching()`函数，在`callThisInDidFinishLaunching`里面已经调用了`exampleHandleAPNSPush()`,所以就不用像文档里面说的那样再在`AppDelegate didFinishLoadingWithOptions`调用`IYWPushService`的`setHandlePushBlockV3:`方法。

#### Step4 Xcode设置为Distribution证书的AdHoc Provision打包

由于阿里百川使用的生产环节下的推送证书，而我们又不能等到应用上线到AppStore后再测试，所以这里Provisioning Profiles使用AdHoc Provision，如何申请百度也有很多教程，这里就不在赘述。

这里讲一下Xcode7设置打包环境的过程：

首先设置Project下的Code Signing：
Code Signing Identity 全部设置为Distribution证书
Provisioning Profiles设置为申请的Distribution-AdHoc Provision证书

再设置Target下的Code Signing：
Code Signing Identity 全部设置为Automatic证书
Provisioning Profiles设置为申请的Distribution-AdHoc Provision证书

在这里我遇到很多坑，经过不断的测试才发现这样的设置可以正常使用。这里我有一个疑问，就是我在第一次使用阿里百川APNS推送时，对证书不是很了解，都没有申请Distribution-AdHoc Provision，糊里糊涂就实现了推送，但是后来不知道自己怎么改动了设置，APNS就不能推送到app了。后期我学习了苹果的证书系统，申请了Distribution-AdHoc Provision证书，才又实现了APNS推送，关于第一次是如何实现功能的，我到现在都没弄明白。

#### Q&A

如果App在后台收不到推送的消息，请参考我那篇博文以及百川的文档，里面有教我们如何利用APNS调试工具通过使用我们的证书，手动Push一条消息到我们的App。通过此方法检测推送证书和DeviceToken是否正确。如果手动推送可以，则说明App证书打包以及申请DeviceToken都是没有问题的，此时应该检测一下提交到阿里百川后台的证书有没有问题。

## 项目总结

即时通讯功能虽然只是我开发的app中的一个功能，但通过在项目中引用阿里百川即时通讯服务，让我懂了一点如何在Swift项目调用OC-SDK，这为我调用更多第三方OC语言SDK提供了基础。

在这里非常感谢阿里百川-即时通讯的技术支持在我开发过程中提供的帮助！


