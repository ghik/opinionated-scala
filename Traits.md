Traits are a rich and complex feature of Scala and thus deserve a separate section.

Traits, defined with the `trait` keyword are something between Java interfaces and abstract classes. In fact, we'll see that they *almost* can do everything that abstract classes can, but at the same time they fulfill all roles of Java interfaces.

### Traits as interfaces

In their most simple form, `trait`s are equivalent to Java interfaces:

```scala
trait StuffHandler {
  def handleStringStuff(stuff: String): Unit

  def handleIntStuff(stuff: Int): Unit
}
```

In this form the `trait` contains only public, abstract methods. Just like with Java interfaces, such a trait may extend multiple other traits and Java interfaces (which are actually seen by Scala type system as traits) and can be implemented by classes. A class may implement multiple interfaces.

Also, note that such Scala trait will compile to regular Java interface in bytecode.

### Method implementations

In Java 8, interface methods may have default implementations ("default methods"). Scala can do this too (although it's not called "default methods"):

```scala
trait StuffHandler {
  def handleStringStuff(stuff: String): Unit

  // just handle Int stuff by converting it to String
  def handleIntStuff(stuff: Int): Unit =
    handleStringStuff(stuff.toString)
}
```

**NOTE**: Traits were designed and implemented long before Java 8 and therefore compile to different bytecode than Java 8 deafult methods (at least for Scala 2.11).

At this point, however, the similarities between Java interfaces and Scala traits end. Now we're going to show capabilities of traits which will lead us much more closer to abstract classes.

### Inheritance syntax

Scala has two keywords for expressing inheritance (which includes mixing in traits): `extends` and `with`. However, as we will explain shortly, be aware that `with` is not an equivalent of Java's `implements`.

When a class extends another class, it uses `extends`:

```scala
class Base
class Impl extends Base
```

When it wants to mix in some traits above the base class, it does it using `with` keyword:

```scala
class Base
trait Logging
trait Utils
class Impl extends Base with Logging with Utils
```

However, a class doesn't need to declare its abstract class, but directly mix in some traits:

```scala
trait Logging
trait Utils
class Impl extends Logging with Utils
```

As you can see, we're still using the `extends` keyword even though we don't explicitly inherit from any base class. Also, when a trait inherits from other traits, it uses `extends`, too:

```scala
trait Logging
trait Utils
trait Commons extends Logging with Utils
```

It just seems like `extends` is simply always used as for the first thing that we inherit (regardless of whether it's a class or trait) and if we have more traits to mix in then we use the `with` keyword. 

This rule may seem somewhat arbitrary and unintuitive. However, there's a fairly simple way to make sense of it. Line like this one:

```scala
class Impl extends Base with Logging with Utils
```

should actually be read as:

```scala
// this is actually not a correct Scala code, but it's good to think about it that way
class Impl extends (Base with Logging with Utils)
```

So it's good to think about it as if we're mixing `Base`, `Logging` and `Utils` together and then `Impl` class extends that unnamed mix.

**NOTE**: The order of traits mixed in by class or trait definition **does matter** - you will soon see why. Also, base class must always come immediately after `extends` keyword.

### Much more than public methods

Unlike in Java interfaces, methods don't have to be public - they may have any access qualifier that a class method may have. They can also be `final` (Java 8 default methods can't)

```scala
trait SafeStuffHandler {
  final def handleSafely(stuff: String): Unit =
    try handleUnsafely(stuff) catch { 
      case NonFatal(t) => t.printStackTrace()
    }
  
  protected def handleUnsafely(stuff: String): Unit
}
```

### Data and state

Just like classes, `trait`s in Scala can have data and state in the form of `val`s, `lazy val`s, `var`s and `object`s. They may be abstract or concrete, they may implement, override, be final, etc.

This is a real game changer, because now we can no longer think about traits as slightly better interfaces. A trait that has state and data can now hold significant amount of inheritable implementation and no longer represent just some abstraction.

That's why we usually don't say that traits are "implemented" but rather "mixed in". A common usage of traits is to split large class implementation into a set of separate traits and "mix" them into a class. This shows a fundamental conceptual difference between Java interfaces and Scala traits: interfaces are for expressing abstraction (which may have some default behavior to allow easier interface evolution) while traits are much more capable of being reusable pieces of implementation.

**TODO** some nice example?

### Traits can extend classes

Traits can extend other traits and thus call, implement, refine and override inherited members. But a trait can also inherit from class and do the same things with its members (almost):

```scala
abstract class BaseHandler {
  def handle(stuff: String): Int = println("Handling!")
}
trait LoggingHandler extends BaseHandler {
  override def handle(stuff: String): Int = {
    println(s"About to handle $stuff")
    super.handle(stuff)
  }
}

abstract class BetterHandler extends BaseHandler {
  def doBetterWork(): Unit = println("Doing it!")
}

class ConcreteHandler extends LoggingHandler
class BetterConcreteHandler extends BetterHandler with LoggingHandler
```

The `LoggingHandler` trait is a simple example of trait created for some code reuse. Every class that extends `BaseHandler` and wants to add some logging to its `handle` method can just mix in the `LogginHandler` trait.

### Multiple class inheritance?

You may think: if a trait can extend a class and a class can mix in multiple traits, does that mean that a class can have multiple (unrelated) base classes?

The answer is **no**. Every class in Scala can still have only one (direct) superclass. In our previous example, the superclass of `ConcreteHandler` is `BaseHandler` (implicitly inherited as a result of mixing in `LoggingHandler`) and superclass of `BetterConcreteHandler` is `BetterHandler` (explicitly inherited). There is no conflict, even though `LoggingHandler` extends `BaseHandler` because `BetterHandler` also extends `BaseHandler`. However, if we were to write:

```scala
abstract class BaseHandler
abstract class Unrelated
trait LoggingHandler extends BaseHandler
// error!
class ConcreteHandler extends Unrelated with LoggingHandler
```

We would get a compilation error, because `BaseHandler` and `Unrelated` can't be both superclasses of `ConcreteHandler`.

### Initialization code

We have already announced that traits can have `val`s, `lazy val`s, `var`s and `object`s. This means that traits may have some initialization code associated with them.

In fact, traits have a "constructor". Just like in a class or object, the body of a trait can have some code which will be executed when an instance is created.

However, the fundamental difference from classes is that **trait constructor cannot take parameters**.

### Linearization

Multiple implementation inheritance and the fact that traits have an initialization code ("constructor") creates some serious challenges:

* the diamond problem: what happens when a class mixes in the same trait multiple times?
* conflict resolution: how to resolve conflicts between two inherited implementations of the same method?
* initialization order: in what order should base class constructors and trait "constructors" be executed when an instance is created?

Scala resolves these issues with a mechanism called *linearization*. It is an algorithm that takes all base classes and mixed in traits of some class and orders them into a linear sequence. The way this happens is well defined and deterministic.

Linearization algorithm can be summarized a bit more formally as follows: having a class `C` which `extends B1 with B2 with ... with Bn`, the linearization of class `C` is defined as follows:

```
lin(C) = C >+ lin(Bn) >+ lin(Bn-1) >+ ... >+ lin(B1)
```

where the `>+` operator denotes concatenation that additionally removes all elements from its left-side operand which are duplicated in the right-side operand.

Let's see an example:

```scala
trait A
trait B extends A
trait C extends A
trait D
class E extends B with C with D
```

So:
```
lin(E) = E >+ lin(D) >+ lin(C) >+ lin(B)
lin(D) = D
lin(A) = A
lin(C) = [C, A]
lin(B) = [B, A]
// here we drop the duplicated A from the left side
lin(C) >+ lin(B) = [C,A] >+ [B,A] = [C,B,A]
lin(E) = [E, D, C, B, A]
```

**NOTE**: actually, `lin(E)` is `[E, D, C, B, A, AnyRef, Any]` but we omitted automatic `Any` and `AnyRef` for brevity.

Note that this algorithm makes it clear why the order of declared mixed in traits is important!

Linearization gives us clear answers to all three questions we had earlier:

### The diamond problem

Linearization removes duplicates, so that state and data from the same trait cannot be inherited multiple times independently.

### Conflict resolution

Linearization order gives us clear priority of one implementation over another. Implementations with earlier position in the chain have priority over the ones after them. The defining class itself has highest priority. Additionally when some member implementation refers to its `super` implementation, the `super` reference will actually point to the next implementation on the linearization order.

**TODO** example

#### Explicit `super` references

**TODO**

#### Stackable traits

**TODO**

#### `abstract override`

**TODO**

### Initialization order

Class constructors and trait "constructors" are invoked according to reverse linearization order, i.e. starting with `Any` and ending with the constructor of the defining class itself.

#### Common pitfalls

**TODO**
