Variance is a problem that appears in the area of interaction between generics/type members and subtyping.

Suppose that we have a generic class `Box` along with some simple class hierarchy:

```scala
class Box[T](var value: T)

abstract class Animal
class Frog extends Animal
abstract class Mammal extends Animal
class Cat extends Mammal
class Dog extends Mammal
```

Now, can we do this?

```scala
val mammalBox: Box[Mammal] = new Box[Mammal](new Dog)
val animalBox: Box[Animal] = mammalBox //error!
```

The answer is no, `scalac` will give us an error and it will be right about it, because otherwise we could do this:

```scala
animalBox.value = new Frog // putting a frog into a box only for mammals :(
```

which would be totally wrong, because we expect that `mammalBox` contains only mammals and we've just put a frog into it. Sooner or later we're going to see a `ClassCastException` when trying to access the `value` of `mammalBox`.

But what if our intention was not to put `value` into `animalBox` but only accessing it? Then it would be totally safe, because `Mammal` is a subtype of `Animal`, so without any problem we could do:

```scala
val animal: Animal = animalBox.value
```

Maybe there's some way to tell the compiler that we only want to use the "safe" portion of `Box` API?
Actually, we could do it using wildcards:

```scala
val mammalBox: Box[Mammal] = new Box[Mammal](new Dog)
val animalBox: Box[_ <: Animal] = mammalBox
```

Now the compiler knows that `animalBox` is not a box for an animal but a box for something that is an animal but may be more specific. Now it has just enough information to know that getting an animal from the box is fine, but putting an animal into it is not:

```scala
val animal: Animal = animalBox.value
animalBox.value = (new Frog): Animal //error!
```

Now, let's consider a similar problem where we're interested into putting things into a box but not interested in accessing it. For example:

```scala
val mammalBox: Box[Mammal] = new Box[Mammal](new Dog)
val catBox: Box[Cat] = mammalBox //error!
catBox.value = new Cat
```

Scala will, again, refuse to compile the line where `mammalBox` is cast to `Box[Cat]`. And it's right to do so, because accessing `catBox.value` should give us a `Cat` and we know that there's a `Dog` instead. So we would see `ClassCastException` again. In order to tell the compiler that we're only interested in putting cats into `catBox`, we can use another wildcard:

```scala
val catBox: Box[_ >: Cat] = mammalBox
```

Now the compiler knows that `catBox` is a box not just for cats but for some more general, unknown class of animals. This way it will allow us to put a cat into this box, but won't let us assume that there's always a cat in it.

### What goes in and out

When a class has a type parameter, looking into its usages inside this class we can determine whether the generic value goes "in" or "out" of the class (or both or neither). In the case of `Box[T]` the `T` goes both "out" and "in" because we can both access and modify the `value` field.
But if `Box` was immutable, the parameter `T` would only go "out", e.g.

```scala
class ImmutableBox[T](val value: T)
```

On other occasions, a generic may only go "in", e.g.

```scala
trait Consumer[T] {
  def consume(value: T): Unit
}
```

By using wildcards, we are telling the compiler that we want to use only the part of an API of a class where a type parameter goes "out" (e.g. `Box[_ <: Animal]`) or only the part where it goes "in" (e.g. `Box[_ >: Cat]`).

## Usage site variance

This usage of wildcards is known as *usage site variance*. In the first case ("out", `Box[_ <: Animal]`) we're talking about *covariance* and in the second one ("in", `Box[_ >: Cat]`), it's *contravariance*.
They're *usage-site* because wildcards need to be repeated at every usage of `Box` type to indicate covariance or contravariance. This is as opposed to *declaration site variance* which we'll talk about later.

When a type parameter is used in a position where it goes "out" of its class, then we call it a *covariant position*. Conversely, when it's used in a position where it goes "in", we call it a *contravariant position*. And finally, when it's used in a position where it could go both "out" and "in", we call it an *invariant position*.

```scala
class VarianceBox[T] {  
  def value: T = rawValue // covariant position, T goes "out"
  def setValue(value: T): Unit = { // contravariant position, T goes "in"
    rawValue = value
  }
  var rawValue: T // invariant position (or technically a covariant getter with a contravariant setter, independently)
}
```

**NOTE**: when type parameter appears in a constructor parameter, we do **NOT** consider it a contravariant ("in") position, because the value only goes "in" during construction when the exact type of the object is known. Such positions are actually *bivariant* (neither "in" nor "out").

Now we can set out the rules about usage-site variance in a more formal way:
* *upper-bounded* wildcards (e.g. `Box[_ <: Animal]`) let us only use the portion of an API where type parameter appears in *covariant* positions
* *lower-bounded* wildcards (e.g. `Box[_ >: Cat]`) let us only use the portion of an API where type parameter appears in *contravariant* positions

In exchange for the limitations imposed by wildcards, we can establish a subtyping relation between otherwise unrelated applications of the outer, parametric class. For example:

* because `Mammal <: Animal` (`<:` = *is-a-subtype-of*), we can say that `Box[Mammal] <: Box[_ <: Animal]`
* because `Mammal >: Cat` (`>:` = *is-a-supertype-of*), we can say that `Box[Mammal] >: Box[_ >: Cat]`

## Declaration site variance

You may have noticed that on many occasions a type parameter either *only* goes "out" or *only* goes "in". For example, all the immutable collections in Scala allow only retrieval of data and therefore the element type parameter appears only in covariant positions (e.g `T` in `List[T]`). In such cases Scala allows us to explicitly mark such type parameter as covariant or contravariant, at the place of its declaration.

In order to declare a type parameter *covariant* ("out"), we use the `+` symbol:

```scala
class ImmutableBox[+T](val value: T)
```

Thanks to this, we no longer need to use wildcards in usage site. Now we can simply say:

```scala
val mammalBox: ImmutableBox[Mammal] = new ImmutableBox(new Cat)
val animalBox: ImmutableBox[Animal] = mammalBox // no need to use wildcards now!
```

At the same time, the compiler now knows that it must ensure that `T` is used in `ImmutableBox` class only in covariant positions. If it's not, we'll get a compilation error.

Similarly, in order to declare a type parameter *contravariant* ("in"), we use the `-` symbol:

```scala
trait Hole[-T] {
  def put(value: T): Unit
}
```

Which allows us to omit lower-bounded wildcards at usage site:

```scala
val mammalHole: Hole[Mammal] = ???
val catHole: Hole[Cat] = mammalHole // no need to use wildcards now!
```

In the same manner, the compiler now checks if `T` only appears at contravariant positions in `Hole`.

### Common uses and examples of variance

The most common situation where a type parameter is declared as covariant are all sorts of immutable data structures like `Option` or `List`. Covariance is natural for them because since they're immutable, data can only be retrieved from them and not mutated inside them. Therefore, the type-parameterized data only goes "out" of them. If you look at various collections in the standard library, you'll see that almost all immutable collection implementations (e.g. `List`, `Vector`) and non-mutable collection traits (e.g. `scala.collection.Traversable`, `scala.collection.Seq`) have their element type parameter declared as covariant. A controversial exception is `Set` which isn't covariant because of its `contains` and `apply` methods (a `Set` can be seen as a predicate, i.e. function from its element type to `Boolean`).

**NOTE**: we have made a distinction between _immutable_ and _non-mutable_ collections. The word _immutable_ refers to collection traits and classes which are guaranteed to never change their contents. These usually inhabit the `scala.collection.immutable` package. However, there are also traits in the `scala.collection` package which are base traits for both immutable and mutable collections. To these we refer as _non-mutable_ because while they may be extended by mutable collection implementations, by themselves they don't expose any mutating API. That alone is enough in order to make them covariant.