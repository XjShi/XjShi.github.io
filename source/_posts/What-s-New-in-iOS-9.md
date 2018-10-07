---
title: What's New in iOS 9
date: 2015-06-14 10:05:49
tags: iOS
---
这篇文章主要总结开发者需要关注 iOS 9 中引入的新特性。

## iPad 多任务增强
iOS 9 增强了用户在 iPad 上的多任务体验（Slide Over、Split View、Picture in Picture）。Slide Over 能让用户选择一个次要的应用，并与之交互。Split View 让用户能够同时使用两个部分（仅部分型号 iPad ，如 ipad Air 2）。PiP 能够让用户在使用其他应用的时候，观看视频。

对于 Slide Over 和 Split View ，开发者需要注意应用的内存展映，以及做好适配（size class）。

开发 PiP 的话，需要用 AVKit 或 AVFoundation 相关 API，以前的多媒体播放器 API 不支持 PiP。

## 3D Touch
3D Touch 给用了一种额外的交互方式。在支持的设备上，在主屏（home screen）上可以通过按压应用的图标来选择应用指定的动作。在应用内部，用户可以通过不同的压力级别，进行不同的行为。

## 搜索
iOS 9 中的搜索能够让用户访问到应用内的信息。在应用内容能够被搜索的时候，用户可以通过 Spotlight、Handoff、Siri suggestion 来访问应用内的内容。使用相关的 API，应用开发者来决定什么内容可以被索引，什么信息可以展示的搜索结构中，以及用户点击搜索结果时，跳转到哪里。

隐私也是 iOS 9 的一个特性。在给用户搜索应用信息的同时，也保护隐私数据。

## App Thinning
App thinning 帮助开发者针对不同平台开发应用，并且以优化的方式分发安装。App thinning 包含以下元素：

* Slicing。
* On-Demand Resources。许多应用的额外信息放在 iTunes App Store 的仓库中，允许应用异步下载、安装。
* BitCode。把你提交到 App Store 的应用打包成目标设备的可执行文件。

## ATS
ATS 能加强应用与后端之间连接的安全性。

苹果建议尽快适配 ATS，不管是新开发的应用，还是维护以前的应用。如果是新开发的应用，你应该只是用 HTTPS。如果应用中需要跟不支持 HTTPS的服务器通信，你需要在 Info.plist 文件中指明。

## Extension Points
> extension point定义了使用策略，并且提供相关模块的 API。

iOS 9 引入了几个新的 extension point，包括：

* Network extension points：

	* 使用 Packet Tunnel Provider 扩展点来实现客户端的自定义 VPN 通道协议。
	* 使用 App Proxy Provider 扩展点来实现客户端的自定义透明网络代理协议。
	* 使用 Filter Data Provider 和 Filter Control Provider 扩展点来实现动态的、设备上网络内容过滤。
	每一个网络扩展带你都需要苹果的特殊权限。

* Safari extension points：

	* 使用 Shared Link 扩展点能让用户在 Safari 的分享链接中看到你的内容。
	* Content Blocking 扩展点给 Safari 一个黑名单，这个黑名单描述了你想要阻止你的用户浏览的内容。

* Index Maintenance 扩展点支持在不重新打开应用的情况下，重建应用数据的索引。
* Audio Unit 扩展点允许你的应用提供乐器、音效、声音发生器等。

## Contacts and Contants UI
iOS 9 引入的 Contacts 和 Contants UI 来替换 AddressBook 与 AddressBook UI，与后者比，前者更加面向对象。

## Keychain
这部分不太好描述，直接引入苹果官方文档的原文。

The keychain provides more item protection options and a new type of encryption keys owned by the secure enclave. Specifically:

* New constraints for access control lists that allow creating constraints with Touch ID only or passcode only.
* A new Touch ID constraint that invalidates keychain items when a fingerprint is added or removed.
* Support for app-provided entropy for keychain item encryption using the Application Password option of the access control list.
* Support for an authentication context that lets you invoke the authentication separately from SecItem calls.
* Support for keys generated and used inside the secure enclave using the kSecAttrTokenIDSecureEnclave attribute. Note that access to these keys can be controlled by all constraints supported by access control lists

## Support for Right-to-Left Languages
iOS 9 全面支持从右向左书写的语音，这会让 UI 开发更简单。比如：

* 标准的 UIKit 控件在从右至左的情况下自动反转。
* `UIView` 定义语义上的内容属性，能够让你指定在从右至左的上下文中视图应该如何展示。
* `UIImage`添加了`imageFlippedForRightToLeftLayoutDirection`方法，让反转图片更简单。

除了上边描述的，还有部分框架的改动，具体的 API 变化，可以看这里：[iOS 9.0 API Diffs](https://developer.apple.com/library/content/releasenotes/General/iOS90APIDiffs/index.html#//apple_ref/doc/uid/TP40016222)