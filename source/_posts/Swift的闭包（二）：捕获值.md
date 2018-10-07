---
title: Swift的闭包（二）：捕获值
date: 2016-11-08 17:08:00
tags: Swift
---
闭包可以从定义它的上下文中捕获常量和变量。

在Swift中，捕获值最简单的例子是嵌套函数，举个例子：

~~~swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
~~~

<!-- more -->

在这个例子中incrementer()捕获两个值，分别是amount、runningTotal。可以运行一下，观察结果：

~~~swift
let incrementByTen = makeIncrementer(forIncrement: 10)
print(incrementByTen())     //10
print(incrementByTen())     //20
let incrementByNine = makeIncrementer(forIncrement: 9)
print(incrementByNine())    //9
print(incrementByNine())    //18
print(incrementByTen())     //30
~~~

注意：如果你把闭包赋值给一个类实例的一个属性，并且闭包通过指向（refer fo）实例或者实例的成员捕获值，那么，在闭包和这个实例间就会有一个强引用环。 

## 闭包是引用类型（Reference Type）
闭包和函数都是引用类型。

## Nonescaping Closures
当一个闭包作为参数传递给一个函数，但是在函数返回后调用的时候，我们说一个闭包是escaped的。当你声明一个有一个闭包作为参数的函数的时候，你可以在参数类型前写@nonescape来暗示这个closure不允许escape。如：

~~~swift
func someFunctionWithNonescapingClosure(closure: @noescape () -> Void) {
    closure()
}
~~~

把一个闭包标记用@nonescape让你在闭包内隐式的引用（refer to）self，看下这个例子：

~~~swift
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithNonescapingClosure { x = 200 }
        someFunctionWithEscapingClosure { self.x = 100 }
    }
}
 
let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"
 
completionHandlers.first?()
print(instance.x)
// Prints "100"
~~~

## Autoclosures
An autoclosure is a closure that is automatically created to wrap an expression that's being passed as an argument to a function. It doesn't take any arguments, and when it's called, it returns the value of the expression that's wrapped inside of it.

Autoclosures可以延迟计算（delay evaluation），因为直到调用闭包时，闭包内的代码才被运行。延迟计算对于有副作用或者计算代价昂贵的代码非常有用，因为你可以控制什么时候代码进行evaluation。

~~~swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"
 
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"
 
print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
~~~

也可以传递给一个参数：

~~~swift
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Alex!"
~~~

使用@autoclosure：

~~~swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
~~~

注意：滥用autoclosure会使代码晦涩难懂。

@autoclosure属性隐含了@nonescape属性，如果你想要一个autoclosure允许esacpe，可以这样使用 @autoclosure(escaping) ，如：

~~~swift
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure(escaping) () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))
 
print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!"
// Prints "Now serving Daniella!"
~~~