---
title: Swift的闭包（一）：闭包简介、闭包表达式的优化
date: 2016-11-08 19:19:06
tags: Swift
---
定义：Closures are self-contained blocks of functionality that can be passed around and used in your code. 

跟oc中的block相似。

Capture can capture and store reference to any constants and variables from the context in which they are defined. This is known as closing over those constants and variables.

<!-- more -->

闭包以下面三种形式中的一种存在：

1. 全局函数是一个有名字并且不捕获任何变量的闭包；
2. 嵌套函数（nested function）是有一个名字并且可以从他们的外层函数（enclosing function）捕获变量的闭包；
3. 闭包表达式是没有名字，可以从他们的surrounding context中捕获变量；

Swift的闭包有干净、清晰的格式，并且有足够的优化，包括：

1. 从上下文中推断参数和返回值；
2. Implicit returns from single-expression closures;
3. 简短的参数名字；
4. 尾闭包语法

# 闭包表达式
闭包表达式提供几种语法优化，下面的例子展示通过迭代提炼一个单一的表达式sorted(by:)，来展示这些优化。

## 排序方法
Swift的标准库提供一个叫做sorted(by:)的方法，基于用户提供的一个排序闭包的输出，可以对一个已知类型的数组进行排序。排序完成后，这个方法返回一个新的数组（跟原来的数组有相同的类型和元素数量）。

下面闭包表达式的例子，使用sorted(by:)方法对一组String值进行逆字母序排序。初始的数组如下：

~~~swift
var names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
~~~

sorted(by:)方法接受一个带两个相同类型的参数并返回一个Bool值的闭包。如果第一个值需要出现在第二个值后边，闭包需要返回true，否则，返回false。

这个例子是对一组String值进行排序，所以排序的闭包的需要是一个(String,  String)->Bool类型的函数。

下面用一个正常的函数来作为这个排序闭包：

~~~swift
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward);
~~~

## 闭包表达式的语法
{% asset_img swift-syntax.png %}

闭包表达式里的patameters可以是in-out参数，但不能有默认值。

下面这段代码展示闭包表示式的版本：

~~~swift
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
~~~

## 从上下文推断类型
因为上面例子中的闭包作为一个参数传递给了一个方法，Swift可以推断它参数的类型和返回值的类型。这个sorted(by:)方法是在一个String数组上调用，因此它的参数必须是一个(String, String) -> Bool类型的函数。这意味着(String, String)和Bool类型不需要写出来作为闭包定义的一部分。因为所有的类型都可以推断出来，返回的箭头->和参数两边的小括号也可以被删除:

~~~swift
reversedNames = names.sorted(by: {s1, s2 in return s1 > s2})
~~~

## 从单表达式闭包隐式返回（Implicit Returns fro Single-Expression Closures）
单表达式闭包可以通过删除return关键字来隐式返回这个单表达式的结果：

~~~swift
reversedNames = names.sorted(by: {s1, s2 in s1 > s2})
~~~

## 简化参数名字（Shorthand Argument Names）
Swift自动给内联闭包提供简短的参数名字，通过 `$0`, `$1`, `$2` 等来引用闭包的参数。

如果你在闭包表达式中使用简化的参数名字，你可以从闭包的定义中删除闭包的参数列表，并且简短参数名字的数量和类型将通过预期的函数类型来推断。因为这个闭包表达式完全的组成了它的body，这个in关键字也可以删除。

~~~swift
reversedNames = names.sorted(by: {$0 > $1})
~~~

## 运算符函数（Operator Functions）
实际上，对于上面的例子有一个更简短的写法。Swift的String类型实现了针对String类型的大于操作符作为一个有两个String类型参数和Bool返回值的函数。恰好跟sorted(by:)的需求吻合。因此，你可以简单的传递一个大于操作符，Swift将推断你想使用针对String的实现:

~~~swift
reversedNames = names.sorted(by: >)
~~~

## 尾闭包
如果你需要给一个函数传递一个闭包作为函数的最后的参数，并且闭包表达式很长，写成尾闭包是更好的选择。一个尾闭包被写在函数调用小括号的后边。当你写尾闭包语法的时候，你不用为函数调用的闭包写参数标签。

~~~swift
func someFunctionThatTasksAClosure(closure: () -> Void) {
    
}
//不使用尾闭包的函数调用
someFunctionThatTasksAClosure(closure: {
    //closure's body
})
//使用尾闭包的函数调用
someFunctionThatTasksAClosure() {
    //closure's body
}
~~~

排序的例子使用尾闭包语法可以这么写：

~~~swift
let reversedNames = names.sorted() {
    (s1: String, s2: String) -> Bool in
    return s1 > s2
}
~~~

或者，这样：

~~~swift
reversedNames = names.sorted() {$0 > $1}
~~~

如果闭包表达式是方法的唯一参数，并且你使用表达式作为尾闭包，那么在调用函数没有写在函数名后边的小括号：

~~~swift
reversedNames = names.sorted {$0 > $1}
~~~

----
参考：The Swift Programming Language(Swift 3)