# [Scala Tutorial]((https://www.scala-exercises.org/scala_tutorial/)) #

## Terms and Types ##

```scala
scala> 1                  // Static typing: Int
val res0: Int = 1

scala> true               // Boolean
val res1: Boolean = true

scala> "Hello"            // String
val res2: String = Hello

scala> 'Hello'
       ^
       warning: symbol literal is deprecated; use Symbol("Hello") instead
             ^
       error: unclosed character literal (or use " for string literal "Hello")

scala> 1 + 2                     // compound expression
val res3: Int = 3

scala> 1.+(2)                    // infix operator as regular method using dot notation
val res12: Int = 3

scala> "Hello, " ++ "Scala"      // string concatenation
val res4: String = Hello, Scala

scala> (1 + 2) * 3
val res5: Int = 9

scala> "Hello".size         // method call without parameter
val res6: Int = 5

scala> "Hello".size()
                   ^
       error: Int does not take parameters

scala> "Hello".toLowerCase
val res11: String = hello

scala> "foo".drop(1)        // method call with parameters
val res21: String = oo

scala> "bar".take(2)
val res22: String = ba

scala> 1.to(10)
val res8: scala.collection.immutable.Range.Inclusive = Range 1 to 10

scala> 1 to 10                 // infix syntax
val res13: scala.collection.immutable.Range.Inclusive = Range 1 to 10

scala> (0 to 10).contains(10)
val res18: Boolean = true

scala> (0 until 10).contains(10)
val res20: Boolean = false

scala> 16.toHexString          // numerical literal is Int object
val res15: String = 10

scala> val x = 3
val x: Int = 3

scala> !x
       ^
       error: value unary_! is not a member of Int
       did you mean unary_+, unary_-, or unary_~?

scala> val x = true
val x: Boolean = true

scala> !x                   // boolean expression applies short-circuit evaluation
val res32: Boolean = false

scala> x && x
val res33: Boolean = true

scala> x || x
val res34: Boolean = true
```


## Definitions and Evaluation ##

```scala
scala> val a = 10     // define variable
val a: Int = 10

scala> val b = 30
val b: Int = 30

scala> a * b          // use variable
val res24: Int = 300

scala> def add(x: Double, y: Double) = x + y          // define method
def add(x: Double, y: Double): Double

scala> def add(x: Double, y: Double): Double = x + y  // explictly specify return method type
def add(x: Double, y: Double): Double

scala> add(a, b)                                      // call method
val res25: Double = 40.0

scala> val c = add(a, b)  // value of c is fixed when defined
val c: Double = 40.0

scala> val a = 3
val a: Int = 3

scala> add(a, b)
val res27: Double = 33.0

scala> c
val res26: Double = 40.0
```


## Functional Loops ##

```scala
scala> val x = 3
val x: Int = 3

scala> if (a > 0) x else -x  // conditional expression not statement
val res30: Int = 3
```


## Lexical Scopes ##

```scala
scala> def circlearea(r: Double) = {  // {} delimites a block expression
     |     def square(x: Double) =    // nested function
     |         x * x
     | 
     |     val pi = 3.14
     |     pi * square(r)             // the block's value is the last expression
     | }
def circlearea(r: Double): Double

scala> circlearea(2)
val res36: Double = 12.56

scala> val x = 5
val x: Int = 5

scala> val result = {
     |     val x = 3    // inner block definition shadow outer block one
     |     x * x        // x == 3
     | } + x            // x == 5.  inner block definition is only visible within the block
val result: Int = 14

scala> val x = 3; x * x  // use semicolon to put several expressions in a single line
val x: Int = 3
val res38: Int = 9
```

Scala programs are written in `.scala` files, and organized in
packages by putting the `.scala` file into the same physical directory
as package's qualified path name and using a package clause:

```scala
// file: foo/Bar.scala
package foo
object Bar {
    def func(x: Double) = x
}
```

```scala
// file: quux/Quux.scala
package quux
object Quux {
    def func(x: Double) = x * 2
}
```

```scala
// file: foo/Baz.scala
package foo
import quux.Quux
object Baz {
    val a = Bar.func(2)
    val b = Quux.func(3)
}
```

Scala executable application program should contain a `main` method:

```scala
object Hello {
    def main(args: Array[String]) = println("hello")
}
```
