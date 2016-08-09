---
title: Android:阿里百川、友盟推送SDK集成问题
date: 2015-12-07 15:50:15
tags:
- Android
- 阿里百川
- 友盟推送
categories:
- 开发
- iOS
---
## 前言

我在安卓项目中先集成了阿里百川即时通讯的SDK，后又集成友盟的消息推送SDK，项目编译的时候就会报以下错误。

```java
Multiple dex files define Lcom/ta/utdid2/android/utils/UTDID
```

报错说的很清楚:是项目引用的JAR包有重复的地方。后发现集成阿里系的SDK（支付宝等）都会出现此错误。

## 错误原因

1.友盟推送SDK为了提高设备标识的唯一性，除了UMID之外，还用到了Taobao提供的一个设备标识生成库(UTDID.jar)做双向保证。当前推送SDK兼容的UDID版本是V1.1.0。

2.淘宝提供的设备标识生成库(UTDID.jar)已经广泛应用在阿里系的App中了，包括支付宝。部分集成过支付宝SDK的App，在集成友盟推送SDK的时候，会存在包冲突的问题。 为此我们提供了去UTDID版本的SDK供开发者集成使用

## 解决方法

1.到友盟[SDK下载页](http://dev.umeng.com/push/android/sdk-download)去下载去UTDID版本的SDK，再集成到安卓项目中


