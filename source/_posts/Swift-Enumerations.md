---
title: 'Swift: Enumerations'
date: 2016-11-19 21:30:32
tags: Swift
---
Enumerations in Swift are flexible, and do not have to provide a value for each case of the enumeration. If a value(known as "raw" value) if provied for each enumeration case, the value can be a string, a character, or a value of any integer for floating-point type.  

<!-- more -->

## 枚举的语法
~~~swift
enum CompassPoint {
	case north
	case south
	case east
	case west
}
~~~
多个case可以写在一行，用逗号分隔  

~~~swift
enum Planet {
	case mercuy, venus, earth, mars, jupiter, saturn, uranus, neptune
}
~~~
简单使用

~~~swift
var directionToHead = CompassPoint.west

directionToHead = .east
~~~
## 关联值(Associated Values)
~~~swift
enum Barcode {
	case upc(Int, Int, Int, Int)
	case qrCode(String)
}

var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOPQRSTUVWXYZ")
~~~
提取关联的值:  

~~~swift
switch productBarcode {
case .upc(let numberSystem, let manufacturer, var product, let check):
    break
case .qrCode(let productCode):
    break
}
~~~
如果关联的值都要提取为常量或者都要提取为变量，可以这样写:

~~~swift
switch productBarcode {
case let .upc(numberSytem, manufacturer, product, check):
    break
case let .qrCode(productCode):
    break
}
~~~
## Raw Values
~~~swift
//raw value可以是字符串、字符、任何整型、浮点型
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}
~~~
### Implicitly Assigned Raw Values
当你使用枚举存储整型或者字符串类型的raw value时，没有必要给每一个case赋值。整型从0开始，字符串的raw value是case的名字。

~~~swift
enum RefinementPlanet: Int {
    case mercury = 1, vanus, earth, mars, jupiter, saturn, uranus, neptune
}

enum RefinementCompassPoint: String {
    case north, south, east, west
}
~~~
~~~swift
let earthsOrder = Planet.earth.rawValue
// earthsOrder is 3
 
let sunsetDirection = CompassPoint.west.rawValue
// sunsetDirection is "west"
~~~
### Initializing from a Raw Value
If you define an enumeration with a raw-value type, the enumeration automatically receives an initializer that takes a value of the raw value’s type (as a parameter called rawValue) and returns either an enumeration case or nil.

~~~swift
let possiblePlanet = RefinementPlanet(rawValue: 7)
//possiblePlanet的类型是 RefinementPlanet?
~~~
## Recursive Enumerations
递归枚举是一个枚举的一个或者多个case使用另一个枚举的实例作为关联值。通过的case前写`indirect`来表示这个case是递归的。

~~~swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}
~~~