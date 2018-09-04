---
layout: tour
title: 型变

discourse: false

partof: scala-tour

num: 17

language: zh-cn

next-page: upper-type-bounds
previous-page: generic-classes
---

型变是复杂类型的子类型关系与它们组件的类型的子类型关系之间的关联性。Scala支持 [泛型类](generic-classes.html)的类型参数上的型变注解，可以使得它们是协变的、逆变的或者不变的（如果没有使用注解的话）。型变在类型系统中的使用使得我们在复杂的类型之间建立起了直观的联系，反之如果没有型变则限制了抽象类的重用。

```tut
class Foo[+A] // 协变类
class Bar[-A] // 逆变类
class Baz[A]  // 不变类
```

### 协变

泛型类的类型参数`A`可以通过使用注解`+A`变成协变的。对于某个`class List[+A]`，让`A`型变意味着对于类型`A`和`B`，如果`A`是`B`的子类型，那么`List[A]`则是`List[B]`的子类型。这让我们可以使用泛型类来创建非常有用且直观的子类型关系。

现在来看下面这个简单的类结构：

```tut
abstract class Animal {
  def name: String
}
case class Cat(name: String) extends Animal
case class Dog(name: String) extends Animal
```

`Cat`和`Dog`都是`Animal`的子类型。Scala的标准库中有一个常用的不可变类`sealed abstract class List[+A]` ，它的类型参数`A`就是协变的。这意味着`List[Cat]`是一个`List[Animal]`而 `List[Dog]`也是一个`List[Animal]`。从直观上，一个猫的列表和一个狗的列表都是一个动物列表是讲得通的。因而你可以把它们其中任意一个替换为`List[Animal]`来使用。

下面的例子当中，方法`printAnimalNames`会接受一个动物列表作为参数，并且依次在新的一行里打印它们的名字。如果`List[A]`不是协变的，则下面两个方法的调用不会通过编译，这将严重限制了`printAnimalNames`方法的使用。

```tut
object CovarianceTest extends App {
  def printAnimalNames(animals: List[Animal]): Unit = {
    animals.foreach { animal =>
      println(animal.name)
    }
  }

  val cats: List[Cat] = List(Cat("Whiskers"), Cat("Tom"))
  val dogs: List[Dog] = List(Dog("Fido"), Dog("Rex"))

  printAnimalNames(cats)
  // Whiskers
  // Tom

  printAnimalNames(dogs)
  // Fido
  // Rex
}
```

### 逆变

A type parameter `A` of a generic class can be made contravariant by using the annotation `-A`. This creates a subtyping relationship between the class and its type parameter that is similar, but opposite to what we get with covariance. That is, for some `class Writer[-A]`, making `A` contravariant implies that for two types `A` and `B` where `A` is a subtype of `B`, `Writer[B]` is a subtype of `Writer[A]`.

Consider the `Cat`, `Dog`, and `Animal` classes defined above for the following example:

```tut
abstract class Printer[-A] {
  def print(value: A): Unit
}
```

A `Printer[A]` is a simple class that knows how to print out some type `A`. Let's define some subclasses for specific types:

```tut
class AnimalPrinter extends Printer[Animal] {
  def print(animal: Animal): Unit =
    println("The animal's name is: " + animal.name)
}

class CatPrinter extends Printer[Cat] {
  def print(cat: Cat): Unit =
    println("The cat's name is: " + cat.name)
}
```

If a `Printer[Cat]` knows how to print any `Cat` to the console, and a `Printer[Animal]` knows how to print any `Animal` to the console, it makes sense that a `Printer[Animal]` would also know how to print any `Cat`. The inverse relationship does not apply, because a `Printer[Cat]` does not know how to print any `Animal` to the console. Therefore, we should be able to substitute a `Printer[Animal]` for a `Printer[Cat]`, if we wish, and making `Printer[A]` contravariant allows us to do exactly that.

```tut
object ContravarianceTest extends App {
  val myCat: Cat = Cat("Boots")

  def printMyCat(printer: Printer[Cat]): Unit = {
    printer.print(myCat)
  }

  val catPrinter: Printer[Cat] = new CatPrinter
  val animalPrinter: Printer[Animal] = new AnimalPrinter

  printMyCat(catPrinter)
  printMyCat(animalPrinter)
}
```

The output of this program will be:

```
The cat's name is: Boots
The animal's name is: Boots
```

### Invariance

Generic classes in Scala are invariant by default. This means that they are neither covariant nor contravariant. In the context of the following example, `Container` class is invariant. A `Container[Cat]` is _not_ a `Container[Animal]`, nor is the reverse true.

```tut
class Container[A](value: A) {
  private var _value: A = value
  def getValue: A = _value
  def setValue(value: A): Unit = {
    _value = value
  }
}
```

It may seem like a `Container[Cat]` should naturally also be a `Container[Animal]`, but allowing a mutable generic class to be covariant would not be safe.  In this example, it is very important that `Container` is invariant. Supposing `Container` was actually covariant, something like this could happen:

```
val catContainer: Container[Cat] = new Container(Cat("Felix"))
val animalContainer: Container[Animal] = catContainer
animalContainer.setValue(Dog("Spot"))
val cat: Cat = catContainer.getValue // Oops, we'd end up with a Dog assigned to a Cat
```

Fortunately, the compiler stops us long before we could get this far.

### Other Examples

Another example that can help one understand variance is `trait Function1[-T, +R]` from the Scala standard library. `Function1` represents a function with one argument, where the first type parameter `T` represents the argument type, and the second type parameter `R` represents the return type. A `Function1` is contravariant over its argument type, and covariant over its return type. For this example we'll use the literal notation `A => B` to represent a `Function1[A, B]`.

Assume the similar `Cat`, `Dog`, `Animal` inheritance tree used earlier, plus the following:

```tut
abstract class SmallAnimal extends Animal
case class Mouse(name: String) extends SmallAnimal
```

Suppose we're working with functions that accept types of animals, and return the types of food they eat. If we would like a `Cat => SmallAnimal` (because cats eat small animals), but are given a `Animal => Mouse` instead, our program will still work. Intuitively an `Animal => Mouse` will still accept a `Cat` as an argument, because a `Cat` is an `Animal`, and it returns a `Mouse`, which is also a `SmallAnimal`. Since we can safely and invisibly substitute the former for the latter, we can say `Animal => Mouse` is a subtype of `Cat => SmallAnimal`.

### Comparison With Other Languages

Variance is supported in different ways by some languages that are similar to Scala. For example, variance annotations in Scala closely resemble those in C#, where the annotations are added when a class abstraction is defined (declaration-site variance). In Java, however, variance annotations are given by clients when a class abstraction is used (use-site variance).