# Type system basics

## Unification of types

**Java** type system divides its types into two groups: *objects*, where every value is an instance of a class that extends from `java.lang.Object` and *primitives* like `int`, `double`, `boolean`, etc. Primitives are treated specially by the compiler. They can be used as types of local variables, fields, method parameters and return values, but do not participate in class hierarchy and and their values are not *objects*. In particular, they cannot be used as type parameters (generics).

Scala is different. It's a purely object oriented language in the sense that every value is an object. That is, all the types have their classes and participate in a single class hierarchy. An outline of this hierarchy is shown below:

![Scala type hierarchy](https://raw.githubusercontent.com/wiki/ghik/opinionated-scala/images/scalatypes.png)

#### `Any`

At the top of entire hierarchy sits the type `Any`. Every single value in Scala is an instance of `Any`. This special class exposes some universal methods and operators available on any value: `==`, `!=`, `equals`, `hashCode`, `toString`, `getClass` and a few others. As you can see, Scala moves some of the methods normally available on `java.lang.Object` to an even more common class.

#### `AnyVal`

`AnyVal` is a direct subtype of `Any`. It does not have any methods by itself, but serves only as a common superclass for the so-called *value* types. Value types include mostly those which correspond to Java primitives - `Unit`, `Boolean`, `Char`, `Byte`, `Short`, `Int`, `Long`, `Float` and `Double`. However, they are different from Java primitives since they are actual Scala *classes*. For example, they can be used as type parameters (e.g. you can have `List[Int]`). However, it is not possible to assign `null` to value types.

Scala primitive classes are largely interoperable with Java primitives:
* Scala tries to represent primitive classes as actual Java primitives in runtime. However, this is not guaranteed. For example, if you create a `java.util.List[Int]`, elements of that list will be actually represented in runtime as `java.lang.Integer` objects.
* Scala methods accepting primitive parameters or returning them are seen by Java as methods accepting and returning Java primitives and vice versa, i.e. the two following signatures are seen as identical when consumed by both Scala and Java type system:

    ```scala
    def stuff(i: Int, d: Double): Long
    ```
    ```java
    public long stuff(int i, double d);
    ```
* Scala defines the same implicit conversions between primitive types as Java does - these are marked on the diagram with dashed arrows. It also maintains implicit conversions between primitives and their boxed counterparts. For example, you can pass `Int` whenever a `java.lang.Integer` is expected and vice versa. These conversions are not shown on the diagram.

Apart from counterparts of Java primitives, it is also possible to define custom classes that extend `AnyVal`. These are so called *value classes* and are more advanced concept that will not be covered here.

#### `AnyRef`

`AnyRef` is simply an alias for `java.lang.Object`. It is the common superclass of all *reference* or *object* types. Entire hierarchy of Java classes sits here. By default, all Scala classes, traits and objects are also subtypes of `AnyRef`.

Apart from all the standard Java methods available for `java.lang.Object`, Scala type system adds reference equality operators to the `AnyRef` class - these are the `eq` and `ne` operators mentioned in previous chapter. It also adds an artificial `synchronized` method to it, which is a replacement for language-native `synchronized` keyword in Java. For example, the Java snippet:

```java
synchronized(someObject) {
    // some code here
}
```

would be rewritten to Scala as:

```scala
someObject.synchronized {
  // some code here
}
```

Scala also makes `synchronized` an expression which can return a value. E.g. we can compute some value under lock and return it:

```scala
val someInt = someObject.synchronized {
  // some computation that needs synchronization
  42 // returned value
}
```

Classes extending `AnyRef` are nullable.

#### `Null`

`Null` is a special Scala type which has only one value - `null`. It is also a subtype of all reference (object) types, which expresses the fact that you can assign `null` to them.

#### `Nothing`

`Nothing` sits at the very bottom of Scala type hierarchy. It is a subtype of all other types. `Nothing`, as the name suggests, has literally *no values*. If something (i.e. a method) has return type declared as `Nothing`, it is guaranteed to throw an exception in runtime (`Throwable`, to be precise). The most common usage of `Nothing` is to express empty collections. For example, `Nil` - an object which represents empty list in Scala has type `List[Nothing]`. Thanks to that and the fact that `List` is *covariant*, `Nil` can be used as empty list of any element type. This will be all covered in a separate chapter, along with explanation about what this *covariant* thing means.

`Nothing` is sometimes also used as a return type of methods which exists only in compile time, e.g. are elided by *macros*. Don't worry about these now, though - we just mention it for the sake of completeness.

#### Universal traits

It is possible to define some types that directly extend `Any`. These are related to *value classes* and will be explained along with them, in some other (more advanced) chapter.

### Summary

As you can see, Scala largely unifies primitive and reference types by giving them a common hierarchy and allowing to use primitives where they could not be normally used in Java. At the same time, Scala still maintains some distinction between primitives and objects.

### Casting

In Java, there is only one syntax for casting - it is the C-style `(ForcedType)castValue` syntax, but we differentiate between:
* *upcasting* - when we force a value of some class to be treated as value of its superclass. This is a perfectly safe cast. It is usually needed for forcing some particular overloaded variant of a method to be used.
* *downcasting* - when we force a value of some class to be treated as value of its subclass. This is an unsafe cast that will resunt in `ClassCastException` being thrown in runtime if the actual class is not what we asked for.

#### Type ascription

In Scala, you can perform upcasting using the *type ascription* syntax:

```scala
def overloaded(any: Any): Unit
def overloaded(str: String): Unit
overloaded("treatMeAsAny": Any)
```

Type ascription is much more useful than just for overloading resolution, because it can guide Scala's type inference:

```scala
val x = "treatMeAsAny": Any
```

The snippet above is equivalent to:

```scala
val x: Any = "treatMeAsAny"
```

but has a slightly different meaning from the point of view of the type system. The latter means "declare value `x` of type `Any` and assign it a string value" while the former means "declare value `x` and assign it a string value explicitly treated as `Any`". The difference is that the first syntax uses type inference - we don't declare the type of `x` but let the compiler infer it from the assigned value. However, the value we assign is a string up-cast to `Any`, so the type of `x` will also be inferred as `Any`.

Type ascription can also be used to force usage of implicit conversion, e.g. `42: java.lang.Integer` will force 42 - a value of type `Int` - to be converted to `java.lang.Integer`. Note however that the static type of such expression will also be `java.lang.Integer` so this is more than just forcing a boxed representation in runtime.

#### Unsafe casts

Downcasting and other unsafe casts can be performed in Scala with the `asInstanceOf` artificial method, i.e. `value.asInstanceOf[Type]`. Of course, this is unsafe so the only situations where this is justified is when you need to work with completely untyped data (i.e. deserialization) or some untyped external APIs. Sometimes it is also needed to work around limitations of the type system.

Scala also provides the `isInstanceOf` method which is an equivalent of Java `instanceof` keyword. Of course, you should avoid it in the same way as for `asInstanceOf`.

Unsafe casting can also be done with the pattern matching syntax:

```scala
someObj match {
   case str: String => println(str.substring(1))
   case _ => println("Not a string")
}
```

Pattern matching will do both runtime type checking and type casting at the same time. Of course, this is not any less unsafe than `isInstanceOf` and `asInstanceOf`, even if it has nicer syntax. We will cover pattern matching in more detail later.

As a somewhat related detail, class objects in Scala can be accessed using the `classOf` syntax, i.e. `classOf[String]` in Scala is equivalent to `String.class` in Java.

### Handling untyped Java APIs

Due to the fact that hierarchy of toplevel types in Scala is a bit more complicated than in Java by having distinction between `Any` and `AnyRef`, we may sometimes run into problems when using some untyped Java APIs. In most simple cases this is not a problem - Scala compiler conveniently sees methods like this one:

```java
public Object doSomething(Object anything);
```

as:

```scala
def doSomething(anything: Any): Object
```

As you can see, the Scala compiler interprets parameters with type `java.lang.Object` in Java methods as parameters with type `Any`. This is nice because we can pass value types to these methods without explicit conversions to their boxed representations. 

But you may wonder - how can this be correct? `java.lang.Object`, or `AnyRef` is more specific than `Any` - isn't the compiler lying to us about the signatures? Technically, it does, but the truth is that in runtime, values of static type `Any` will need to be boxed anyway which means that they can safely be passed where `java.lang.Object` is required. When someone uses `java.lang.Object` in Java API, it usually means "anything" so it makes sense to assume that if this API was written in Scala, there would be `Any` in the signature and not `AnyRef`.

However, things get a bit more complicated when method signatures are a bit more complex:

Imagine we have a (completely untyped and ugly) Java API for saving some arbitrary value to database. The value is represented as `java.util.Map<String, Object>`:

```java
public void save(java.util.Map<String,Object> fieldsWithValues);
```

This will be seen by Scala as:

```scala
def save(fieldsWithValues: java.util.Map[String,AnyRef]): Unit
```

Let's assume we want to put ints as values into our map:

```scala
val map = new java.util.HashMap[String,AnyRef]
map.put("something", 42) // error: the result type of an implicit conversion must be more specific than AnyRef
```

The above snippet will not compile, because `42` is of type `Int` and it cannot be passed where `AnyRef` is expected. We are forced to request a conversion to `java.lang.Integer`:

```scala
map.put("something", 42: java.lang.Integer)
```

We would prefer to use `Any` instead of `AnyRef` so that we don't have to convert explicitly, but then we won't be able to pass our map to the Java API, which expects `java.util.Map[String,AnyRef]`:

```scala
val map = new java.util.HashMap[String,Any]
map.put("something", 42)
save(map) // error
```

However, we can employ an ugly hack and leverage the fact that if something is declared as `Any`, it will *always* be boxed in runtime. Since `Any` is effectively represented as `AnyRef` in runtime, we can do an ugly but perfectly safe cast:

```scala
save(map.asInstanceOf[java.util.Map[String,AnyRef]])
```

In fact, it turns out that `x.asInstanceOf[AnyRef]` *always* works, no matter what `x` is. Casting to `AnyRef` simply forces a boxed representation. So for example `42.asInstanceOf[AnyRef]` will not fail but force the int to be represented as `java.lang.Integer`. We can use this trick to fool the Scala type system and retain slightly nicer API on Scala side (with `Any` instead of `AnyRef`).

As a somewhat related fact, there's another trick with `asInstanceOf`: it turns out that `null.asInstanceOf[T]` also always works (except for when `T` is `Nothing`). This is because `null` is a valid representation for primitive types and will be understood as "zero" of given type. For example, `null.asInstanceOf[Int]` will evaluate as 0.

All of the stuff described above is of course a dark, dirty corner of the language and its implementation, but may be needed and even make your Scala APIs better sometimes. That's why we decided to mention it here.

#### Summary

The morale of the story is: if you have some untyped value in your Scala code, always represent it with `Any`. Even if some legacy Java API expects `java.lang.Object`, you can always safely cast.