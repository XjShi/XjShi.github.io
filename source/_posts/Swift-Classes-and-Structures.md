---
title: 'Swift: Classes and Structures'
date: 2016-11-10 22:09:01
tags: Swift
---
## 对比类与结构体
类与结构体有许多的相同点，它们都可以：  

* 定义属性来存储值；
* 定义方法来提供功能；
* 定义下标操作；
* 定义初始化函数；
* 扩展它的默认的实现；
* 遵从协议；

类有一些额外的能力，但是结构体没有：

* 继承；
* 类型转换可以让你在运行时check、interpret类的实例的类型；
* 析构函数（Deinitializer）；
* 引用计数允许不止一个指向类实例的引用；

**结构体的传递总是复制，不使用引用计数**

<!-- more -->

### 结构体的成员逐一初始化函数
所有的结构体都有一个自动生成的*memberwise initializer*，看下这个例子：

~~~swift
struct Resolution {
	var width = 0
	var height = 0
}
~~~
我们可以这样直接生成一个Resolution:

~~~swift
let someResolution = Resolution()
~~~
也可以使用*memberwise initializer*来初始化：

~~~swift
let vga = Resolution(width: 640, height: 480)
~~~

## 结构体和枚举是值类型(Value Type)
**所谓值类型，是指当它被赋值给一个变量或常量，或者传递给一个函数时，它的值被复制。**在Swift中，所有的基本类型，包括整型、浮点型、布尔型、字符串、数组、字典都是值类型，他们都是使用结构体实现的。Swift中所有的结构体、枚举也都是值类型。
## 类是引用类型(Reference Type)
当给常量或者变量赋值，或者传递给函数的时候，引用类型不会被复制。
### Identity Operators
用于确定两个常量或变量是不是指向（refer to）一个相同的类对象。  

* Identical to: `===`
* Not identical to: `!===`

### 指针
If you have experience with C, C++, or Objective-C, you may know that these languages use pointers to refer to addresses in memory. A Swift constant or variable that refers to an instance of some reference type is similar to a pointer in C, but is not a direct pointer to an address in memory, and does not require you to write an asterisk (*) to indicate that you are creating a reference. Instead, these references are defined like any other constant or variable in Swift.