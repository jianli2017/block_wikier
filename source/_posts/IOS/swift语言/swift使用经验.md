---
title: swift 使用经验
date: 2019-07-17 09:07:12
tags: 
categories: swift
toc: true
---

列举swift使用经验

<!--more-->

## Swift 数组Array作为函数参数时如何在函数内部改变其值？

Swift中我们可以在参数类型的前面加上一个 inout 关键字，并在调用函数时在参数前加个取地址符 &，如下所示：

```
func doSomething(numArr: inout [String]){
    numArr.append("4")
}
var numbers = ["1","2","3"]
doSomething(numArr: &numbers)
print(numbers)
```

> 注意，inout 修饰参数时采用的是拷入拷出模式，即在函数内部使用的是参数的copy，函数结束后，又对参数重新赋值。
> 
> 由此，可以延伸一下，当一个类的属性被设置了 willSet 和 didSet 观察器时，如果该属性被作为函数参数，同时被 inout 修饰，那么当调用此函数时，会触发 willSet 和 didSet 观察器。

## for循环倒序

```
for i in (0...10).reversed() {
    print(i)
}

for i in stride(from:3,through:0,by: -1) {
    print(i)
}
```

Swift的stride函数返回一个任意可变步长类型值的序列。可变步长类型是可以设置偏移量的一维标量。他有两个变种:

1. from，to，最后一个值将会严格小(大)于to的值stride(from:3, to:0, by:-1) 表示3，2，1
2. from，through，最后一个值将会小(大)于等于through的值stride(from:3, through:0, by:-1) 表示3，2，1，0

