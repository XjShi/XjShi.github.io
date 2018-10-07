---
title: load vs. initialize
date: 2015-10-21 13:13:04
tags: Objective-C
---
这篇文章来对比一下NSObject类的两个方法，`+load`与`+initialize`。

## `+ (void)load;`
Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.

不管一个类、类别是被动态加载或者静态链接，都会被发送load消息，但是只有新加入的类或者类别实现了这个方法，它才可以响应。

The order of initialization is as follows:

1. All initializers in any framework you link to.
2. All `+load` methods in your image.
3. All C++ static initializers and C/C++ `__attribute__(constructer)` functions in your image.
4. All initializers in frameworks that link to you.

注意：

* 一个类的`+load`方法在他所有的父类之后调用；
* 一个类别的`+load`方法，在类本身的`+load`之后调用。

## `+ (void)initialize;`
运行时在每个类或者该类的子类被首次使用时，调用该方法。父类比子类先收到该消息。

运行时以线程安全给每个类发送`initialize`消息。这造成`initialize`会阻塞其他线程。所以，在`initialize`中只做必须的少量工作。

