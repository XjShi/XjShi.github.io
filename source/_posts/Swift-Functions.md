---
title: 'Swift: Functions'
date: 2016-11-09 20:39:29
tags: Swift
---
### 默认参数值(Default Parameter Values)
~~~swift
//默认参数值
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    
}

someFunction(parameterWithoutDefault: 10)
someFunction(parameterWithoutDefault: 10, parameterWithDefault: 23)
~~~
### 可变长参数(Variadic Parameters)
~~~swift
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1,2,3,4,5)
arithmeticMean(3, 8.25,13.23)
~~~
*一个函数最多有一个可边长参数*
### In-Out Parameters
* 函数参数默认是常量，如果你想要修改一个参数的值，并且让这个改变在函数调用结束后依然有效，就要把参数定义为in-out。  
* 语法：在参数类型前边加`inout`关键字。  
* 只能把变量传递给函数的inout参数，不能传递一个常量或者字面值，因为常量或者字面值不能够被修改。  
* 可以在变量名字前边放`&`符号，表示变量可以被函数改变。  

*in-out参数不能有默认值，可变参数不能标记为inout。*

~~~swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
~~~
