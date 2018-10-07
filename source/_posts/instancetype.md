---
title: instancetype
date: 2015-02-28 20:09:49
tags: Objective-C
---

在 OC 中，约定（convention）不仅仅是编码时的最佳时间，也是对编译器的隐式说明。

比如，`alloc`和`init`都返回`id`类型，然而在 Xcode 中，编译器要做类型检查。这是怎么做到的？

<!-- more -->

在 Cocoa 中，有这么个约定：像`alloc`、`init`这样的方法名，返回接受者类型的实例。这些方法叫做有一个关联的结果类型（related result type）。

类的构造方法也返回`id`类型，但是却没有得到类型检查的益处，因为类构造方法没有遵从命名约定。

看个例子：

~~~objc
[[[NSArray alloc] init] mediaPlaybackAllowsAirPlay]; // ❗ "No visible @interface for `NSArray` declares the selector `mediaPlaybackAllowsAirPlay`"

[[NSArray array] mediaPlaybackAllowsAirPlay]; // (No error)
~~~

因为`alloc`和`init`遵从命名约定，类型检查返回`NSArray`执行后边的操作。然而，等价的类构造方法`array`不遵从约定，被解释为id类型。

`id`在不需要确保类型安全的情况下非常有用，但如果需要保证类型安全，`id`就没什么用了。

有一个替代的方法，可以显式的声明返回类型（在前边的例子中是`NSArray`）。但是如果要子类化，这样写就不太好。

这时编译器开始解决 OC 的类型系统的这种临界情况：

`instancetype`是一个上下文相关的关键字，可以用于返回类型来表示一个方法返回一个关联的结果类型。如：

~~~objc
@interface Person
+ (instancetype)personWithName:(NSString *)name;
@end
~~~

> 跟`id`类型不一样，`id`可以还可以用于函数参数，`instancetype`只能用于函数声明中的返回类型。

使用`instancetype`，编辑器能正确的推断`+personWithName：`返回了一个`Person`的实例。

译自：[nshipster 的 instancetype](http://nshipster.com/instancetype/) 一文。

转载请注明出处：<https://xjshi.github.io>