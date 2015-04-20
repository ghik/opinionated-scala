# Methods and operators

This section explores various syntactic features of Scala that can be used when defining and using methods and operators. Most of the things describe here can be quite "magic" and very confusing for Scala newcomers. That is why I decided to list them all fairly early in the guide. You can treat this section as a reference that you can come back later to if you meet any of strange syntactic creatures listed below.

## Collections - syntax basics

Scala has a vast and fairly complicated collection library. It definitely deserves a separate section, so we won't go deeply into it, but we will use collections as an example to present some common patterns and syntactic peculiarities often used when working with them.

### The `apply` magic method

#### Creating collections

Usually, a collection in Scala can be created by simply "calling" a collection name and passing elements to it. For example, to create a `Vector` of elements 1, 2 and 3, we can write:

```scala
val v = Vector(1,2,3)
```
##### Note about `Vector`

`Vector` in Scala is a very handy collection. It is an immutable sequence (a collection with defined order of elements) with near constant-time access to its element by index. It also has very fast append and prepend operations while still staying an immutable. This is because a vector created by appending something to other vector can share most of the data with its predecessor. Scala's `Vector` class implements this in some clever way. Immutable data structures with an ability to create new instances based on old ones by sharing most of the data are usually called *persistent data structures*.

##### Coming back to the syntax

But what is actually happening in our code? We call `Vector` as if it was a function. It turns out that what we actually do is calling an `apply` method on an `object` called `Vector`:

```scala
val v = Vector.apply(1,2,3)
```

This is simply because all methods named `apply` are treated "magically" by the Scala compiler. They can be used as if they were some kind of overloaded function call operator. That is why we can omit the name `apply` when calling `Vector.apply(1,2,3)` and simply write `Vector(1,2,3)`.

You may also note that type inference is also working here. If we inspect the type of `v` (e.g. by hitting `Alt`+`=` in IntelliJ IDEA while having cursor on `v`), we will see that its type is `Vector[Int]`. Scala compiler has inferred the type of vector element from the values that we passed to the `apply` method. A completely desugared version of our invocation would look like this:

```scala
val v = Vector.apply[Int](1,2,3)
```
##### Note about companion objects

It may also be somewhat strange to you that there is an object named `Vector` even though there is already a class named `Vector` (which represents the collection). Scala allows to define a class and an object (or a trait and an object) in the same file and give them the same name. Such object is then called a *companion object* of its class and the class is a *companion class* of its object.

#### Accessing collection elements

To access vector elements by index, Scala also uses the `apply` magic method. We can look into the definition of `Vector` class and we will see a method with such signature:

```scala
// 'A' is the type of Vector elements
def apply(index: Int): A
```

Note that this `apply` method is different from `apply` method we used before to create a vector. That one was defined in the `Vector` *object* but the one we're going to use now is defined in `Vector` *class*. This means that our `v` value has it. We can use it like here:

```scala
val firstElement = v(0)
```

which is of course the same as:

```scala
val firstElement = v.apply(0)
```

Due to existence of magic `apply` method, Scala doesn't need any special operator for array access (like square brackets in C or Java). Scala treats arrays just like any other collection:

```scala
val args = Array("some", "args", "here")
val firstArg = args(0)
```

### The magic `update` method

However, unlike `Vector`, arrays are mutable. We need a way to modify each element of the array. This syntax will do it:

```scala
args(0) = "new first arg"
```

This works thanks to another magic method. Scala compiler will understand the code above as:

```scala
args.update(0, "new first arg")
```

So everything works simply because `Array` has a method named `update` that takes two arguments.

## Methods and operators