# Basic functional constructs

One of the most basic features of any functional language are lambdas, i.e. anonymous functions. It's a no surprise that Scala has them too.

## Function objects

Scala represents functions as objects. There is a set of *traits* (something similar to Java interfaces), each one representing a function of some shape. They can be implemented in Scala just like any other trait, but this generally looks similar to anonymous classes in Java and introduces a lot of boilerplate. Instead, there is a short lambda syntax to represent anonymous function objects. 

## `Function0`

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

Thanks to that, we can now call our function:

```scala
doSomething.apply()
// or shorter
doSomething()
```

Note that this may look like a call to `doSomething` method but it's actually a call of `apply` method on `doSomething` object.

## `Function1`

`Function1` trait represents the most commonly used function type - a function that takes a single argument. The full syntax to define anonymous `Function1` is as follows:

```scala
val fun = (x: String) => x.toInt
```

This defines a function which takes a single string argument and parses it into an `Int`. Just like with `Function0`, we're relying on type inference, but we may be explicit about the type:

```scala
val fun: String => Int = 
  (x: String) => x.toInt
```

`String => Int` denotes a type of function from `String` to `Int`. This is equivalent to `Function1[String,Int]`.

Now that we explicitly declared the type of our function, we may omit the type declaration in lambda parameter:

```scala
val fun: String => Int = 
  x => x.toInt
```

In this case, the body of anonymous functions is simple enough so that we can shorten it even more:

```scala
val fun: String => Int = _.toInt
```

Also, we could have used the underscore syntax even without the the explicit type declaration, but this time we need to declare the type on the underscore itself:

```scala
val fun = (_: String).toInt
```

As you can see, Scala has multitude of different syntactic flavors for lambdas. 

*TODO write more about underscore syntax limitations*

## `Function2` and higher *TODO*

* higher order functions
* fold and multiple param lists
* by-name arguments
* interop with Java 8
