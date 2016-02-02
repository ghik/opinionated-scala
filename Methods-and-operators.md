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

`Vector` in Scala is a very handy collection. It is an immutable sequence (a collection with defined order of elements) with near constant-time access to its elements by index. It also has very fast append and prepend operations while still staying fully immutable. This is because a vector created by appending something to other vector can share most of the data with its predecessor. Scala's `Vector` class implements this in some clever way. Immutable data structures with an ability to create new instances based on old ones by sharing most of the data are usually called *persistent data structures*.

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

## Methods == operators

*Does Scala have operator overloading? Can we define custom operators in Scala?*

The answer to questions above can be both "yes" and "no". In practice, it is "yes" - Scala allows us to define almost arbitrarily named operators. This can be even abused to write very unreadable Scala code full of bizarre symbolic names like `<++=` and the likes.

However, the questions above can technically be answered "no". That's because Scala has no real distinction between methods and operators. They're the same thing. Operators are just methods. Even the most basic ones, like arithmetic or logical operators are simply methods defined in classes like `Int`, `Boolean`, `Double` etc. For example, addition operator for ints is declared as:

```scala
def +(x: Int): Int
```

But you may think: *Methods and operators are still different. Methods are called using syntax with dot and parens (i.e. `obj.method(arg)`) while operators are used with infix syntax (`a + b`), aren't they?* Apparently, they are not. Both standard "method call" syntax and infix notation can be used for any method and operator. So, you can write `obj method arg` instead of `obj.method(arg)` and `a.+(b)` instead of `a + b`. It is just a good convention in Scala to use "method call" syntax for method with alphanumeric names and infix syntax for methods with symbolic names ("operators").

So there *really* is no distinction between methods and operators in Scala.

### Operator precedence

Ok, so we know that `a + b` is translated to `a.+(b)`. But what if we write `a + b ++ c op d *+* e`? Which operator has the precedence? If Scala only allowed to overload well-known mathematical operators like `+` and `*`, this would not be a problem - it is well known that `*` has precedence over `+` etc. But Scala allows to define completely arbitrary operators, like in the example above. So how does it resolve such situations?

Scala has somewhat peculiar rule for determining operator precedence. It does it by looking at the **first character** of the operator and then applies predefined priority for each one:

* letters have the lowest priority
* `|`
* `^`
* `&`
* `<` `>`
* `=` `!`
* `:`
* `+` `-`
* `*` `/` `%`
* all other symbolic characters have highest priority

As you can see, the order is set up in such a way that well-known operator precedence for mathematical operators is retained.

You don't have to remember that precedence order, but it is very important to remember that such rule exists. When we previously said that Scala allows you to define almost arbitrarily named operators, it may have seemed that it gives us a great freedom in making up fancy names for operators. But the fact that precedence has to be taken into account makes that freedom much more constrained.

In general, be very careful when creating APIs with fancy operators. Or in fact - avoid them. They can easily lead to very cryptic code and is recommended only when you're carefully crafting some kind of well-documented DSL (domain specific language) and meaning of each operator is well-known in that domain. If you're in doubt, it's better to be conservative and stay with plain, alphanumeric names. This is an area of Scala that can be easily [abused](http://www.flotsam.nl/dispatch-periodic-table.html).

### Associativity

OK, so we have the operator precedence set up. But how should `a + b + c` be understood? Is it `a + (b + c)` or `(a + b) + c`?

By default, Scala operators are left-associative, so it is `(a + b) + c`

### Right associative operators

Everything we have said so far has a **very important exception** - it is also a quite arbitrary and non-intuitive one, which makes it even more important to know as soon as possible.

Scala operators whose names **end with a colon** (`:`) are treated specially.

First, the infix notation itself is understood differently - the arguments are **flipped**, So `a +: b` is translated to `b.+:(a)`. To be very precise, it actually translated to:

```scala
{
  // _x is just a placeholder - the compiler always chooses some unused, opaque name
  val _x = a
  b.+:(_x)
}
```

The reason for such strange encoding done by the compiler is to ensure that `a` is evaluated before `b`.

Associativity of colon-ending operators is also flipped. They are **right-associative**. So, `a +: b +: c` is actually `a +: (b +: c)`.

### Assignment operators

*Note*: Some symbols which are usually called "operators" in other languages are called *delimiters* in Scala nomenclature. An example is the assignment symbol `=`. So, in context of this discussion, plain assignment is *not* an operator, but a delimiter.

In one of the very simple code examples in the [first chapter](Dissection-of-Hello-World), we have noted that there are no pre-increment or post-increment operators in Scala, so we had to use `+=` to increment a variable:

```scala
i += 1
```

We have also just said that all operators are methods, but if we look into the `Int` class, we won't find a declaration of `+=` there. So why does it work?

When we use an infix operator that ends with a `=` character, e.g.:

```scala
// <+> is an example, this can be arbitrary operator
l <+>= r
```

and the compiler is unable to find actual method named `<+>=` on `l` then it translates the statement to:

```scala
l = l <+> r
```

### Unary operators

Scala's unary prefix operators `!`, `~`, `+` and `-` are also translated to method calls. For example, negation:

```scala
!something
```

is understood by compiler as

```scala
something.unary_!
```

This way unary operators can also be overloaded. However, this applies only to the four listed operators - `!`, `~`, `+` and `-`. You cannot define any other unary operators.

### Multi-argument infix syntax

Scala allows infix syntax even for methods with more than one argument. So you can write:

```scala
javaMap put ("key", "value")
```

instead of

```scala
javaMap.put("key", "value")
```

Unfortunately, this comes at a price. Parens in Scala are also used to denote *tuples*. So, by `("key", "value")` we may also mean a pair of two strings, `"key"` and `"value"`. For example, to add a key-value pair to a Scala mutable map, we could use its `+=` operator which takes a single argument - a pair.
We'd like to do it like this:

```scala
scalaMap += ("key", "value") // error
```

This won't work, because the compiler will understand it as `scalaMap.+=("key","value")` where we actually meant `scalaMap.+=(("key", "value"))`. In order to make the compiler happy, we need to wrap our pair in more parentheses:

```scala
scalaMap += (("key", "value"))
```

### Autotupling

In other situations, Scala is doing exactly the opposite - it creates a tuple where we meant to pass multiple arguments. Such transformation is called *autotupling*. For example:

```scala
println(1, 2, 3)
```

will be understood by the compiler as `println((1,2,3))`, because there's no `println` method with three parameters. This can be very confusing, so it's good to know as soon as possible.

## No-argument methods

While talking about various syntaxes to call methods in Scala, it's also worth to mention that Scala differentiates between methods that have an _empty parameter list_ and methods that have _no parameter lists_:

```scala
def emptyParamList() = ...
def noParamLists = ...
```

Why is there such distinction and when to use both variants?

Empty parameter list should be used when a method has some side effects (e.g. it invokes some I/O or modifies a variable). Method with no parameter lists should be used when it's *pure* (has no side effects). This is not enforced by Scala type system in any way, but it's a well-established, strong convention.

The same rule applies to invocations of these methods. We would write:

```scala
val x = emptyParamList()
val y = noParamLists
```

However, Scala allows to omit the empty parameter list:

```scala
val x = emptyParamList
```

This is for Java compatibility. Scala sees all no-argument Java methods as methods with empty parameter list. Some of them may be pure, but some may not. When calling Java methods, you should use empty parameter list only when the method has side effects. For example, you should not use it when calling getters:

```scala
val cls = "stuff".getClass
```

If a method with empty param list is defined in Scala, you should never omit the empty param list when calling it.

## Summary

Scala has many syntactic sugars that are meant to make the language more flexible and concise. Unfortunately, these sugars and arbitrary syntax rules can often make it very confusing and sometimes cause annoying ambiguities.