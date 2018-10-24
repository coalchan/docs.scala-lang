---
layout: tour
title: Default Parameter Values

discourse: false

partof: scala-tour

num: 31

language: zh-cn

next-page: named-arguments
previous-page: annotations
---

Scala提供了参数默认值的功能，允许调用者忽略那些参数。

```tut
def log(message: String, level: String = "INFO") = println(s"$level: $message")

log("System starting")  // prints INFO: System starting
log("User not found", "WARNING")  // prints WARNING: User not found
```

参数`level`拥有一个默认值所以该参数是可选的。最后一行，参数`"WARNING"`覆盖了默认参数`"INFO"`。在Java中你可能需要进行重载方法，而在Scala中使用可选参数能够达到同样的效果。但是，如果调用者忽略了一个参数，那么后面的参数必须带上名字。

```tut  
class Point(val x: Double = 0, val y: Double = 0)

val point1 = new Point(y = 1)
```

这里一定要写上`y=1`。

注意，在Java代码中调用时，Scala中的默认参数并不是可选的：

```tut
// Point.scala
class Point(val x: Double = 0, val y: Double = 0)
```

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        Point point = new Point(1);  // does not compile
    }
}
```

