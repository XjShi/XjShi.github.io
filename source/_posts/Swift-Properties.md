---
title: 'Swift: Properties'
date: 2016-11-09 23:30:55
tags: Swift
---
属性常与类、结构体、枚举联系在一起。stored properties存储常量值、变量值，而computed properties计算一个值。类、结构体、枚举都可以有computed property，但是只有类与结构体可以有stored property。
## Stored Properties
~~~swift
struct FixedLengthRange {
	var firstValue: Int
	let length: Int
}

var rangeOfThreeItems = FixedLengthRange(firstValue: 0, length: 3)
rangeOfThressItems.firstValue = 6		//ok
let rangeOfFourItems = FixedLengthRange(firstValue: 0, length: 4)
rangeOfFourItems.firstValue = 6	//error
~~~
###Lazy Stored Properties
lazy stored property的值直到第一次使用时才会被计算。

>注意:

>* lazy stored property必须声明为变量，不能是常量。因为这样的属性的值可能会在初始化完成后才获得，而常量属性必须在初始化函数执行完成前有一个值。
>* 这样的属性，在没有被初始化的情况下同时被多个线程同时访问，不能保证这个属性只初始化一次。

~~~swift
class DataImporter {
    var fileName = "data.txt"
}

class DataManager {
    lazy var importer = DataImporter()
    var data = [String]()
    
}

let manager = DataManager()
manager.data.append("Some date")
manager.data.append("Some more data")

print(manager.importer.fileName)	//importer在这里才被创建
~~~
### Stored Properties and Instance Variables
在oc中有两种方式来为类存储值或引用，一是实例变量的方式，二是属性。
Swift统一了这些概念，只有声明属性的方式。Swift中的property没有对应的实例变量。

## Computed Properties
类、结构体、枚举都可以定义computed property，computed property不存储值，相反，他们提供一个getter和一个可选的setter来获取和设置其他属性和值。

~~~swift
struct Point {
    var x = 0.0, y = 0.0
}

struct Size {
    var width = 0.0, height = 0.0
}

struct Rect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2.0)
            let centerY = origin.y + (size.height / 2.0)
            return Point(x: centerX, y: centerY)
        }
        /*
        set(newCenter) {
            origin.x = newCenter.x - (size.width / 2)
            origin.y = newCenter.y - (size.height / 2)
        }
 */
        set {
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}

var square = Rect(origin: Point(x: 0.0, y: 0.0),
                  size:Size(width: 10.0, height:10.0))
let initialSuqareCenter = square.center
square.center = Point(x: 15.0, y: 15.0)
print("quare.origin is now at \(square.origin.x), \(square.origin.y)")
~~~
上面的例子中`Rect`的`center`属性是computed property。

如果setter没有为新值定义一个名字，默认的名字是`newValue`。
### Read-Only Computed Properties
可以通过移除`get`关键字和它对应的括号来简单声明一个read-only computed property，看个例子：

~~~swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
    var volume: Double {
        return width * height * depth
    }
}
~~~

## Property Observers
属性观察者监测并相应属性值的变化。没一次一个属性值被设置的时候，观察者都被调用。

1. 除了lazy stored property，你可以给你定义的任何stored property添加属性观察者。
2. 你也可以通过在子类中重写继承的属性（stored或者computed都可以），来给属性添加属性观察者。
3. 没有必要给没有重写的computed property添加观察者，因为你可以在setter中观察和相应值的变化。

`willSet`在值被存储前调用，`didSet`在新值被存储后立即调用。(newValue,oldValue)

如果你在自己的`didSet`中给属性赋了一个新值，这个新值会替换掉刚刚设置的值。

> The `willSet` and `didSet` observers of superclass properties are called when a property is set in a subclass initializer, after the superclass initializer has been called. They are not called while a class is setting its own properties, before the superclass initializer has been called.

~~~swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("Abount to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}

let stepCounter = StepCounter()
stepCounter.totalSteps = 200
stepCounter.totalSteps = 360
~~~

> 如果你把一个有属性观察者的属性传递给一个函数的in-out参数，那么`willSet`和`didSet`总是会被调用。这是由in-out参数的内存模型决定的。

## Type Properties
type property可以理解为C++中的静态成员变量。
stored type property可以是变量或者常量。computed type property只能声明称变量。

跟stored instance property不一样的是，你必须给stored type property一个默认值。因为类型本身没有初始化函数可以给stored type property赋值。

stored type property是lazy initalized，可以确保在多线程条件下只初始化一次。
### Type Property Syntax
可以使用`static`关键字来定义type property。对于类类型的computed type property，可以使用`class`关键字来允许子类重写父类的实现。

~~~swift
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
~~~