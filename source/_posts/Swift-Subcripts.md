---
title: 'Swift: Subcripts'
date: 2016-11-11 16:24:19
tags: Swift
---
类、结构体、枚举都可以定义下标（subscript），下标是访问集合、列表、序列的元素的快捷方式。

在Swift中可以为类型定义下标，而且不限于一维。

## 语法
下标定义的方法：跟实例方法的语法类似，`subscript`关键字，一个或多个输入参数，一个返回值。

下标可以是可读可写的，也可以是只可读的，由getter和setter决定，跟computed property类似。

~~~swift
subscript(index: Int) -> Int {
    get {
        // return an appropriate subscript value here
    }
    set(newValue) {
        // perform a suitable setting action here
    }
}
~~~
newValue的类型跟下标操作的返回值的类型相同，与计算属性一样，你可以不指定newValue这个参数，那么会默认提供一个newValue。

跟只读的计算属性一样，只读的下标也可以这样写：
~~~swift
subscript(index: Int) -> Int {
	// return a appropriate subscript value here
}
~~~

## Subscript Options
下标可以带任意数量的输入参数，这些参数可以是任何类型。下标也可以返回任何类型。**下标可以使用可变参数（variadic parameters），蚕食不能使用in-out参数，或者提供默认参数值。**
