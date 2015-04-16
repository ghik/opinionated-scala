## Dissection of Hello World

```scala
object Main {
  def main(args: Array[String]): Unit = {
    println("Hello")
  }
}
```

### Objects

Scala's `object` creates a singleton. In above example, the identifier `Main` refers to the singleton instance itself, not to its class. This means that `Main` can be used as a regular value, e.g. it can be passed as an argument to some function.

### Method definition

Our singleton class has one method called `main`. As we can see, method definition syntax differs from Java:
* method declaration or definition is denoted by the `def` keyword, followed by name of the method
* arguments are declared with `name: Type` syntax
* formal argument list is followed by return type declaration, in our example it is the `: Unit` part
* return type declaration is followed by `=` sign and a method body

#### The `Unit` type



## String interpolations

* String interpolations allow splicing arguments into string literals

  ```scala
  val name = "Fred"
  s"My name is $name, uppercased ${name.toUpperCase}"
  ```
* `s` is only one of the available interpolators, there are also `f`, and `raw` in standard library **MORE...**
* It is possible to define custom interpolators - each interpolator may do whatever it wants with its arguments and 
  String literal parts, it may also return arbitrary type as an interpolation result, not just a string
* String interpolations suffer from some annoying syntactic quirks **MORE...**
  
## Multiline strings

* It is possible to define multiline strings in Scala using triple-quotes

    val ms = 
      """some long
        |multiline text here
        | "nothing" needs escaping
        | \ more stuff \
        |
        |""".stripMargin
        
* Multiline strings treat all characters literally, not just newlines - you don't have to escape anything
* Multiline string interpolations are also possible

## Type system basics

* Scala toplevel type is `Any` which has two direct subtypes - `AnyVal` and `AnyRef`
* `AnyVal` is a common supertypes for non-nullable types which includes mostly equivalents of Java primitives.
  However, these types are more powerful than Java primitives, because they can be used as type parameters (generics).
  It is also not always guaranteed that they are represented as actual primitives in runtime (e.g. when they are used
  as collection elements).
* Scala standard library defines implicit conversions between primitives and `java.lang.*` boxed equivalents of primitives.
* It is possible to define custom types that extend `AnyVal` - value classes **ADVANCED, MORE LATER**
* `AnyRef` is equivalent to `java.lang.Object`. This is the base type for all nullable types - classes, traits, Java
  interfaces.
* All nullable types have also a common bottom *subtype*, the `Null` type
* All types have common bottom type, the `Nothing` type - the only valid expression that has type `Nothing` is something
  that throws an exception (or is elided by compiler).
* We may request that a compiler treats some expression as a particular type using type ascription, e.g. `0: Long`
* Type ascription is can be used for upcasting and forcing usage of implicit conversions.
  However, it must be a "safe cast" - the compiler will report an error when it is unable to match types or 
  find an implicit conversion.
  
## When you're force to go untyped  
  
* The syntax for ugly, unsafe cast is `x.asInstanceOf[SomeType]` - should obviously be avoided
* `null.asInstanceOf[T]` *always* works - if `T` is primitive, the value will be appropriately typed "zero"
* `x.asInstanceOf[AnyRef]` *always* works - it simply forces boxed representation of `x`
* The equivalent of Java's `instanceof` operator in scala is `x.isInstanceOf[SomeClass]` - should be avoided even more
* Pattern matching is also doing runtime type checks and has nicer syntax **LATER**
* Class objects in Scala are accessed using `classOf[SomeClass]` syntax (equivalent of Java `SomeClass.class`)
* **SUBJECTIVE** `AnyVal` and `AnyRef` are pretty much useless (to be used in APIs). 
  If you really need to use toplevel type (i.e. in situations similar to when you use `java.lang.Object` in Java), simply use `Any`.
  If some Java API requires you to pass `java.lang.Object`, just cast your `Any` to `AnyRef` (which as we mentioned earlier,
  always works).

## Collection basics - syntax

* There is no separate operator for accessing array/collection/map elements by index/key
* Instead, when something has a method named `apply` (a magic method), it may be called with just parens (i.e.
  sort-of overloaded call operator)
* Most collections use `apply` for element-by-index access, i.e. `someArray(0)`, `someMap("key")`
* The `apply` method is also defined on companion objects of collection and can be used for colleciton creation, e.g.
  `Array(1,2,3)` is actually `Array.apply(1,2,3)` where `Array` in this context is a companion object of `Array` class.
  Type inference also works here to recognize that it's actually `Array.apply[Int](1,2,3)`
* In order to modify some element of a mutable collection, you can say `someArray(0) = 5` which actually desugars to
  `someArray.update(0, 5)` - the `update` method is another magic method

## Operators and infix syntax

* In scala, there is no distinction between methods and operators
* Every method that takes single argument may be called with an infix syntax
* An operator is just a method with symbolic name
* Methods taking multiple parameters may also be called using infix syntax, but it's annoying when you want to pass
  a tuple as an argument
* There is also a "complementary" syntactic sugar called autotupling which may automatically build a tuple out of
  multiple arguments when they are passed to a method accepting single argument.
* Operator precedence is governed by the first character in operator/method name **LINK**
* Normally, operators called with infix syntax are left-associative
* **IMPORTANT** Operators which have a colon `:` at the end are treated specially - the arguments are flipped when 
  translating to a standard method call and they are right-associative
* Syntactic convention: use infix syntax for operators (methods with symbolic names) and standard syntax for methods
  with alphanumeric names
* Methods with no parameter lists vs methods with single but empty parameter list
  
## Basic functional stuff

* Function objects
* Lambdas - full and shorter syntax
* Eta expansion
* Higher-order functions
* Closures - closing over variables, this and return
