# Basic functional constructs

One of the most basic features of any functional language are lambdas, i.e. anonymous functions. It's a no surprise that Scala has them too.

## Function objects

Scala is a purely object oriented language in the sense that every value is an object. This includes functions. There is a set of *traits* (something similar to Java interfaces), each one representing a function of some shape/arity. They can be implemented in Scala just like any other trait, but this generally looks similar to anonymous classes in Java and introduces a lot of boilerplate. Instead, there is a short lambda syntax to represent anonymous function objects. 

### `Function0`

The most primitive type of a function is a function that takes no arguments. Anonymous no-argument function can be declared in Scala with following syntax:

```scala
val doSomething = () => println("stuff")
```

We're relying on type inference to deduce the type of `doSomething`, but we could make it explicit:

```scala
// we have split it into two lines to make it more readable
val doSomething: () => Unit = 
  () => println("stuff")
```

`() => T` is special scala syntax to denote types of no-argument functions. It is equivalent to `Function0[T]`. If we look into the `Function0` trait, we'll see that it has the `apply` method:

```scala
def apply(): R
```

As we know previous section, `apply` is a magic method which here allows us to invoke our function like this:

```scala
doSomething()
```

We're using the magic of `apply` to make the invocation look like we're calling `doSomething` method even though it's actually a call of `apply` method on `doSomething` object.

#### Variance

`Function0` has a nice property - it is *covariant*. This means that when `B` is a subtype of `A` then `() => B` is a subtype of `() => A`. Thanks to covariance, we can write:

```scala
val produceString: () => String =
  () => "someString"

val produceAnything: () => Any =
  produceString
```

Well, a function that produces a string is definitely a function that produces anything. This may seem like something very natural and obvious - how could this even work differently and why do we praise it so much? Well, try assigning a `Supplier<String>` to `Supplier<Object>` in Java - no luck. Variance (or strictly speaking - *declaration site* variance) is a surprisingly complex feature even though it seems obvious in usage site. We'll talk more about it in some other chapter.

### `Function1`

`Function1` trait represents the most commonly used function type - a function that takes a single argument. The full syntax to define anonymous `Function1` is as follows:

```scala
val fun = (x: String) => x.toInt
```

This defines a function which takes a single string argument and parses it into an `Int`. Just like with `Function0`, we're relying on type inference, but we may be explicit about the type:

```scala
val fun: String => Int = 
  (x: String) => x.toInt
```

`String => Int` denotes a type of function from `String` to `Int`. This is equivalent to `Function1[String,Int]`. It's also important to remember that the `=>` symbol is right-associative. This means `A => B => C` means "a function from `A` that returns a function from `B` to `C`", i.e. `A => (B => C)`.

Now that we explicitly declared the type of our function, we may omit the type declaration in lambda parameter:

```scala
val parseInt: String => Int = 
  x => x.toInt
```

In this case, the body of anonymous functions is simple enough so that we can shorten it even more:

```scala
val parseInt: String => Int = _.toInt
```

Also, we could have used the underscore syntax even without the the explicit type declaration, but this time we need to declare the type on the underscore itself:

```scala
val parseInt = (_: String).toInt
```

As you can see, Scala has multitude of different syntactic flavors for lambdas.

#### `Function1` methods

Now, let's look what we can do with our function. Here's what the `Function1` trait exposes:

```scala
// assume function of type B => C
def apply(b: B): C
def compose[A](g: A => B): A => C
def andThen[D](g: C => D): B => D
```

Of course, there's a magic `apply` method which allows us to call the function just like it was a method:

```
parseInt("123")
```

But there are also two other methods - `compose` and `andThen`. These two allow us to combine two functions into one. For example:

```scala
val parseAndDivide = parseInt andThen (_ / 2)
val parseAndMultiply = ((_: Int) * 2) compose parseInt
```

`compose` and `andThen` do the same thing - `compose` is equivalent to `andThen` with arguments flipped.

#### Variance

Just like `Function0`, `Function1` is covariant in its return type. If `B` is a subtype of `A` then function which returns `B` is a valid (subtype of) function that returns `A`.

But there's something more going on here. `Function1` is also *contravariant* in its argument type, which means that a function which takes `A` as its argument is a (valid subtype) of function that takes `B` as its argument (note that this is a "reversed" covariance). For example:

```scala
val consumeAnything: Any => Unit = 
  println(_)

val consumeString: String => Unit =
  consumeAnything
```

This is also very natural - a function that consumes anything can is definitely a function that can consume a string.

Covariance and contravariance can even work together:

```scala
val consumeAnythingAndProduceString: Any => String =
  _.toString

val consumeIntAndProduceCharSequence: Int => CharSequence =
  consumeAnythingAndProduceString
```

Don't worry if you don't understand how exactly variance works. It can become very complex sometimes, but you definitely don't need to think about it when working with functions. We're mentioning it here only because it's an important improvement over Java. As noted earlier, we'll get back to variance in a separate chapter.

### `Function2`

Functions that take two arguments have similar syntax:

```scala
// Scala introduces a `*` operator on strings which repeats a string given number of times
val repeatUpper = (s: String, n: Int) = s.toUpperCase * n
```

Again, we may omit type declarations when explicitly stating the type of function:

```scala
val repeatUpper: (String, Int) => String = 
  (s, n) => s.toUpperCase * n
```

And again, this function is simple enough so that we can use underscore syntax:

```scala
val repeatUpper: (String, Int) => String = 
  _.toUpperCase * _
```

Note that when using underscore syntax, each underscore represents different parameter.

#### `Function2` methods

Let's see what can we do with our two-argument function:

```scala
// assuming function (A, B) => C
def apply(a: A, b: B): C
def curried: A => B => C
def tupled: ((A, B)) => C
```

The magic `apply` method doesn't need explanation at this point. But we also have two strange converters: `curried` and `tupled`.

`curried` transforms our two-argument function into a function that takes one argument and returns a function that takes the other argument and then returns final result. So, instead of taking both parameters at once, curried version takes them one by one, producing intermediate, *partially applied* function.

`tupled` transforms our two-argument function into a single-argument function where the argument is a pair that contains our two original arguments.

The difference between regular function and its curried and tupled version is mostly syntactical. You use them differently when calling them and in different situations one may be more convenient than the other. But other than that, all three variants are doing the same computation inside.

#### Variance

Variance in `Function2` works the same way as in `Function1`, except that `Function2` is contravariant in both its parameter types.

### `Function3` and beyond

Scala defines function traits up to `Function22`. They all work pretty much the same way as `Function2`. The limit of 22 parameters is somewhat arbitrary and is caused by some kind of limitation of the JVM. Anyway, who would want to declare a function with over 20 parameters?

### More about underscore syntax

In examples of `Function1` and `Function2`, we used underscore syntax to define anonymous functions:

```scala
val parseInt: String => Int = _.toInt
val repeatUpper: (String, Int) => String = _.toUpperCase * _
```

This is the most concise syntax to define anonymous functions, but it has limitations. When we showed these examples, we said that we can use underscore syntax only because our functions are "simple" enough. But what does that mean, exactly?

Underscore syntax cannot be used in following situations:
* When we refer to function argument more than once in function body:

    ```scala
    val squared: Int => Int = x => x * x
    val squared2: Int => Int = _ * _ // error!
    ```

    We can't do it because every underscore in function body represents separate argument. Scala compiler understands `_ * _` as `(x, y) => x * y`

* When reference to argument is more deeply nested in function body:

    ```scala
    val printUpper: String => Unit = s => println(s.toUpperCase)
    val printUpper2: String => Unit = println(_.toUpperCase) // error!
    ````

    This time it doesn't work because the compiler understands `println(_.toUpperCase)` as `println(s => s.toUpperCase)`. It thinks that we're passing a function to `println`.

### Turning methods into functions

Very often we need a function that does nothing more than passing its arguments to some method:

```scala
def addModulo10(x: Int, y: Int) = (x + y) % 10
// the `reduce` method applies two-argument function consequently on all 
// elements of the vector until it has single value left
val sumModulo10 = Vector(1,2,3).reduce((x,y) => addModulo10(x,y))
```

In such situations, if the type of the function is clear from the context, then we can simply write name of the method where the function is expected:

```scala
val sumModulo10 = Vector(1,2,3).reduce(addModulo10)
```

Scala's automatic conversion of methods into functions is a transformation called *eta expansion*.

However, sometimes the compiler doesn't have enough information to do this automatically. For example, in the snippet below we don't declare type of `val` and the compiler doesn't know that it has to treat the method as function:

```scala
def addModulo10(x: Int, y: Int) = (x + y) % 10
val reductor = addModulo10 // error!
```

We have a few options to fix this:
```scala
val reductor1: (Int, Int) => Int = addModulo10
val reductor2 = addModulo10(_, _)
val reductor3 = addModulo10 _
```

We already know the first two methods. But the third has some new syntax - method name with a space and underscore after it. That's simply another way to force the compiler to treat the method as function - a slightly shorter version of the lambda syntax in second option.

## Higher order functions
