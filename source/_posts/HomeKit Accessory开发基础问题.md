layout: homekit
title: 非商业用途的 HomeKit 配件开发的常见问题
date: 2017-06-09 11:02:07
tags:
- HomeKit
- HomeKit Accessory Protocol 

categories:
- HomeKit
---

# 前言

前天在 iOS11 的家庭 app 中看到苹果提醒我自己开发的智能设备没有经过认证，还以为苹果会关闭黑 HomeBridge 协议这个HomeKit接口，没想到 WWDC2017 今天公布了 HomeKit 非商业用途的配件的接入协议。可以让个人开发者接入自己的WiFi和BLE类型的智能硬件设备。本文是对官网 FAQs 的整理。

# 基础问题
HomeKit 配件协议（HAP）是苹果专门用来使第三方配件（例如灯、恒湿器和门锁类）和苹果设备（iPhone、iPad、Apple Watch、AppleTV）之间相互通信的协议。HAP 支持两类通信方式：IP 和 BLE 。在 HAP（非商业版）协议中提供了如何为你的非商业用途配件实现 HAP 协议，意思就是我们自己开发的设备怎样以官方认可的方式接入 HomeKit，而不是利用黑 HomeBridge 协议来实现。 

## 如何获得这个协议规范？
HomeKit 配件协议规范可以从[HomeKit开发者网页](https://developer.apple.com/homekit/)下载。在你下载文档之前，你需要用你的 AppleID 登陆，然后同意开发协议。

## 利用本规范开发的配件和利用商业版本的 HomeKit 的配件有什么不同？
使用本规范创建的配件不包含 Apple 认证协处理器或者获得Wi-Fi Alliance 认证，并且不会在MFI项目下完成 HomeKit 认证。对于用户来说，差异主要是基于IP的附件添加过程，并且在用户继续添加的过程中会弹出警告对话框，说明该设备没有经过苹果认证。

## 如何开发生产 HomeKit 的配件来发布销售？
你的公司必须首先参加 MFI 计划，进行商业销售的第三方HomeKit 配件必须在生产销售之前要实现 MFI 计划，包括完成HomeKit 认证。

## 是否可以在我的配件上添加“Works with Apple HomeKit”标志？
不允许，这个标志只能用于 MFI 项目并且完成 HomeKit 认证的商业配件。


# 技术相关问题
## 是否可以通过 iOS 内置的家庭app 和 Siri 来控制我的配件，并且创建家庭场景？
可以，如果你使用 Apple 定义的 HomeKit 服务和本规范中的属性实现HAP协议，并且手机中运行 iOS10 及更高版本，则可以使用家庭app 和 Siri 来控制配件的功能。可以通过[家庭App 使用说明](https://support.apple.com/zh-cn/HT204893)来获得更多帮助。

## 可以创建一个配件实现 Apple 尚未定义的 HomeKit 服务吗？
可以，你可以创建一个使用自定义服务类型，并且实现 HAP 协议的配件。但为了控制这个配件，你需要开发一个使用 HomeKit API 的自定义应用程序。但这样你就无法通过家庭应用程序或者Siri 进行控制你的设备了。

## 我的配件如何与 iOS设备配对？
对于IP类配件，配件和iOS设备首先得在同一个WiFi网络下，联网显示配件的方式依赖于你所使用的硬件开发平台。然后你可以通过家庭App或者你自己开发的App来完成配对过程。对于BLE类配件，可以像你平常与BLE设备配对那样进行配对。

## 我是否可以远程控制我的配件，并且使用家庭App设置自动化操作？
可以，如果配件使用 Apple定义的 HomeKit 服务，并且你将iPad 或者 Apple TV（第四代）设置为家庭中枢，你就可以实现远程控制。有关详细信息，可以查看[自动化和远程访问 HomeKit 配件](https://support.apple.com/zh-cn/HT207057)文档。

## HAP使用了什么安全特性？
HomeKit 配件与 Apple 产品之间通过 HAP 的所有通信都是端到端加密的，并且是相互认证的。

## IP类的 HomeKit 配件是否必须实现 Bonjour？
是的，配件必须实现Bonjour才能被发现。Bonjour代码和规范可以在[Bonjour](https://developer.apple.com/bonjour/)文档中查看。

## BLE 类配件是否必须遵守蓝牙核心规范？
强烈推荐设备遵守蓝牙核心规范，这样可以确保配件的正常运行。

## 是否可以从 Apple 获得 HomeKitSDK？
暂时不行。

## 为什么我的附件在 iOS设备上会有警告对话框？
不包含Apple认证协处理器的配件将在iOS中触发警告对话框，这表示该配件未经 HomeKit 认证，用户可以点击确认，继续使用该配件。

## 是否可以在我的配件中添加一个Apple认证协处理器？
不允许，认证协处理器仅用于商用 HomeKit 配件，该组件只适用于经过MFI认证的公司。

## 在哪可以提问关于实现HAP的问题？
向其他开发者或者智能家居爱好者询问技术问题或者讨论 HomeKit 配件协议，请访问[Apple开发者论坛](https://forums.developer.apple.com/welcome)。





