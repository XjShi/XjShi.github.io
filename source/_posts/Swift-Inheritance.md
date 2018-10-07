---
title: 'Swift: Inheritance'
date: 2016-11-11 17:17:38
tags: Swift
---
为了在属性值改变的时候获得通知，类可以为继承的属性添加属性观察者。属性观察者可以添加到任何属性上，不管这个属性原来是存储属性还是计算属性。

Swift中的类没有一个统一的基类。

为了讲明白继承，我们先来定义一个基类：

~~~swift
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        
    }
}
~~~

下面定义一个子类继承自`Vehicle`:

~~~swift
class Bicycle: Vehicle {
    var hasBasket = false
}
~~~

## 重写
重写以`override`开头，任何不带`override`关键字的重写，都会在编译时判断为一个错误。
### 访问超类的方法、属性、下标
* 父类有一个被重写的someMethod()方法：`super.someMethod()`
* 父类有一个被重写的someProperty属性：`super.someProperty`
* 父类有一个对于someIndex的下标被重写：`super[someIndex]`

### Override Methods
~~~swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
~~~

### 重写属性的getter/setter
不论原来的属性时存储型还是计算型，你都可以提供一个自定义的getter来重写这个继承的属性。一个属性时存储型，还是计算型，子类并不知道，子类只知道被继承的属性有一个确定的名字和类型。你要规定你重写的属性的名字和类型，来让编译器检查你的重写跟父类中的属性有相同的名字和属性。如果觉得这段不好理解，可以看下原文：

“The stored or computed nature of an inherited property is not known by a subclass—it only knows that the inherited property has a certain name and type. You must always state both the name and the type of the property you are overriding, to enable the compiler to check that your override matches a superclass property with the same name and type.”

Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3).” iBooks. 

看个例子：

~~~swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
~~~

### 重写属性观察者
你可以使用属性重写给继承的属性添加属性观察者。

因为常量存储型属性和只读的计算型属性不能被设置新值，所有不能给这样的属性通过继承添加属性观察者。

~~~swift
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}
~~~

## Perventing Overrides
可以把一个方法、属性、下标标记为`final`来防止其被重写，也可以把真个类用`final`来标记，任何企图重写final标记的，都会报一个运行时错误。