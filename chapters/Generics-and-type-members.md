*Parametric polymorphism* is a key feature of statically typed programming languages. It is essential for writing expressive, typesafe and reusable code.

Parametric polymorphism found in Java and Scala is usually known as *generics*. It allows various declarations and definitions to take *type parameters*. In Java, classes, interfaces and individual methods can be made generic by taking type parameters. In Scala, type parameters may be taken by methods, classes, traits and also type aliases and type members.

### Type parametrization - basic syntax

Java uses angle brackets (`<>`) for denoting generics - Scala changes them to square brackets (`[]`).

Here's an example generic method in Scala - it takes a value and duplicates it into a pair:

```scala
def double[T](value: T): (T,T) = (value, value)
```

Here, type parameter `T` serves to express the fact that return type of a method is determined by the type of `value` parameter. This way the method is actually *polymorphic* - you can think about this as if there is a separate method for every type `T`.

Here's another example:

```scala
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

We'll talk about type aliases and abstract type members later.

### Bounds

Type parameter may be constrained by type bounds. Type bounds define that given type parameter must always be either a subtype or supertype of some other type. Java uses `extends` and `super` keywords for type bounds. Scala invents its own pair of symbols for that, `<:` (*upper* bound) and `>:` (*lower* bound):

A good technique to memorize that `<:` corresponds to `extends` and `>:` corresponds to `super` is to think of these as "less" and "greater" operators. For example, `String` is a subtype of `AnyRef`. This means that every `String` is an `AnyRef` but not vice versa. In other words, set of `String`s is contained in the set of `AnyRef`s, so `String` is "less" than `AnyRef` and so `String <: AnyRef`.

Here's a convenience method that closes a `Closeable` but does it in fluent manner, i.e. it returns its own argument:

```scala
def closed[T <: Closeable](closeable: T): T = {
  closeable.close()
  closeable
}
```

We want the method to retain the actual type of argument passed and that's why we need the type parameter in the first place, but we also need to constrain the parameter so that only `Closeable`s can be passed. That's how the above upper type bound expresses it.

Lower bound can be used in the same manner. It is also possible for a type parameter to have *both* upper and lower bound.

#### `<: AnyRef` vs `>: Null`

A common mistake with bounds is when someone wants to express that the type parameter represents a nullable value. For example, if you wanted to write a method which returns `null` as a value of type parameter, it may seem natural to implement it like this:

```scala
def nullIfFailed[T <: AnyRef](expr: => T): T =
  try expr catch {
    case NonFatal(_) => null
  }
```

The above is a method which tries to evaluate some expression of generic type `T` and if that evaluation fails with an exception, a `null` fallback value is returned. It may seem natural to express the fact that `T` is nullable by placing an upper `AnyRef` type bound on it. `AnyRef` is `java.lang.Object`, so it can be `null`, right?

Well, not so much. If you try to compile the above method, you'll get:

```
error: type mismatch;
 found   : Null(null)
 required: T
           case NonFatal(_) => null
                               ^
```

The type bound `<: AnyRef` is **NOT** the correct way to express "nullability", because:

* There is a type `Nothing`. You'll recall that it's a special type that's a subtype of all other types but has no values. That means it is within the type bound `<: AnyRef` but at the same time `null` can't be returned where `Nothing` is expected.
* `null` is also a correct value of type `Any`, but `Any` doesn't fall within the `<: AnyRef` type bound

The correct way to do this is to place the *lower* bound of Scala special type `Null` (a subtype of all nullable types):

```scala
def nullIfFailed[T >: Null](expr: => T): T =
  try expr catch {
    case NonFatal(_) => null
  }
```

### Wildcards, existentials

Like Java, Scala also has *wildcards* - parameterized types whose type parameters are unknown. Java expresses wildcards with a question mark `?`, e.g. `Set<?>` while Scala uses underscore `_` for that (yet another meaning of underscore in Scala), e.g. `Set[_]`. Such type should be read as "a set of some unknown element type".

Wildcard types may also have type bounds, e.g. `Set[_ <: Number]`.

Thanks to the mechanism of *declaration site variance* (we'll talk about this later), wildcards in pure Scala code are much less frequently used than in Java. In fact, they're mostly needed for interoperability with Java, which doesn't have declaration site variance.

For example, in Java stream API, the `filter` method has the following signature (translated from Java to Scala):

```scala
trait Stream[T] {
  ...
  def filter(predicate: Predicate[_ >: T]): Stream[T]
  ...
}
```

If we used Scala's `Function` instead of `java.util.function.Predicate`, the signature could look like:

```scala
def filter(predicate: T => Boolean): Stream[T]
```

We don't need to use any wildcards thanks to the fact that `Function` is *contravariant* in its input type. We have briefly explained that in [Basic functional constructs](Basic-functional-constructs). We'll also talk more about variance in detail later.

#### Java raw types

For backwards compatibility reasons, Java retained the possibility of using *raw types*, e.g. `Set` instead of `Set<?>`. Please refer to resources about Java for more detailed explanation of raw types. Here, we just mention them to point out that Scala does **not** have raw types.

However, sometimes Scala compiler must understand Java signatures which contain raw types. It does so by translating them to wildcard types. For example, Java raw type `java.util.Set` will be seen by Scala compiler as `java.util.Set[_]`.

#### Existential types

Wildcard types are a special case of a more general concept called *existential types*. For example:

```scala
type S = Set[_]
```

is actually a shorter version of:

```scala
type S = Set[A] forSome { type A }
```

This should be read as "there *exists* type `A` for which `T` is `Set[A]`" - that's why they're called *existential* types.

You may wonder what's the point of that much more verbose syntax. Sometimes one is forced to declare an existential type which is inexpressible in terms of wildcards. An example of such type would be `Map[T,T] forSome { type T }`. Note that this is something completely different from `Map[_,_]` which means `Map[K,V] forSome { type K; type V }` (each wildcard represents a separate unknown type param).

Complex existential types are rarely used - usually only in some more complex cases of Java interoperability. You should generally avoid designing your code around them.

### Type members

You may have noticed that Scala has a `type` keyword. It's used for defining *type aliases* and *type members*.

#### Type aliases

Type aliases are just what you think. You can use a `type` declaration to give a shorter or more meaningful name to some type:

```scala
type IntPair = (Int, Int)
```

Type aliases may also be parameterized:

```scala
type Mapping[A] = Map[A,A]
type JList[A] = java.util.List[A]
```

Type aliases may be defined locally (inside arbitrary block of code) or be *type members* of a class, trait or object:

```scala
class Klass {
  type IntPair = (Int, Int)
}
```

#### Abstract type members

Type members may also be abstract. A declaration of abstract type member looks like this:

```scala
trait MyStack {
  type Elem

  def push(elem: Elem): Unit
  def pop(): Elem
}
```

Abstract type may also have bounds. For example:

```scala
trait NumberCollection {
  type Elem <: Number
}
```

Such abstract type member can then be inherited and made less abstract by other traits or classes. For example, a subtrait of `MyStack` may apply an additional constraint on the type member (while still leaving it abstract):

```scala
trait ObjectStack extends MyStack {
  type Elem <: AnyRef
}
```

Finally, some class or trait may want to make the type member non-abstract, thus making it a type alias in the scope of that class or trait:

```scala
trait StringStack extends ObjectStack {
  type Elem = String
}
```

Quite a confusing property of abstract type members is the fact that even though they're abstract, they don't necessarily need to be "implemented". They may remain abstract even though they are members of non-abstract classes. For example, the following code compiles just fine:

```scala
class Klass {
  type T
}
object Klass {
  new Klass
}
```

However, the type member is not really useful in that example - there's no way to actually create any value of that abstract type.

#### Type members vs type parameters

Looking at our previous examples you may wonder what's the point of using type members for something that can be as easily expressed with plain generics:

```scala
trait MyStack[E] {
  def push(elem: E): Unit
  def pop(): E
}
trait ObjectStack[E <: AnyRef] extends MyStack[E]
```

Type members are indeed in many ways orthogonal to type parameters - they are often two ways of doing the same thing. However, there are some subtle syntactic and typechecking differences between them.

For example, type members may be made non-abstract by "implementing" them with an inner class or trait:

```scala
trait CustomStack extends MyStack {
  class Elem // an inner class becomes a concretization of abstract type member
}
```

You cannot do something like this with a type parameter.

On the other hand, type members cannot be used in constructor parameters. Something like this:

```scala
class Box[T](val value: T)
```

cannot be (easily) expressed using a type member.

Using plain generics it is also possible to refer to types like `MyStack[String]` which doesn't seem doable with type members. 

However, it actually *is* doable - a type-member equivalent of `MyStack[String]` would be `MyStack { type Elem = String }`. This syntax is called a *type refinement* - brackets here don't mean a block of code or a class/trait body but have a completely new meaning (a refinement). We don't need to go much into details about this. The trick with refining a type member should be enough for now.

Actually, you can even define a type alias which will "translate" a type member into type parameter:

```scala
type MyStackAux[E] = MyStack { type Elem = E }
```

It's also possible to create a specialized subclass or subtrait which "translates" the type member to type parameter:

```scala
trait MyGenericStack[E] extends MyStack {
  type Elem = E
}
```

Of course, in this case the "translation" works only inside the hierarchy of that specialized trait, while type alias encompasses every possible subtype of `MyStack`.

#### Referring to type member from outside

Using plain generics, you can write e.g. `MyStack[_]` (a stack of unknown element type). An equivalent version of that using type members would simply be `MyStack`. We haven't used a *type refinement* to bind the `Elem` member to some concrete type and so the type member remains abstract.

However, the difference is not just cosmetic. With generics and wildcards, there is no way to refer to the unknown element type. This is however possible with type members. For example:

The following code will not compile:

```scala
trait MyStack[E] {
  def push(e: E): Unit
  def pop(): E
}
object Test {
  def popAndPush(stack: MyStack[_]): Unit = {
    val elem = stack.pop()
    stack.push(elem) // error!
  }
}
```

The compiler cannot follow the fact that the type of `elem` returned from `pop` is exactly the same type that `push` accepts. The type of `elem` is pretty much `Any`. Using type members, this problem disappears:

```scala
trait MyStack {
  type Elem
  def push(e: Elem): Unit
  def pop(): Elem
}
object Test {
  def popAndPush(stack: MyStack): Unit = {
    val elem = stack.pop()
    stack.push(elem) // no problem!
  }
}
```

Now the compiler is able to track the types properly. This is because type members (even though they're abstract) can be referred to outside of its containing class/trait. In the above example, `elem` has type `stack.Elem`. If we wanted, we could write it explicitly:

```scala
val elem: stack.Elem = stack.pop()
```

`stack.Elem` is an example of *path dependent type*. You may remember this term from discussion about inner classes and traits. Exactly the same rules as these discussed there apply to abstract type members, e.g. if `stack1` and `stack2` are different stack instances, then `stack1.Elem` and `stack2.Elem` are types incompatible with each other (although they still have a common supertype, `Stack#Elem`).

Of course, there is also an easy way to fix our example and still using plain generics - simply make the `popAndPush` method generic:

```scala
trait MyStack[E] {
  def push(e: E): Unit
  def pop(): E
}
object Test {
  def popAndPush[E](stack: MyStack[E]): Unit = {
    val elem = stack.pop()
    stack.push(elem)
  }
}
```

However, adding that generic is somewhat a boilerplate. It isn't very painful in our simple example, but it may become much more annoying when there is e.g. more than just one type parameter, some of the params have type bounds that need to be repeated and you have many places in your code that needs to be made generic just because of that. That's why type members are often much more flexible in such situations.

#### Naming conventions

You are probably used to giving one-letter names to type parameters in Java. This convention generally persists also for generics in Scala, but you should **NOT** carry it over to abstract type members.

Names of type members should be longer and much more descriptive than names of type parameters. This is because names of type members are actually part of the public API, while names of type parameters are not so much. This has two important consequences:

* Changing the name of type member can introduce a serious backwards compatibility breakage
  Let's assume we have a library with a stack trait similar to the one from previous examples:
  ```scala
  trait Stack {
    type Elem
  }
  ```
  Somebody uses our library and implements the stack:
  ```scala
  class IntStack extends Stack {
    type Elem = Int
  }
  ```
  If we now change the `Elem` to something else, the implementor's code would be broken. The `type Elem = Int` will still compile but will just become a type alias unrelated to base trait and the type member from `Stack` will remain abstract.

  Note that such problem doesn't exist with plain generics:
  ```scala
  trait Stack[E]
  ```
  Implementation:
  ```scala
  class IntStack extends Stack[Int]
  ```
  Changing `E` to some other name would not break anything.

* There may be name conflicts between type members inherited from different traits
  **TODO**

#### The my-type problem

Here's a very common problem in Java, Scala and possibly other object oriented languages with inheritance: how to declare a method that returns "self" type? In other words, how to return the same type as the actual type of an instance on which the method is called? For example:

```scala
trait HasName {
  def name: String
  def withName(newName: String): <???>
}
final class Person(val name: String, val surname: String) extends HasName {
  def withName(newName: String): Person = new Person(newName)
}
```

Now, we'd like to replace `<???>` with something that would allow us to write, e.g.:

```scala
def renameToJohn[T <: HasName](hasName: T): T =
  hasName.withName("John")
```

What should we declare as return type of `withName` so that `renameToJohn` compiles? This is a tricky problem that has a lot of alternative solutions. One of these solutions (a fairly nice one) uses type members and that's why we're dealing with it in this chapter. However, let's go through some more basic solutions first before getting to the point.

##### Using `this.type`

From the discussion about inner classes and path dependent types, you may remember something called *singleton types* - types that belong to `object`s, `val`s and `lazy val`s and represent exactly the single instance that lies underneath.

One special example of a singleton type is `this.type`. It's a type that represents the `this` instance inside a class or trait. `this.type` is a simple and nice solution to "my-type" problem, assuming that we always return the same instance. For example, `this.type` is particularly good as a return type of "fluent setter" - a method which mutates some internal state of an object and then simply returns it. For example:

```scala
trait MutableHasName {
  var name: String
  def fluentSetName(newName: String): this.type = {
    name = newName
    this
  }
}
```

We can then do:

```scala
class MutablePerson extends MutableHasName
object Test {
  val person: MutablePerson = new MutablePerson().fluentSetName("John")
}
```

Note that the type of `person` is `MutablePerson` (the concrete class type) even though `fluentSetName` method is defined in `MutableHasName` trait.

This solution is easy and nice but limited - it can't be used if we want to return something else than exactly the same instance. In our initial example, we return a copy of `Person` which is not a correct value of `this.type`.

##### F-bounded polymorphism

This scary name refers to a fairly simple technique often used e.g. in Java to deal with this-type problem.

Let's go back to our original example with `HasName` trait and `Person` class. We can introduce a type parameter that will track the "my-type" information:

```scala
trait HasName[Self <: HasName[Self]] {
  def name: String
  def withName(newName: String): Self
}
final class Person(val name: String, val surname: String) extends HasName[Person] {
  def withName(newName: String): Person = new Person(newName)
}
```

The recursive `Self <: HasName[Self]` bound is not strictly necessary, but it explicitly ensures that `Self` inherits from `HasName`. This is how F-bounded polymorphism is usually implemented in Java. However, this version has a problem: it does not strictly ensure that `Self` is _always_ set to the implementing class type. For example, we could write:

```scala
final class Item(val name: String) extends HasName[Person]
```

Something like this could be a very common mistake stemming from copy-pasting. The compiler does not protect us from this - from its point of view this is perfectly correct code. Because of that the compiler also can't assume that `this` instance is always an instance of `Self` and will reject code like this:

```scala
trait SelfTyped[Self <: SelfTyped[Self]] {
  def returnSelf: Self = this // ERROR, the compiler can't be sure that `this` is an instance of `Self`
}
```

One way to solve this problem in Scala is to use additional self-type annotation:

```scala
trait SelfTyped[Self <: SelfTyped[Self]] { this: Self =>
  def returnSelf: Self = this
}
```

In general, the solution with type parameter (generic) is nice, but introduces some syntactic crust. We can no longer simply use `HasName` as a standalone type, now we have to always write `HasName[_]` which may be annoying.

##### Type member solution

Let's replace type parameter with type member:

```scala
trait HasName {
  type Self >: this.type <: HasName
  def name: String
  def withName(newName: String): Self
}
final class Person(val name: String, val surname: String) extends HasName {
  type Self = Person
  def withName(newName: String): Person = new Person(newName)
}
```

The `>: this.type` lower bound serves the same purpose as `this: Self =>` self-type annotation from type parameter based solution. Note that lower bound must be specified before upper bound.

This solves the problem in pretty much the same way the type parameter did, but relieves us from polluting definition of `HasName` with unnecessary generic crust. We don't have to rewrite all usages of `HasName` to introduce a generic or wildcard.

#### Java interoperability

Type members are purely Scala feature - Java does not see them. If you want your types to be fully visible from Java code, you must use plain generics instead of type members.

#### Type members - summary

Advantages of type members over generics:
* type members don't pollute type declarations of your trait, like generics do
* adding a type member to class or trait is by itself a fully backwards compatible change, while adding a type parameter will require you to rewrite a lot of code that uses your trait or class
* type member may be referred to outside of its declaring class or trait using path-dependent types
* you can concretize an abstract type member with inner class or trait

Advantages of generics over type members:
* generics may be referred to in constructor parameters (and self-type annotation)
* type parameters are recognized by Java
* names of type parameters may be much more freely changed than names of type members (without breaking backwards compatibility)

In general, a type member seems to be a good replacement for type parameter in situations where the generic was only there to be made concrete by various subclasses.

As we have already noted, type members and type parameters are fairly orthogonal concepts that often serve similar purposes. In the possible future version of Scala called [dotty](https://github.com/lampepfl/dotty), type members and type parameters have been merged into a single being.