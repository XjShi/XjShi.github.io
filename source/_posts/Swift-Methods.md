---
title: 'Swift: Methods'
date: 2016-11-10 15:50:09
tags: Swift
---
这里的“方法”指的是跟特定类型有关系的函数。

方法包括实例方法(Instance Method)和类型方法(Type Method)。实例方法可以访问所有该类型所有其他的实例方法和属性。
## 实例方法
### 在实例方法中修改值类型的数据(Modifying Value Types from Within Instance Mothod)
结构体和枚举是值类型。默认情况下，值类型的属性不能在它的实例方法中修改，但是，如果你需要这么做，你可以为这个方法选择`mutating`行为。这样这个方法这可以修改它的属性，并且在方法执行结束时把这个修改写回原来的结构体中。甚至，你可以把一个完全新的实例赋值给`self`属性，这个新的实例将完全替换原来的那个。

~~~swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        x += deltaX
        y += deltaY
    }
    mutating func moveBy2(x deltaX: Double, y deltaY: Double) {
        self = Point(x: x + deltaX, y: y + deltaY)
    }
}
~~~
在枚举中，可以把self设置成一个不同的case，如下：

~~~swift
enum TriStateSwitch {
    case off, low, high
    mutating func next() {
        switch self {
        case .off:
            self = .low
        case .low:
            self = .high
        case .high:
            self = .off
        }
    }
}
~~~
## 类型方法
在`func`关键字前加上`static`关键字来表示类方法。对于类，也可以使用`class`关键字来允许子类重写父类的实现。

在类方法中，隐藏的`self`属性指向类型本身。