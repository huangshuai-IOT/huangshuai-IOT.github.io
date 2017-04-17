---
title: iOS Swift Xcode7 的 APNS 使用
date: 2015-12-02 20:55:15
tags:
- iOS
- Swift
- APNS推送
categories:
- 开发
- iOS
---
# iOS9-Swift2-Xcode7的APNS使用

## 前言

最近在开发基于阿里百川的即时通信功能，其中对于iOS平台的消息推送是要使用苹果的APNS服务的，对于这个服务不太了解的同学，可以看[apple开发者文档](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html?spm=0.0.0.0.EUOBVe)学习一下。

## 总结一下使用APNS推送的步骤：

1.iOS应用需要向APNS系统申请DeviceToken

2.应用需要向应用服务器上传DeviceToken

3.服务器通过DeviceToken向APNS系统推送消息

4.APNS系统往用户的手机推送消息

## 实现步骤：

### 1.申请DeviceToken

在AppDelegate.swift中的application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool中添加以下函数向系统申请DeviceToken

```swift
UIApplication.sharedApplication().registerUserNotificationSettings(UIUserNotificationSettings(forTypes: [UIUserNotificationType.Sound , UIUserNotificationType.Alert , UIUserNotificationType.Badge], categories: nil))
UIApplication.sharedApplication().registerForRemoteNotifications()
```

### 2.获取DeviceToken

在AppDelegate.swift中添加下面这个回调函数，用于获取DeviceToken

```swift
func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
    let token:String = deviceToken.description.stringByTrimmingCharactersInSet(NSCharacterSet(charactersInString: "<>"))
    print("token==\(token)")
    //将token发送到服务器
}
```

使用阿里百川的SDK不需要将DeviceToken发送到服务器，IMSDK会自动得到该DeviceToken，你无须手动传给IMSDK

### 3.获取APNS推送证书

在iOSAppIDs你的应用里的证书里，点击edit，打开Push Notifications功能，然后会生成两个证书。Production SSL Certificate和Development SSL Certificate。前面是生产环境推送证书，后面是沙箱环境推送证书。将其下载下来，并安装到钥匙串中。

在钥匙串应用找到这两个证书，右键点击导出，生产p12格式（阿里百川需要p12格式)，在将其上传到阿里百川的控制台。

### 4.测试消息推送
使用[该工具](http://pan.baidu.com/s/1ntngmcL),使用你上传的证书，向设备控制台打印出的DeviceToken，发送一条Push，确认是否可以收到Push。

需要注意的是：项目的证书要和推送证书的类型一致，若是Disturbution证书就得使用Production SSL Certificate，若是Developer证书就得使用Development SSL Certificate。


