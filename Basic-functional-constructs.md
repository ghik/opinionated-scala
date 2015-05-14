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

A higher order function (or method) is a function that takes another function as its argument.

Therefore, it's not a surprise that anonymous functions are mostly used when being passed to higher order methods. We've already used a few of these, e.g. `foreach` or `reduce`. In this section, we'll stay a bit more with them to discuss not only how to use them, but how to *define* them.

There is a great number of HOFs available in the Scala collection API. Here are some examples of most basic ones:

```scala
val nums = Vector(1,5,2,6,4,3,2,7).filter(_ < 5) // Vector(1,2,4,3,2)
val ints = Vector("123", "456").map(_.toInt)     // Vector(123,456)
Vector("abc", "hello").foreach(println)          // prints "abc" and "hello"
```

These are simple - they only take single argument (which is a function). However, when a HOF takes more than one argument, it tends to have a bit more peculiar signature. For example, let's look at the `foldLeft` method available in the Scala collection API:

```scala
// A is the type of elements in the collection
def foldLeft[B](z: B)(op: (B, A) => B): B 
```

It takes two arguments - a value `z` of arbitrary type `B` and a binary function which is able to combine a value of `B` with collection element to produce another value of `B`. At the beginning, the function is applied on `z` and first element of the collection. Then we have another value of type `B` ("next" `z`) which is combined with second element of the collection. This goes on until we apply our function on every collection element and produce final value of type `B`. That value is then returned as the result of entire `foldLeft` invocation.

Very simple example of `foldLeft` usage is to compute a sum of squares of every element in collection:

```scala
val sumOfSquares = Vector(1,2,3,4).foldLeft(0)((sumSoFar, el) => sumSoFar + el * el)
```
#### Multiple parameter lists

You may have already spotted that there's something strange about the signature of `foldLeft` - it takes two arguments, but each of them is passed in a separate pair of parentheses, i.e. each has a separate parameter list in method declaration.

First of all, it may be surprising that Scala even allows something like this. And there's even more than that - `foldLeft` has two parameter lists, but we can define a method which takes arbitrary number of parameter lists, each one with arbitrary number of parameters (zero in particular).

But why would we need something like this? There is a few good reasons for that:

##### Limitations of type inference
  
This is probably the most important reason and the primary purpose for multiple parameter lists in `foldLeft`. We'll try to outline what kind of limitations we have in mind. Let's try to define our own `foldLeft` method that takes its argument in a *single* parameter list. For simplicity, let's assume that it works on a collection that we have in a local variable:

```scala
val vec = Vector(1, 2, 3, 4)
def vecFoldLeft[B](z: B, f: (B, Int) => B): B = vec.foldLeft(z)(f) 

vecFoldLeft(0, (sumSoFar, el) => sumSoFar + el * el) // error!
```

Unfortunately, when we try to compile it, we get an error:

```
error: missing parameter type
vecFoldLeft(0, (sumSoFar, el) => sumSoFar + el * el)
                ^
```

We have passed an `Int` value (zero) as the `z` parameter, so it should be clear for the compiler that `B` == `Int` and the type of `sumSoFar` parameter is `Int`. But unfortunately, it is not. The compiler cannot infer the type of one argument based on type of other argument in the same argument list. That is simply a somewhat arbitrary limitation of type inference algorithm used in Scala.

Apparently, the problem disappears when we split the parameters into two separate parameter lists:

```scala
def vecFoldLeft[B](z: B)(f: (B, Int) => B): B = vec.foldLeft(z)(f) 
vecFoldLeft(0)((sumSoFar, el) => sumSoFar + el * el)
```

This is because Scala type inference processes every argument list one after another. While doing so, it is able to use information obtained from previous argument lists to infer types in subsequent argument lists. In the example above, when the first argument list is processed, the compiler determines that `B` == `Int`. Then it becomes clear for it that the type of `f` is `(Int,Int) => Int` and no longer requires explicit type declaration for `sumSoFar`.

Of course, we could also fix the problem by actually giving the `sumSoFar` parameter some explicit type. But it is better to force the inference to work how we want instead of giving it explicit information.

##### Nicer syntax with long functions

Taking out the function parameter into separate parameter list also has slight aesthetical advantage. If our function is long, we can pass it like this:

```scala
Vector(1,2,3).foldLeft(0) { (sumSoFar, el) =>
  // some long code here
  sumSoFar + el * el
}
```

instead of:

```scala
// let's ignore type inference problems for this example
Vector(1,2,3).foldLeft(0, (sumSoFar, el) => {
  // some long code here
  sumSoFar + el * el
})
```

That's minor syntactical advantage - we don't need to wrap our function into both curly braces and parentheses.

*Summary*: when declaring higher order methods which take more than one argument, it is good to place the function argument in a separate parameter list and put it at the end.

##### Slightly nicer partial application

When you expect your higher order method to be partially applied, it is good to move the parameters likely to be NOT supplied when partially applying into the separate parameter list and put it at the end. For example, the following snippet:

```scala
def takeManyArgs(i: Int, s: String, d: Double, vi: Vector[Int], ls: List[String]) = ???
val takeOnlySomeArgs = takeManyArgs(42, "stuff", _, _, _)
```

could become:

```scala
def takeManyArgs(i: Int, s: String)(d: Double, vi: Vector[Int], ls: List[String]) = ???
val takeOnlySomeArgs = takeManyArgs(42, "stuff") _
```

which may be a bit more concise.

Also, by splitting your parameters into separate lists, you kind-of "suggest" that your HOF is likely to be partially applied.

## By-name parameters

Normally, when you pass some expression as an argument to some method, the expression is evaluated eagerly and its result value is passed to that method, which only then is invoked. However, Scala allows to alter this behavior with the so-called *by-name parameters*.

Let's start with an example. Imagine we'd like to write our own implementation of `if-else` statement using a regular method. The method would have to take three parameters: the condition and two expressions - the first one evaluated when condition is true and the second one otherwise:

```scala
// the method is parameterized (generic) with arbitrary type T
def ifelse[T](condition: Boolean, whenTrue: T, whenFalse: T): T =
  if(condition) whenTrue else whenFalse
```

However, this implementation is not good. If we were to call:

```scala
val x: Int = fetchSomeInt()
ifelse(x < 5, println("x < 5"), println("x >= 5"))
```

the output would be:

```
x < 5
x >= 5
```

Both `x < 5` and `x >= 5` have been printed. This is because the `println` invocations have been evaluated *eagerly*, before the `ifelse` method was called. We need to find some way to defer the evaluation until the condition is checked.

#### Solution with `Function0`

Instead of passing eagerly evaluated values to our method, we could use no-argument functions:

```scala
def ifelse[T](condition: Boolean, whenTrue: () => T, whenFalse: () => T): T =
  if(condition) whenTrue() else whenFalse()
```

We need to wrap passed expressions into no-arg functions, too:

```scala
ifelse(x < 5, () => println("x < 5"), () => println("x >= 5"))
```

This will work as intended - the no-argument functions are going to be invoked only after checking the condition and it is guaranteed that only one of them will be invoked.

#### Solution with by-name params

By-name arguments work very similarly to no-argument functions, but have more concise syntax and are treated by the Scala compiler a bit differently. Using by-name arguments, our `ifelse` method could be implemented like this:

```scala
def ifelse[T](condition: Boolean, whenTrue: => T, whenFalse: => T): T =
  if(condition) whenTrue else whenFalse
```

The `=> T` part is a new syntax, previously not used in this guide. It denotes that the parameter is a *by-name* parameter. This is pretty much the same as if the parameter was a no-argument function, except that we don't need to wrap the passed expression explicitly into a function. We could pass the arguments as if it was a regular method:

```scala
ifelse(x < 5, println("x < 5"), println("x >= 5"))
```

Passed expressions will still be evaluated on-demand, after checking the condition.

#### Multiple evaluation

It is very important to remember that by-name arguments are re-evaluated **every single time** they are referred to. For example, the following code will print `"Hello"` twice:

```scala
def doTwice(code: => Any): Unit = {
  code
  code
}
doTwice(println("Hello"))
```

Therefore, by-name arguments cannot be treated simply as "lazy" arguments.

#### Use by-name params carefully

By-name arguments provide nicer syntax than no-argument functions, but that syntax can also be dangerous. When looking at method invocation, we can never be sure if its parameters are regular or by-name until we look at method signature. This can lead to very tricky errors. 

Someone could break the program simply by extracting to a local variable an expression passed as by-name argument, causing it to be evaluated eagerly. Things could also go bad when the by-name argument is evaluated more than once by its method or saved somewhere to be evaluated later. 

When defining methods with by-name arguments, we recommend to adhere to following rules:

* method name and purpose should make it clear that its arguments are (or may be) by-name arguments, so that this is apparent in the call site
* by-name arguments should not be evaluated more than once inside the method, unless its name and usage make it very clear that they are
* by-name arguments should not be passed for evaluation outside of its method, unless - again - method name and usage make it very clear that they are

In other words, you should design your API so that someone who reads code that uses it sees clearly that some arguments are passed as by-name.

If you have doubts whether it's safe to use by-name arguments, you can always fall back to no-argument functions.