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

Scala has one more flavor of local variables, the `lazy val`. It is equivalent to `val` except that its value is computed lazily, upon first reference to the `lazy val`. Lazy local values can help us make our code cleaner by making small refactorings easier. Consider the following example:

```scala
def processArgs(args: Array[String], start: String, end: String) = {
  // args(0) refers to the first element of args
  if(args.nonEmpty && args(0).toLowerCase.startsWith(start) && args(0).toLowerCase.endsWith(end)) {
    // do something
  }
}
```

The code above is a bit ugly. The `if` condition has duplicated code - we refer to `args(0).toLowerCase` twice. This is bad for two reasons:
* it violates the DRY principle and makes code verbose
* it may be a performance problem - we evaluate the same expression twice

The natural solution would be to extract the duplicated expression to a variable, like so:

```scala
val arg = args(0).toLowerCase
if(args.nonEmpty && arg.startsWith(start) && arg.endsWith(end)) {
  // do something
}
```

But we have a problem - if `args` is empty, this will fail with an `ArrayIndexOutOfBoundsException`, because `arg` is assigned before we check for non-emptiness.

In order to fix this, we can simply turn the `val` into a `lazy val`:

```scala
lazy val arg = args(0).toLowerCase
if(args.nonEmpty && arg.startsWith(start) && arg.endsWith(end)) {
  // do something
}
```

This way `args(0).toLowerCase` will not be evaluated until the non-empty check is positive. It is also guaranteed that it will be evaluated at most once.

### Local methods

In Scala it is possible to define methods locally - any block of code can contain method definitions. They are visible only inside that block. Let's take the example from `lazy val` description and modify it as follows:

```scala
def processArguments(args: Array[String], start: String, end: String) = {
  def checkArg(arg: String) = 
    arg.startsWith(start) && arg.endsWith(end)
  if(args.nonEmpty && checkArg(args(0).toLowerCase)) {
    // do something
  }
}
```

Local methods are better than plain private methods for following reasons:
* Local method can refer to all the values visible at the point where it's defined. For example, our `checkArg` method can access the `start` and `end` parameters of its outer method. If it were a plain private method, it would have to accept them as its own parameters.
* Local method is visible only where it's actually needed. We do not pollute other namespaces. This also makes it easier to read the code.

Local methods can greatly improve readability. They allow you to give local but meaningful names to small pieces of your code while still keeping it concise.

### Loops

Scala has `while` and `do-while` loops:

```scala
var i = 0

while(i < 100) {
  println(i)
  i += 1
}

do {
  println(i)
  i -= 1
} while(i >= 0)
```

Again, the difference from Java is that both `while` and `do-while` are expressions. They are not very interesting though, because they always evaluate to `()`. These loops are very rarely used in Scala, even than in Java - usually only in some low-level or performance critical code.

In the above example, you may have also noticed that we incremented and decremented our local variable using `+=` and `-=`. Scala does not have the C-style prefix and postfix operators `++` and `--`.

Technically, Scala doesn't have a `for` loop. Instead, it has a more general construct called _for comprehension_, which can be used as a foreach-style loop. We will not cover the entire _for comprehension_ syntax here, but only show how to use it like it's a loop. Example:

```scala
val args: Array[String] = fetchArgs()
for(arg <- args) {
  println(arg)
}
// (x to y) creates an object which represents an integer range from x to y, inclusively
for(i <- (0 to args.length-1)) {
  println(args(i))
}
```

The `for` "loop" above is actually just a syntactic sugar for calling the `foreach` method which takes some action and invokes it for every element. The above example is equivalent to:

```scala
val args: Array[String] = fetchArgs()
args.foreach(arg => println(arg))
(0 to args.length-1).foreach(arg => println(arg))
```

The first loop can be even shorter: `args.foreach(println)`. We are using lambdas and higher-order functions here. We will cover them in more detail later.
Loops written using `for` comprehensions are somewhat less performant than `while` and `do-while` loops due to usage of lambdas, whose body must be compiled to a separate anonymous class.
Loops in Scala can be avoided much more than in Java thanks to various higher-order functions available on collections and usage of tail recursion. We will cover these topics in some other chapter.
