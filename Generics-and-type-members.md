*Parametric polymorphism* is a key feature of statically typed programming languages. It is essential for writing expressive, typesafe and reusable code.

Parametric polymorphism found in Java and Scala is usually known as *generics*. It allows various declarations and definitions to take *type parameters*. In Java, classes, interfaces and individual methods can be made generic by taking type parameters. In Scala, type parameters may be taken by methods, classes, traits and also type aliases and type members.

### Type parametrization - basic syntax

Java uses angle brackets (`<>`) for denoting generics - Scala changes them to square brackets (`[]`).

Here's an example generic method in Scala - it takes a value and duplicates it into a pair:

```
def double[T](value: T): (T,T) = (value, value)
```

Here, type parameter `T` serves to express the fact that return type of a method is determined by the type of `value` parameter. This way the method is actually *polymorphic* - you can think about this as if there is a separate method for every type `T`.

Here's another example:

```
def listOfTwo[T](first: T, second: T): List[T] = List(first, second)
```

In this example, we not only express the fact that return type is determined by type of arguments but we also express a type constraint - arguments may be of any type but both of them must be of the same type.

Above examples show generic methods. The same syntax is used to define generic classes and traits:

```scala
class Box[T](var boxedValue: T)
trait Callable[T] {
  def call(): T
}
```

Type aliases can also be generic:

```scala
type JList[T] = java.util.List[T]
```

We'll talk about abstract type members later.

### Bounds

Type parameter may be constrained by type bounds. Type bounds define that given type parameter must always be either a subtype or supertype of some other type. Java uses `extends` and `super` keywords for type bounds. Scala invents its own pair of symbols for that, `<:` and `>:`:

**TODO**
* `<: AnyRef` vs `>: Null`

### Wildcards, existentials

**TODO**
* existentials full syntax

### Type members

**TODO**
* orthogonal to type parameters
* good when set by a subclass
* f-bounded polymorphism example
 * `this.type`
