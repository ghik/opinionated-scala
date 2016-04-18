*Implicits* are an important, unique and controversial Scala feature. They are a low-level language mechanism that underlies many hihger-level features and are responsible for much of Scala's expresiveness, conciseness and type safety.

Awesome, but what are they, actually?

## The `implicit` keyword

In Scala, every *term member* may be marked as `implicit`. This includes:
* local `val`s, `var`s, `lazy val`s, `def`s and `object`s
* `val`s, `var`s, `lazy val`s, `def`s and `object`s which are members of some class, object or trait

```scala
def main(args: Array[String]): Unit = {
  implicit val executionContext: ExecutionContext = ???
  implicit def conversion(s: String): ByteString = ByteString(s)
}
trait TypeInfo[T]
object TypeInfo {
  implicit object stringInfo extends TypeInfo[String]
}
```

When some symbol is marked as `implicit`, the compiler is allowed to use it "automatically" in some particular contexts. This means that the compiler may automatically insert a reference to an implicit value into the compiled code when it "needs" it.

## When does the compiler look for implicits?

### Implicit parameters

The first situation where the compiler may automagically use some implicit symbol is when a method or constructor invocation lacks *implicit parameters*.

What are implicit parameters? As you already know, every method and constructor in Scala can take more than one parameter list. Additionally, the last parameter list (and only the last) may be marked as `implicit`:

```scala
def doSomethingWith[T](value: T)(implicit ec: ExecutionContext, ct: ClassTag[T]) = ???
```

**NOTE** Although syntactically it looks like the `implicit` keyword applies to the `ec` parameter, it's not true - it applies to the entire parameter list, i.e. both `ec` and `ct` are *implicit parameters*.

`implicit` keyword applied to a parameter group has two effects:
* when invoking the method, implicit parameters *may be omitted* and the compiler will try to *find* appropriate implicit values and *automatically* pass them to the method.
* implicit parameters themselves are seen inside method body as implicit values (i.e. as if they were local `implicit val`s)

*Example* - perhaps the most commonly used API in Scala standard library that uses implicit parameters are methods on `Future`s. Most of the `Future` API consists of methods which register a listener that will consume the result of the `Future` when it's complete, e.g. `onComplete`, `foreach`, `map`, `flatMap`, `recover`. That listener cannot be executed just anywhere - it needs to be submitted to an `ExecutionContext` which usually represents some thread pool or task queue. Since we usually invoke many `Future` methods in a small code block, it makes sense to accept the `ExecutionContext` as implicit parameter, so that we don't need to manually pass it to every single invocation.

```scala
def getUserIdByName(name: String): Future[Long] = ???
def getUserAge(id: Long): Future[Int] = ???

val name = "Implisław"

import ExecutionContext.Implicits.global

getUserIdByName(name)
  .flatMap(id => getUserAge(id)
    .map(age => s"$name (ID $id) is $age years old")
  )
  .foreach(println)
```

Without the implicit parameter this would look like this:

```scala
getUserIdByName(name)
  .flatMap(id => getUserAge(id)(executionContext)
    .map(age => s"$name (ID $id) is $age years old")(executionContext)
  )
  .foreach(println)(executionContext)
```

The "implicitness" is especially important here, because the above example can be rewritten to use the for-comprehension syntax:

```scala
val name = "Implisław"
for {
  id <- getUserIdByName(name)
  age <- getUserAge(id)
} {
  println(s"$name (ID $id) is $age years old")
}
```

The way for-comprehension syntax is translated into actual method calls is hardwired into the Scala parser and can't be changed. For example, the above code is parsed as:

```scala
getUserIdByName(name).flatMap(id => getUserAge(id).foreach(age => println(s"$name (ID $id) is $age years old")))
```

The parser always generates `flatMap` and `foreach` calls which take exactly one parameter. But using implicit parameters, we can "smuggle" some additional dependencies, like `ExecutionContext` in this case.

### Implicit conversions

The second situation when the compiler may automatically use an implicit value is when some types in the code don't match (i.e. when we pass a value of wrong type to some method). In such case, the compiler will not immediately issue an error, but first it will try to find an *implicit conversion*.

An implicit conversion is any implicit value whose type is a `Function1`. For example:

```scala
implicit val stringToInt: String => Int = _.toInt
```

Here's an example of a situation when the compiler will automatically use the conversion:

```scala
implicit val stringToInt: String => Int = _.toInt
def takeInt(i: Int): Unit = ???
takeInt("123") // this will compile thanks to `stringToInt` being in scope
```

Passing parameters or returning a value are the most obvious situations where types may not match. What may be less apparent is that accessing a nonexistent member or member with wrong signature on some value is treated similarly by the compiler. For example:

```scala
"somestring".toByteString
```

Of course, there is no such method as `toByteString` on the `String` type. But the compiler will not immediately issue an error. Before that, it's going to try finding an implicit conversion from `String` to a type that actually has a member named `toByteString`. In other words, it will look for an implicit value of type `String => { def toByteString: Any }`. The `{ def toByteString: Any }` part is so called *structural type* - a type identified not by some class or trait name and parameters but by signatures of its members.

Implicit conversions used to extend APIs of existing types are usually called *implicit views* and methods that they provide are usually called *extension methods*.

## Where does the compiler look for implicits?

We've just said that the compiler automatically searches for values marked as implicit when it needs them and we specified well *when* it needs them. But where exactly does it look for implicits?

In examples above, we always defined implicit values so that they're visible in the local scope of code that requires them. We did that either by putting implicit definitions directly into the local scope or by importing them into local scope from somewhere.

The compiler indeed looks for implicits in the current scope. That means, it may use every implicit symbol which can be referred to just by its name, without any prefix. This includes local symbols, members of all enclosing classes plus their superclasses/supertraits. This also includes all symbols imported into current scope.

But if the compiler can't find an implicit with proper type, it doesn't stop there. Before failing, it looks into what's called *implicit scope* of the type that it's looking for.

The name "implicits" by itself doesn't mean anything and encompasses two independent but closely related features *implicit parameters* and *implicit conversions*. You'll also hear about *implicit classes* and *implicit views* which are just other names or special cases of implicit conversions.

### Implicit parameters

**TODO**

* last parameter list of a method or constructor may be marked as *implicit*, using the `implicit` keyword - this means that if we don't supply these parameters to a method call, the compiler will try to automatically find appropriate values
* only values marked by themselves as `implicit` are eligible to be passed as implicit parameters
* implicit `def`s that take parameters themselves are seen as implicit function values, unless the parameters of the `def` are themselves implicit - this would mean a chained implicit (an implicit which depends on other implicits)
* implicit parameters are themselves seen as implicit values inside the method body
* implicits are searched for in current scope or, when not found, in the *implicit scope* which consists of companion objects of classes and traits that occur in the type of the implicit, or their base classes and traits
* beware of shadowing!
* overloading resolution
* the type of an implicit should be explicit

Implicit parameter usages:
* static dependency injection, e.g. `ExecutionContext`
* additional type information, e.g. `ClassTag`
* additional type constraints, e.g. `<:<`
* type classes - later

Implicit conversions:
* as part of "implicit parameters" feature, the compiler looks for implicits when it needs something to pass as an implicit parameter
* as part of "implicit conversions" feature, the compiler also looks for implicit values of type `A => B` when it has value of type `A` in a context where value of type `B` is required
* implicit `def`s are implicit conversions by eta-expansion
* additionally, as part of "implicit conversions" feature, the compiler looks for implicit conversions which would allow it to call some otherwise unresolved member of some type - extensions methods
* `implicit class` syntax, value classes
* extension methods - actually more powerful than regular methods

Convenience implicit conversions - why not cool:
* we usually want the conversion to work in only one specific context, but it will pollute everything
* alternative: extension methods and "converters"
* extension methods and overloading
* implicit conversion to controlled type - magnet pattern example
* still annoying - implicit conversions don't chain
* refactor method into generic and implicit conversion stops working

Type classes:
* the idea in Haskell - `Monoid` example
* a way to do polymorphism
* scala encoding using implicits, ops implicit class
* `Ordering`, `Numeric`
* more powerful than inheritance
 * we can define typeclass instances for arbitrary types, not classes
 * we can define typeclass instances for types not defined by us
 * we can express complex rules for type class instances and dependencies between typeclasses
* `Foldable`, higher-kinds and ops