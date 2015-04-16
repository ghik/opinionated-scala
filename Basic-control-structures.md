## Basic control structures

### Local variables

Scala has two keywords to define local variables: `val` and `var`. The `val` keyword defines a *value* - a "variable" that cannot change its value after initialization. The `var` keyword denotes a mutable variable.

```scala
val name: String = "Fred"
var counter: Int = 0
```

As you can see, the type of variable is declared after its name and separated from it with a colon `:`, similarly to how method return type is declared. This syntax of "typing" things with a colon is used in many more contexts and is generally called *type ascription*.

After the type comes the `=` sign and an initial value. The value can be arbitrary expression. From the previous section about Hello World we know that in Scala, even a block is an expression and can be assigned to a variable. This comes in handy when we need to perform some more complex computations to obtain the value, e.g.

```scala
//TODO better example
val adjustedString: String = {
  val str = fetchSomeString()
  str.substring(0, str.length-1).toUpperCase
}
```

#### Type inference

Type inference also works for local variables. We could omit type declarations and write our original example as:

```scala
val name = "Fred"
var counter = 0
```

However, it is recommended that for mutable variables (`var`s), the type should always be explicit. This is because the value of a `var` may change and therefore should not be inferred just from the initial value. The type of initial value may be narrower than what we intended. So, the recommended version of our example would be:

```scala
val name = "Fred"
var counter: Int = 0
```

#### Mutable variables

Scala encourages programming style that leverages immutability. Therefore, mutable variables should be avoided as much as possible. As you progress in your learning of Scala, you will see that it has many interesting features which make it possible. Many situations which require mutable variables in Java can be completely avoided in Scala. Try to express as much as you can with `val`s.

### `if` expression

Scala's `if` is similar to Java:

```scala
val x = 5
if(x < 10) {
  println("<10")
} else if (x == 10) {
  println("=10")
} else {
  println(">10")
}
```

However, there is similar difference between `if`s in Java and `if`s in Scala as there is for blocks. In Java, `if-else` is purely imperative structure, whereas in Scala it is a valid expression that can be assigned to variables or passed as arguments. Therefore, the above example can be refactored to:

```scala
val x = 5
println(
  if(x < 10) {
    "<10"
  } else if(x == 10) {
    "=10"
  } else {
    ">10"
  }
)
```

This can be further simplified - we don't need to wrap bodies of `if` and `else` into a block. They can also be arbitrary expressions:

```scala
println(if(x < 10) "<10" else if(x == 10) "=10" else ">10")
```

Scala's `if` is *always* an expression - even if the `else` clause is missing. If so, then what will the following code print?

```scala
val x = 15
println(if(x < 10) "<10")
```

It turns out that this code prints `()` (the "unit"). This is because Scala always implicitly adds a missing `else` clause and fills it with the `()`. The example above is actually translated to:

```scala
val x = 15
println(if(x<10) "<10" else ())
```

This can be *very* tricky, especially if you forget the `else` clause when assigning to a variable:

```scala
val x = 15
val str = if(x < 10) "<10"
```

Scala compiler will *not* issue an error here. Instead, it will add the implicit `else ()` clause and infer the type of `str` as `Any`, which is the most common type of `Int` from the `if` clause and `Unit` from the `else` clause. This can later cause compilation errors which are very hard to understand if you are not familiar with this peculiar Scala rule. Rule of thumb is that if you're getting strange compilation errors involving type `Any`, see if you haven't forgotten an `else` clause somewhere.

It is also worth to note that since Scala's `if` is an expression, there is no longer a need for the ternary conditional operator known from C and Java (`?:`). Scala doesn't have it.

### Equality comparisons

In Java, the `==` and `!=` operators perform value comparison on primitive types, but they do a reference comparison for objects. If you want to perform proper equality comparison on objects, you need to use the `equals` method.

In Scala, this is different. The `==` and `!=` operators *always* perform value equality. For objects, they internally call the `equals` method. They also handle situations where any of the operands is `null`, so you don't have to put any nullguards. For example, you can now safely write:

```scala
val str: String = fetchSomeStringThatMayBeNull()
if(str == "something") { ... }
```

If you really need to use reference comparison in Scala, you can still do it with the `eq` and `ne` operators.

### Lazy vals

Scala has one more flavor of local variables, the `lazy val`. It is equivalent to `val` except that its value is computed lazily, upon first reference to the `lazy val`. Lazy local values can help us make our code cleaner. Consider the following example:

```scala
def main(args: Array[String]) = {
  //TODO example
}
```

