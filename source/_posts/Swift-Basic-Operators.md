---
title: 'Swift: Basic Operators'
date: 2016-11-09 12:09:51
tags: Swift
---
*这里只讲一下Swift中比较特殊的操作符，在其他语言中也存在操作符就不再讲了*  
## Nil-Coalescing Operator: ??
The nil-coalescing operator (`a ?? b`) unwraps an optional a if it contains a value, or returns default value b if a is nil.  
这里的表达式`a`必须是optional type，表达式`b`的类型必须和存储在a中的值的类型相同。实际上这个运算符是下边这行代码的简写  

~~~swift
a != nil ? a! : b
~~~
**这个操作符跟`&&`类似，都短路计算（short-circuit evaluation）特性。**
## Range Operator
Swift包含两个范围操作符
### Closed Range Operator: a...b
`a...b`定义了一个从a到b的范围（包含a和b），a的值不能比b大。  

~~~swift
for index in 1...5 {
	print("\(index) times 5 is \(index * 5)")
}
~~~
### Half-Open Range Operator
`a..<b`定义一个从a到b的范围（不包括b），a的值不能比b大。  

~~~swift
let names = ["Anna", "Alex", "Brian", "Jack"]
let count = names.count
for i in 0..<count {
    print("Person \(i + 1) is called \(names[i])")
}
~~~