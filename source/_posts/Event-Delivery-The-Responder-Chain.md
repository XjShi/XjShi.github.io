---
title: 'Event Delivery: The Responder Chain'
date: 2015-04-04 20:08:43
tags: iOS
---
当我们设计语言的时候，我们很可能想动态的响应事件。例如，触摸一个拥有许多不同对象的屏幕，你要决定给哪个对象一个响应事件，怎么样对象接收到事件。

当一个用户产生事件发生时（如 点击），UIKit 产生一个对象，这个包含要被处理的对象的信息。接着把事件对象放到 active app's（一个application对象，负责初始化用户事件路由、从全局管理一个运行的application）事件队列中。对于触摸事件，这个对象是一组触摸，打包放在 UIEvent 对象。对 motion 事件而言，事件对象根据你使用框架的不同和你感兴趣的 motion 事件类型而有所差异。

一个事件对象沿一个明确的路径传递，直到有个对象可以处理它。首先，单例 UIApplication 对象从队列的顶部取一个事件，并进行分发处理。一般，UIApplication 对象把事件发给应用的 key window 对象，key window 对象传递给一个 initial 对象来处理。这个 initial 对象取决于事件的类型。

* Touch events。对于触摸事件，window 对象首先尝试将事件发送给事件发生的 view（hit-test view）。找到 hit-test view 的过程被称为 hit-testing（我们将在后边谈到）。
* Motioin and remote control enents。这类事件，window 对象发送 shaking-motion 或者 remote control event 给第一响应者处理。

事件路径（event paths）的终极目标是找到一个对象来处理响应一个事件。因此，UIKit 首先把这个事件发送给最适合处理这个时间的对象。对于触摸事件，这个对象是 hit-test view；对于其他事件，这个对象是第一响应者。下面的部分介绍关于确定 hit-test view 和第一响应者的更多细节。

Hit-Testing Returns the View Where a Touch Occurred（hit-test 返回触摸发生的 view）。有的地方翻译感觉很别扭，为了避免误导（真的有人看吗？），原文贴上。

iOS use hit-testing fo find the view that is under a touch. Hit-testing involves checking whether a touch is within the bounds fo any relevant（更多强调直接相关） view objects. If it is, it recursively checks all of that view's subviews. The lowest view in the view hierarchy that contains the touch point becomes the hit-test view. After iOS determines the hit-test view, it passes the touch event to that view for handling.

举例说明，假设用户在 View E 中触摸，iOS通过用下面的次序检查 subviews 来查找 hit-test view：

触摸在 view A 的 bounds 内，所以检查 subview B 和 C；
触摸不在 View B 的 bounds 内，但在 View C 的 bounds 内，所以检查 subview D 和 E；
触摸不在 View D 的 bounds 内，但在 view E 的 bounds 内。
View E is the lowest view in the view hierarchy that contains the touch, so it becomes the hit-test view.

{% asset_img hit-test.png %}

`hitTest:withEvent:` 方法返回一个 CGPoint 和 UIEvent 的 hit-test view。 `hitTest:withEvent:` 方法开始执行通过 view 本身调用 `pointInside:withEvent:` 方法。如果传入`pointInside:withEvent:`的点在 view 的 bounds 内，`pointInside:withEvent:`返回 YES。接着，在这个 view 的每个 subview 上调用 `hitTest:withEvent:`。

传入`hitTest:withEvent:`的点不在 view 的 bounds 中得情况不细说了。

注意：一个触摸事件在它的整个生命周期中和它的 hit-test view 相关联，即使稍后触摸移出了这个 view。

hit-test view 被给予了第一次来处理触摸事件的机会。如果 hit-test view 不能处理这个事件，事件将沿这个 view 的响应者链条往下走，直到找到一个可以处理事件的对象。

 The Responder Chain Is Made Up of Responder Objects（响应者链条由响应者组成-。-）

许多类型的事件以来响应链条传递事件。响应者链条是一系列连起来的响应者对象。以第一响应者开始，结束于 application 对象。如果第一响应者不能处理一个事件，它就转发给响应者链条中的下一个响应者。

A responder object is an object that can respond to and handle events. The  UIResponder  class is the base class for all responder objects, and it defines the programmatic interface not only for event handling but also for common responder behavior. Instances of the UIApplication, UIViewController, and UIView classes are responders, which means that all views and most key controller objects are responders. Note that Core Animation layers are not responders.

第一响应者被指定首先接受事件。通常，第一响应者是一个 view 对象。一个对象可以变成第一响应者通过做下面两件事：

重写 canBecomeFirstResponder 方法，返回YES;
接受一个 becomeFirstResponder 消息。如果必要，一个对象可以给自己发送这条消息。
依赖响应者链条的对象不止事件对象。响应者链条可以用在下面的地方：

1. Touch events.
2. Motion events.
3. Remote control events.
4. Action messages.
5. Editing-menu messages.
6. Text editing.

UIKit自动设置用户点击的text field 或 text view成为第一响应者；Apps必须显式设置所有其他的第一响应者，用becomeFirstResponder方法。

更多详细内容可查阅苹果开发者文档：
<https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html>