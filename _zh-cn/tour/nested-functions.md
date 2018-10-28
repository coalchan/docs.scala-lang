---
layout: tour
title: 嵌套方法

discourse: false

partof: scala-tour
language: zh-cn
num: 8
next-page: multiple-parameter-lists
previous-page: higher-order-functions

redirect_from: "/tutorials/tour/nested-functions.html"
---

在Scala中，可以定义嵌套方法。下面的例子中，提供了一个方法`factorial`用于计算给定数字的阶乘：

```tut
 def factorial(x: Int): Int = {
    def fact(x: Int, accumulator: Int): Int = {
      if (x <= 1) accumulator
      else fact(x - 1, x * accumulator)
    }  
    fact(x, 1)
 }

 println("Factorial of 2: " + factorial(2))
 println("Factorial of 3: " + factorial(3))
```

该程序的输出是：

```
Factorial of 2: 2
Factorial of 3: 6
```
