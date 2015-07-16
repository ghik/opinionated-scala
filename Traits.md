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

##### Anonymous mix-ins

It's also possible to instantiate anonymous classes which mix in multiple traits. For example:

```scala
val b: Base = new Base with Logging with Utils
```

##### Intersection types

You may encounter the `with` keyword being used in another context - in type declarations. For example, you may see something like this:

```scala
val ab: A with B = ???
```

Such type should be read as "something that is an instance of both `A` and `B`". Unfortunately, usage of `with` keyword is a bit accidental here, because its meaning is substantially different than in inheritance syntax. In particular:
* `A` and `B` can be arbitrary types, not just classes or traits
* the order of types does not matter

  For example, if `A` and `B` are traits, this code will compile:
  
  ```scala
  val awb: A with B = new B with A
  ```

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
* initialization order: in what order should base class constructors and trait "constructors" be executed when an instance is created?
* conflict resolution: how to resolve conflicts between two inherited implementations of the same method?

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

Linearization gives us fairly clear answers to all three questions we had earlier:

### The diamond problem

Linearization removes duplicates, so that state and data from the same trait cannot be inherited multiple times independently.

### Initialization order

Linearization unambiguously resolves initialization order - constructors and trait initializers are invoked one after another, in the reverse initialization order, i.e. starting with `Any` and ending with the constructor of the instantiated class itself.

```scala
trait A { print("A")
trait B extends A { print("B") }
trait C extends A { print("C") }
trait D { print("D") }
class E extends B with C with D { print("E") }
```

In the above example, linearization of `E` is `[E,D,C,B,A]` and so invoking `new E` would print

```
ABCDE
```

### Conflict resolution

When inheriting multiple implementations of the same method from more than one base trait, then not only linearization order is important but also the usage of `override` and `final` keywords in conflicting implementations.

Example:

```scala
trait A { 
  def someString: String
}
trait B extends A {
  def someString = "B"
}
trait C extends A {
  def someString = "C"
}
```

Now, if we create a class that mixins both `B` and `C`, we'll get a compilation error:

```scala
// error: class D inherits conflicting members
class D extends B with C
```

There are multiple ways this conflict can be resolved, either by leaving the job to linearization or doing it explicitly:

* Override in class `D`:

  ```scala
  class D extends B with C {
    override def someString = "D"
  }
  ```
* Override in class `D` and refer to super implementation:

  ```scala
  class D extends B with C {
    override def someString = super.someString
  }
  ```
  The `super` keyword will choose first implementation of `someString` available in the linearization chain. In this case, the linearization is `[D,C,B,A]` and because of that, implementation from `C` will be used.
  Of course, the implementation of overriding method is not limited to calling super implementation - it may contain arbitrary code and call super implementation at any point.
* Override in class `D` and explicitly refer to particular super implementation:

  ```scala
  class D extends B with C {
    override def someString = super[B].someString
  }
  ```
  In this case, we have used a new syntax - qualified `super` keyword - to explicitly state that we want to use implementation from trait `B`.
* Add an `override` modifier to implementation in `C`:
  ```scala
  trait C extends A {
    override def someString = "C"
  }
  ```
  By adding an `override` modifier to `someString` method in trait `C`, we say that "implementation in `C` has priority over whatever comes after it in linearization order". In our case the linearization is `[D,C,B,A]` (`C` is before `B`) and that's why `C`'s implementation has priority over `B`'s. However, scala compiler required us to explicitly allow such overriding with an `override` keyword.

  If we added the `override` to implementation in `B` instead of `C` or changed `D` to extend `C with B` instead of `B with C`, this would start to fail again. We could also add `override` to both implementations - this way we could declare classes that extend both `B with C` and `C with B` and linearization would always resolve the conflict automatically.

#### The undetermined meaning of `super`

Previous examples for conflict resolution have shown a very peculiar property of Scala's `super` keyword: when some trait refers to a `super` implementation, it does not know where that implementation is until some class actually mixes in that trait and linearization takes its final word. This is a serious semantic difference from Java and can be very surprising.

In particular, note that a trait `B` defined like this:

```scala
trait A { def method: String = "A" }
trait B extends A { override def method: String = super.method + "B" }
```

may not assume that the `super.method` will refer to `super[A].method` despite the fact that `A` is a direct supertype of `B`. When some class mixes in `B`, there may be another trait put "in between" `A` and `B`.

#### `abstract override`

We have just shown that when a trait overrides a method from its supertype and refers to super implementation, someone may provide that super implementation later - at the moment our trait is mixed in to a class. This in particular means, that when the trait is defined, the super implementation may not be there at all!

This means that the trait can actually *override* an *abstract* method. Scala has a special combination of keywords for that: `abstract override`. Example:

```scala
trait Abs {
  def someString: String
}
trait AppendMoreStuff extends Abs {
  abstract override def someString = super.someString + " and more stuff"
}
```

We have overridden an abstract method and at the same time referred to `super` implementation. That implementation is not yet there - a (non-abstract) class which finally mixes in our `AppendMoreStuff` trait must "inject" some implementation of `someString` under `AppendMoreStuff`. For example:

```scala
trait AbsImpl extends Abs {
  def someString = "Impl"
}
class Klass extends AbsImpl with AppendMoreStuff
```

The value returned by `someString` method on class `Klass` would return `Impl and more stuff`.

The fact that a trait is not necessarily bound to any super implementation gives us an ability to reuse traits and compose them in many different configurations. Traits are "put over" another or "stacked" in potentially arbitrary order. That is why you may often hear about *stackable traits* when reading about Scala.

#### `super` references to classes

**TODO**

### Pitfalls and limitations of traits

#### Initialization traps

Here's an example of one of the most common pitfalls of inheritance in Scala (or maybe even entire Scala language). It deals with the initialization order and the fact that `val`s can be abstract and overridden.

```scala
trait A { 
  val someString: String
  val someStringUpper = someString.toUpperCase
}
class B extends A {
  val someString = "String from B"
}
```

Now, what will happen when we instantiate `B`? It turns out, we'll get a `NullPointerException` at `someString.toUpperCase`! Why is that?

This is a direct consequence of initialization order of `B`. From linearization we know that initialization code of `A` will execute before constructor of `B`. So here's exactly what happens when we try to invoke `new B`:

1. New object is created. No constructors or initializers have been invoked yet. The fields that hold values of `someString` and `someStringUpper` are therefore `null`.
2. Initializer of `A` is invoked. It tries to assign an initial value to `someStringUpper` and evaluates `someString.toUpperCase` which unfortunately fails with a `NullPointerException` because `someString` didn't get the chance to be initialized by constructor of `B`.
3. If it weren't for the NPE, constructor of `B` would be invoked here and it would assign the initial value to `someString`.

So, how do we fix it? There are several ways, but all have their limitations:

* Implement the abstract `val` with a constructor parameter:

  ```scala
  trait A { 
    val someString: String
    val someStringUpper = someString.toUpperCase
  }
  class B(val someString: String) extends A
  ```
  Fields that have their values from constructor parameters are initialized *before* super constructors and initializers, that's why this works.
* Implement your abstract `val` with a `lazy val`:

  ```scala
  trait A { 
    val someString: String
    val someStringUpper = someString.toUpperCase
  }
  class B extends A {
    lazy val someString = "String from B"
  }
  ```
  This works, but note that this also causes `someString` to be initialized *before* the constructor of `B` is invoked. Therefore, you must be careful not to refer to something that is initialized by this constructor (e.g. another `val` defined in class `B`) in the initial value of your `lazy val`.
  For example, if you do this:

  ```scala
  class B extends A {
    lazy val someString = s"Number is $someNumber"
    val someNumber = 42
  }
  ```
  then the NPE comes back, but at a different place.

  Also, notice an unintuitive fact: `lazy` keyword is normally supposed to *delay* the initialization of a field, but here it does the opposite - it actually makes it happen *earlier* than normally.
* Make the non-abstract member (i.e. `someStringUpper`) a `lazy val` or `def`:

  ```scala
  trait A { 
    val someString: String
    lazy val someStringUpper = someString.toUpperCase
  }
  class B extends A {
    val someString = "String from B"
  }
  ```
  However, in order for this to work, you must ensure that nobody uses `someStringUpper` before constructor of `B` initializes `someString`. For example, if you do this:

  ```scala
  trait A { 
    val someString: String
    lazy val someStringUpper = someString.toUpperCase
    val inBrackets = s"[$someStringUpper]"
  }
  ```
  then everything will fail again.
* As an absolute last resort, use an *early initializer*:

  ```scala
  trait A { 
    val someString: String
    lazy val someStringUpper = someString.toUpperCase
  }
  class B extends {
    val someString = "String from B"
  } with A
  ```
  This is a bizarre syntax that we have not yet shown in this guide. It forces the `someString` field to be initialized before initialization code of `A` is executed. This feature is a very dark corner of Scala and should be avoided as much as possible. If you find yourself using it, you should probably look again at your code and find a way to redesign it so that it doesn't depend so much on initialization order.

#### Calling protected methods from classes

**TODO**

### Self-type annotations

**TODO**

