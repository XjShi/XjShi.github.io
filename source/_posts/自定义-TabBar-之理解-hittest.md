---
title: 自定义 TabBar 之理解 hittest
date: 2015-05-03 10:23:16
tags: iOS
---
需求的TabBar是这样的：5个 tabItem， 中间的那个 item 部分超出系统默认TabBar的上边界。

<!-- more -->

那么实现的关键点就是如何在点击它突出的部分的时候，也可以正常获得响应。我来把问题简化，我把下图中的红色的视图（类型为`RedView`，继承自`UIView`）称为 redview，蓝色的视图（类型为`BlueView`，继承自`UIView`）称为 blueview。redview 添加在试图控制器的视图上，blueview 添加在 redview上。

{% asset_img red-blue.png %}

在 RedView 和 BlueView 的`- (UIView *)hitTest:withEvent:`方法和`- (void)touchesBegan:withEvent:`都保持默认实现的情况下，点击 blueview 超出 redview 的部分，两个视图的`- (void)touchesBegan:withEvent:`均不执行。

下面我来改造一下 RedView 的`- (UIView *)hitTest:withEvent:`方法，如下：

~~~objc
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *view = [super hitTest:point withEvent:event];
    UIView *blueview = self.subviews.firstObject;
    if (CGRectContainsPoint(blueview.frame, point)) {
        return blueview;
    }
    return view;
}
~~~

这样重写以后，再点击上边说的那部分， RedView 的 hittesst 就会把 blueview 作为 hittest view，因此会执行到 blueview 的`- (void)touchesBegan:withEvent:`方法。

为什么会这样？

我们可以看一下`- (UIView *)hitTest:withEvent:`的说明(读原文吧，比我描述的清楚)：

> This method traverses the view hierarchy by calling the pointInside:withEvent: method of each subview to determine which subview should receive a touch event. If pointInside:withEvent: returns YES, then the subview’s hierarchy is similarly traversed until the frontmost view containing the specified point is found. If a view does not contain the point, its branch of the view hierarchy is ignored. You rarely need to call this method yourself, but you might override it to hide touch events from subviews.

> This method ignores view objects that are hidden, that have disabled user interactions, or have an alpha level less than 0.01. This method does not take the view’s content into account when determining a hit. Thus, a view can still be returned even if the specified point is in a transparent portion of that view’s content.

> Points that lie outside the receiver’s bounds are never reported as hits, even if they actually lie within one of the receiver’s subviews. This can occur if the current view’s clipsToBounds property is set to NO and the affected subview extends beyond the view’s bounds.

想要跟深入的了解这部分内容，可以阅读 Response Chain 相关的文档。这里贴上地址：<https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html>

到这里如何定义需求中的Tabbar怎么实现就很明显了。直接放个Button在中间就好了，然后调整其他四个TabItem的位置。