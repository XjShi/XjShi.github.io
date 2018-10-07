---
title: 'Siwft: Initialization-1'
date: 2016-11-11 18:12:05
tags: Swift
---
初始化的过程包括为每一个存储属性设置一个初始值和其他步骤。通过定义构造函数来实现初始化的过程，跟oc的初始化函数不同，Swift的构造函数不返回一个值。它们的主要角色是确保一个类型的实例在初次使用前被正确的初始化。

<!-- more -->

类的实例也可以有析构函数，析构函数在类的实例在释放前完成一些清理工作。
## Setting Initial Values for Stored Properties
类和结构体必须为它们所有的存储属性设置一个初始值，在类或结构体的实例创建完成前。存储属性不能是不确定的状态。

可以通过构造函数或默认值的方式给存储属性一个初始值，而且通过这两种方式，属性的值都是被直接设置，不会调用属性观察者。
### Initializers（构造函数）
~~~swift
init() {

}
~~~

## Customizing Initialization（自定义初始化函数）
你可以自定义初始化过程，通过使用输入参数，可选的属性类型，或者在初始化期间给常量属性赋值。具体描述在下面的部分中描述。
### 1. Initialization Parameters
你可以为构造函数提供参数，看个例子：

~~~swift
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
    init(_ celsius: Double) {
        temperatureInCelsius = celsius
    }
}

let boilingPointOfWater = Celsius(fromFahrenheit: 212.0)
let freezingPointOfWater = Celsius(fromKelvin: 273.5)

let bodyTemperature = Celsius(37.0)
~~~
#### Parameter Names and Argument Labels
Swift provides an autoamtic argument label for every parameter in an initializer if you don't provide one.看个例子：

~~~swift
struct Color {
    var red, green, blue: Double
    init(red: Double, green: Double, blue: Double) {
        self.red = red
        self.green = green
        self.blue = blue
    }
    init(white: Double) {
        red = white
        green = white
        blue = white
    }
}
~~~

我们可以这样初始化一个`Color`的实例：

~~~swift
let megenta = Color(red: 1.0, green: 0.0, blue: 1.0)
let halfGray = Color(white: 0.5)
~~~
#### Initializer Parameters Without Argument Labels
参考上边`Celsius `类中的第三个构造方法，及`bodyTemperature `的初始化。

### 2. Optional Property Types
如果你自定义的类型有一个存储属性在逻辑上允许“没有值”——或许是因为在初始化期间不能设置设置它的值，也可能是在后边的某个时间点允许它“没有值”，那么可以把这个属性声明为*optional type*。optional type的属性被自动初始化为`nil`。原文如下：

“If your custom type has a stored property that is logically allowed to have “no value”—perhaps because its value cannot be set during initialization, or because it is allowed to have “no value” at some later point—declare the property with an optional type. Properties of optional type are automatically initialized with a value of nil, indicating that the property is deliberately intended to have “no value yet” during initialization.”

Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3).” iBooks. 

看下下边这个例子：

~~~swift
class SurveyQuestion {
    var text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}

let cheeseQuestion = SurveyQuestion(text: "Do you like cheese")
cheeseQuestion.ask()
cheeseQuestion.response = "YES, i do like chesse."
~~~

### 3. Assigning Constant Properties During Initialization
你可以在初始化期间给一个常量属性赋一个值，只要在初始化完成时有一个明确的值就可以。

注意：对于类的实例，常量属性只能由引入该属性的类在初始化期间修改，不能被子类修改。

可以在上边那个类稍作修改，观察text属性：

~~~swift
class SurveyQuestion {
    let text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}

let cheeseQuestion = SurveyQuestion(text: "Do you like cheese")
cheeseQuestion.ask()
cheeseQuestion.response = "YES, i do like chesse."
~~~

## Default Initializers（默认构造函数）
在类或结构体为每个属性提供默认值并且没有其他构造函数时，Swift提供默认构造函数。默认构造函数简单的创建一个实例并把其所有属性设置成默认值。

~~~swift
class ShoppingList {
    var name: String?
    var quantity = 1
    var purchased = false
}

var item = ShoppingList()
~~~
上面的例子中，如果我们不给`quantity`设置默认值，编译器将提示`ShoppingList`类没有构造函数。
### Memberwise Initializers for Structure Types
结构体类型如果没有自定义的构造函数，那么会有一个*memberwise initializer*。

~~~swift
struct Size {
	var width = 0.0, height = 0.0
}
let twoByTwo = Size(width: 2.0, height: 2.0)
~~~

## Initializer Delegation for Value Types
Initializers can call other initializers to perform part of an instance's initialization. This process, known as *initializer delegation*, avoids duplicating code across multiple initializers.

对于值类型，可以使用`self.init`来引用该类型的其他构造函数。`self.init`只能在构造函数内使用。看个例子：

~~~swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}
struct Rect {
    var size = Size()
    var point = Point()
    init() {}
    init(size: Size, point: Point) {
        self.size = size
        self.point = point
    }
    init(size: Size, center: Point) {
        let originX = center.x - size.width / 2
        let originY = center.y - size.height / 2
        self.init(size: size, point: Point(x: originX, y: originY))
    }
}
~~~
上面例子中第一个构造函数其实就相当于默认构造函数，第二个相当于memberwise initializer，第三个是完全自定义的。这个例子还有一种写法，可以不写`init()`和`init(size:point:)`，需要用到extension。

如果为值类型自定义了一个构造函数，那就不能再访问默认构造函数了。如果你想要你定义的值类型既可以使用默认构造函数，也可以使用自定义的构造函数，那需要把自定义的构造函数写在extension中。

## Class Inheritance and Initialization
一个类所有的存储属性包括从父类继承的属性，都以徐在初始化几千有一个初始值。

Swift为类类型定义了两种类型的构造函数来确保存储属性有一个初始值，分别是*designated initializer*和*convenience initializer*。在下面的讨论中，我们分别把这两种称为指定构造函数、便利构造函数。

每个类都必须至少有一个指定构造函数。在一些条件下，这个需求通过从父类继承指定构造函数来满足。

便利构造函数可以调用该类中的指定构造函数。
### Syntax for Designated and Convenience Initializers
指定构造函数的语法：

![Designated Initalizer](./pic/designated_initializer_syntax.png)

便利构造函数的语法：

![Convenience Initializer](./pic/convenience_initializer_syntax.png)
### Initializer Delegation for Class Types
为了简化指定构造函数与遍历构造函数之间的关系，Swift在构造函数间的delegation call上使用下边三条规则：

1. A designated initializer must call a designated initializer from its immediate superclass.
2. A convenience initializer must call another initializer from the same class.
3. A convenience initiazlier must ultimately call a designated initializer.

可以简单的这样记：

* Designated initializers must always delegate up.
* Convenience initiazlizers must always delegate across.

这里有一个示意图来说明上面描述的规则：

![initializer_delegation_rules](./pic/initializer_delegation_rules.png)

### Two-Phase Initialization
在Swift中类的初始化分为两个阶段。在第一个阶段，类的每一个存储属性由引入其的类赋一个初始值，一旦每一个存储属性的初始状态确定了，第二个几段就开始了。第二个阶段开始以后，每个类在类可以使用前都有机会给每一个存储属性进行进一步自定义。

跟oc中的初始化有些许不同，主要体现在第一个阶段，在第一个阶段中，oc给每一个属性赋0值（如0、nil）。

Swift的编译器执行四步安全性检查(safaty-check)来确保两个阶段的初始化没有问题：

1. **A designated initializer must ensure that all of the properties introduced by its class are initizlied before it delegates up to a superclass initializer.**
2. **A designated initializer must delegate up to a superclass initializer before assigning a value to an inherited property.** If it doesn't, the new value the designated initializer assigns will be overwritten by the superclass as part of its own initialization.
3. **A convenience initializer must delegate to another initializer before assigning a value to any property.** If it doesn't, the new value the convenience initializer assigns will be overwritten by its own class's designated initializer.
4. **An initializer cannot call any instance methods, read the value of any instance properties, or refer to `self` as a value until after the first phase of initialization is complete.**

### Initializer Inheritance and Overriding
跟oc不同，Swift中的子类默认不继承父类的构造方法。当你为子类定义了一个构造方法，这个构造方法match一个父类的指定构造方法，你实际上是对那个指定构造方法进行了重写。因此，自己在子类构造方法前边用`override`标记。这个原则对于默认构造方法也适用。看个例子：

~~~swift
class Vehicle {
    var numberOfWheels = 0
    var description: String {
        return "\(numberOfWheels) wheel(s)"
    }
}

class Bicycle : Vehicle {
    override init() {
        super.init()
        numberOfWheels = 2
    }
}
~~~

### Automatic Initializer Inheritance
正如上边提到的，默认情况下，子类不继承弗雷德构造方法。但是，在满足一些条件的前提下，父类的构造方法可以被自动继承。

假设你已经为子类新引入的属性提供了默认值，那么有下面两条规则适用：

1. 如果子类没有定义任何指定构造方法，那么子类会自动继承父类所有的指定构造方法。
2. 如果子类提供了父类所有指定构造方法（不论是规则1提到的，还是通过自定义实现），那么子类自动继承父类所有的便利构造方法。**需要注意，子类可以把父类指定构造方法实现为便利构造方法，这样也可以满足这个条件**

这些规则即使在子类添加了便利构造方法时，依然适用。

### Designated and Convenience Initializer in Action
在这一部分，我们通过一个例子来解释上边的两条规则。

~~~swift
class Food {
    var name: String
    //指定构造方法
    init(name: String) {
        print("super versioin")
        self.name = name
    }
    //便利构造方法
    convenience init() {
        self.init(name: "[Unnamed]")
    }
}

class RecipeIngredient: Food {
    var quantity: Int
    init(name: String, quantity: Int) {
        self.quantity = quantity
        super.init(name: name)
    }
    override convenience init(name: String) {
        print("sub version")
        self.init(name: name, quantity: 1)
    }
}
~~~
现在这两个类的构造方法可以用下标的图来清楚的展示：

![RecipeIngredient](./pic/RecipeIngredient.png)

我们直接看一下`RecipeIngredient`类，首先有一个自定义的指定构造方法`init(name:quantity:)`，然后有一个自定义的便利构造方法`init(name:)`，这个方法实际上是重写了`Food`类的唯一的一个指定构造方法，所以满足了上面的第二条原则，因此自动继承`Food`类所有的便利构造方法，即`init()`方法。这里需要注意的是，`RecipeIngredient`继承来的`init()`方法中，调用的`init(name:)`不是`Food`版本的，而是`RecipeIngredient`版本的。

现在再来定义一个类：

~~~swift
class ShoppingListItem: RecipeIngredient {
    var purchased = false
    var description: String {
        var output = "\(quantity) * \(name)"
        output += purchased ? "✔️" : "❌"
        return output
    }
}
~~~
因为`ShoppingListItem`本身没有定义任何的构造方法，因此它从父类继承所有的构造方法。此时，示意图如下：

![ShoppingListItem](./pic/ShoppingListItem.png)
