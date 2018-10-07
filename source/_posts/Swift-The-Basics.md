---
title: 'Swift: The Basics'
date: 2016-11-09 11:42:15
tags: Swift
---
1. Swift是类型安全的语言；
2. Swift introduces optional types, which handle the absence of a value. Optional say either "there is a value, and it equals x" or "there isn't a value at all".
3. 类型注解

	~~~swift
	var welcomeMessage: String
	var red, green, double: Double
	~~~

4. 类型别名（Type Alias）

	~~~swift
	typealias AudioSample = UInt16
	var maxAmplitudeFound = AudioSample.min
	~~~

5. 元组（Tuples）：元组内的值可以是不同类型的任何值。

	~~~swift
	let http404Error = (404, "Not Found")
	//http404Error的类型是(Int, String)
	~~~

	分解（decompose）元组：可以使用下划线来忽略一些值

	~~~swift
	let (statusCode, _) = http404Error
	print("The status code is \(status)")
	~~~

	也可以使用下标来取出单个值:

	~~~swift
 	print("The status code is \(http404Error.0)")
 	// Prints "The status code is 404"
 	print("The status message is \(http404Error.1)")
 	// Prints "The status message is Not Found"
 	~~~

	可以在定义元组的时候，给每个元素命名：

	~~~swift
	let http200Status = (statusCode: 200, description: "OK")
	~~~

	这个时候，可以这样获取每个元素的值：

	~~~swift
	print("The status code is \(http200Status.statusCode)")
	~~~

6. Optional type
	
	~~~swift
	let possibleNumber = "123"
	let convertedNumber = Int(possibleNumber)
	//convertedNumber的类型是 Int?， 不是Int
	~~~

	Swift的nil跟oc中的nil不同。In Objective-C, nil is a pointer to a nonexistent object. In Swift, nil is not a pointer -- it's the absence of a value of certain type. Optionals of any type can be set to nil, not just object types.
	
	forced unwrapping
	optional binding
	implicity unwrapped optionals