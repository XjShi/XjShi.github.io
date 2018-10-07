---
title: Objective-C 中的 BOOL
date: 2016-03-02 21:30:45
tags: Objective-C
---
之前开发了一个针对单个数据模型，自动建表、增删改查等操作的小工具，后边在 iPhone 5c 上使用时，出现了 crash 的情况。

经定位，问题就出在了模型中的 BOOL 类型的属性上。

<!-- more -->

看下 BOOL 在 objc.h 中的定义：

~~~objc
/// Type to represent a boolean value.
#if (TARGET_OS_IPHONE && __LP64__)  ||  TARGET_OS_WATCH
#define OBJC_BOOL_IS_BOOL 1
typedef bool BOOL;
#else
#define OBJC_BOOL_IS_CHAR 1
typedef signed char BOOL; 
// BOOL is explicitly signed so @encode(BOOL) == "c" rather than "C" 
// even if -funsigned-char is used.
#endif
~~~

显而易见，在64位系统下，实际上时 bool 类型；而在32位系统下，是 signed char 类型的。