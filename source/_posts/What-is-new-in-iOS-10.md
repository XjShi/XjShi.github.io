---
title: What is new in iOS 10
date: 2016-06-18 20:12:04
tags: iOS
---
个人认为 iOS 10 是 iOS 7 之后最大的改版，刚装上测试版的时候，感觉是新奇又有一点不适应。

下面主要来看一下开发者需要关心的部分。
## Providing Haptic Feedback
主要是体现在 iPhone 7 和 iPhone 7 Plus 两款设备上。为此，`UIKit`中引入了`UIFeedbackGenerator`这个类和三个子类，这些类能让触觉对特定的场景更加合适。

类名 | 使用示例
--- | ---
UIImpactFeedbackGenerator | 对完成一个功能或任务的视觉反馈提供一个物理的体验。比如，一个视图滑到指定位置时，或者两个对象碰撞时，用户可能感到砰地一声。
UINotificationFeedbackGenerator | 比如，存入支票、解锁汽车这样的任务（功能）完成、失败或者产生某种类型的警告。
UISelectionFeedbackGenerator | 暗示选择在积极得改变。例如，用户轻触一个滚动的picker。

使用的时候，让系统对指定的场景生成触感（haptic），然后 iOS 会给予场景管理反馈的强度和行为。

## SirtKit
SiriKit 让特定领域的应用可以通过 iOS 中的 Siri 来使用服务。如果应用要接入的话，需要通过 Intents Extension来实现。目前 SiriKit 支持下面这几种领域：

* 语音通话、视频通话
* 短信息
* 支付、接收付款
* 搜索照片
* 约车
* 训练管理
* <del>car play(automotive vendors only)<del>
* <del>餐馆预订（需要苹果的额外支持）<del>

## iMessage App
iOS 10 中，可以通过开发应用扩展来来 Messages 这个应用中发送文本，表情，多媒体文件，甚至是可交互的消息。

可以开发两种类型的应用扩展：

* Sticker pack： 提供一组可以在 Messages 应用中使用的表情。
* iMessage app： 可以在 Messages 应用中展现自定义的用户交互。

开发 Sticker pack 不需要写然和代码，只要把图片拖到 asset 目录中就可以了。  
开发 iMessage app 需要 Message 框架中的相关 API。

## 语音识别
iOS 10 引入了新的 API 来支持语音识别，并可以转为为子。通过 Speech 框架中的相关 API，即可以进行实时转换，也可以从录音文件中转换。


## CallKit
对 VoIP 应用是一大福音，同时也能够满足 iOS 用户期待已久的功能----知道呼入者是快递，还是房产中介。腾讯手机管家已经通过 CallKit 实现了后边描述的这个功能。

## User Notifications
iOS 10 之前，设备收到通知时，只能选择清除或者通过通知打开应用。iOS 10 引入了 UserNotification 框架，支持分发、处理应用。应用可以使用这个框架来接收或者在设备收到通知时悄悄的改变通知。

iOS 10 同时也引入了 UserNotificationsUI 框架，能让你通过扩展自定义通过的外观。

## App Extensions
iOS 10 引入了一些新玩意，以便创建应用扩展，千遍已经提到了一些，这里再次列举一下：

* Call Directory
* Intents
* Intents UI
* Messages
* Notification Content
* Notification Service
* Sticker Pack

到这里 iOS 10 新增的部分基本介绍完了，除了这些，iOS 10 还对已有的部分框架做了增强，可自行到苹果开发者网站查阅。

iOS 9.3 到 iOS 10 的 API 变化可以看这里：<https://developer.apple.com/library/content/releasenotes/General/iOS10APIDiffs/index.html>