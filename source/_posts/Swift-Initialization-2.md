---
title: 'Swift: Initialization-2'
date: 2016-11-11 23:24:18
tags: Swift
---
## Failable Initializers
有的时候，可能是参数问题、需要的外部资源没有到位等原因，初始化可能失败。为了应对这种情况，我们可以定义一个或多个可失败的构造方法。 `init？`

A failable initiazlier creates an optional value of the type it initializes.

通过返回nil来表示初始化失败，看个例子：

~~~swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty {
            return nil
        }
        self.species = species
    }
}

let someCreature = Animal(species: "Giraffe")
//someCreature的类型是Animal?
if let giraffe = someCreature {
	//
}
~~~

### Failable Initializers for Enumerations
~~~swift
enum TemperatureUnit {
    case kelvin, celsius, fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .kelvin
        case "C":
            self = .celsius
        case "F":
            self = .fahrenheit
        default:
            return nil
        }
    }
}

let fahrenheitUnit = TemperatureUnit(symbol: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."
 
let unknownUnit = TemperatureUnit(symbol: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
~~~

### Failable Initializers for Enumerations with Raw Values
Enumerations with raw values automatically receive a failable initializer, `init?(rawValue):)`, that takes a parameter called `rawValue` of the appropriate raw-value type and selects a matching enumeration case if one if found, ro triggers an initialization failure if no matching value exists.

把上面的例子改写一下：

~~~swift
enum TemperatureUnit: Character {
    case kelvin = "K", celsius = "C", fahrenheit = "F"
}
 
let fahrenheitUnit = TemperatureUnit(rawValue: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."
 
let unknownUnit = TemperatureUnit(rawValue: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
~~~

### Propagation of Initialization Failure
一个类、结构体、枚举的可失败的构造方法可以delegate across同一个类型中另一个可失败的构造方法。子类的一个可失败的构造方法可以delegate up父类的可失败的构造方法。

一个可失败的构造方法也可以delegate to不可失败的构造方法。

看个例子：

~~~swift
class Product {
    let name: String
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}
 
class CartItem: Product {
    let quantity: Int
    init?(name: String, quantity: Int) {
        if quantity < 1 { return nil }
        self.quantity = quantity
        super.init(name: name)
    }
}
~~~

### Overriding a Failable Initializer
父类的可失败的构造函数可以被重写为可失败的或者不可失败的。看个例子：

~~~swift
class Document {
    var name: String?
    // this initializer creates a document with a nil name value
    init() {}
    // this initializer creates a document with a nonempty name value
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}

class AutomaticallyNamedDocument: Document {
    override init() {
        super.init()
        self.name = "[Untitled]"
    }
    override init(name: String) {
        super.init()
        if name.isEmpty {
            self.name = "[Untitled]"
        } else {
            self.name = name
        }
    }
}
~~~

### The init! Failable Initializer
`init!` implicitly unwrapped optional instance

## Required Initiazliers
在构造方法的定义前边加上`required`修饰符来表示这个类的每一个子类都必须实现这个构造方法：

~~~swift
class SomeClass {
    required init() {
        
    }
}
~~~

当重写一个required指定构造器时，不用在前边加`override`修饰。

注意：**如果你可以使用一个继承的构造方法来满足的话，那么你可以不提供一个显式的required构造方法。**
比如说我们定义一个子类：

~~~swift
class SomeSubClass: SomeClass {
    required init() {
        //
    }
}
~~~
这样写，没有问题。我们也可以这样写：

~~~swift
class SomeSubClass: SomeClass {
    
}
~~~
这样写也没有问题，因为`init()`方法被自动继承了。我们接着看：

~~~swift
class SomeSubClass: SomeClass {
    init(name: String) {
        //
    }
}
~~~
这样因为我们自定义了一个指定构造方法，`SomeSubClass`不会自动继承父类的指定构造方法（`init()`），所以编译器会提示我们一个错误。

## Setting a Default Property
主要用途是为存储属性进行一些处理，可以使用全局函数或者闭包。看起来大概是这个样子：

~~~swift
class SomeClass {
    let someProperty: SomeType = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
~~~