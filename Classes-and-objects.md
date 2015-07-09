## Classes and objects

#### Classes

As we have already seen in many examples in this tutorial - classes in Scala are defined in a standard way - with the `class` keyword and instantiated with the `new` keyword:

```scala
// we can omit braces if class has empty body
class MyClass
// we can omit parens just like for methods with no params
new MyClass
```

#### Objects

We've also seen `object`s. They're essentially singleton classes:

```scala
object Utilities {
  def utilMethod(param: Int): String = ???
}
```

Objects can have everything that a regular class has - methods, fields, inner classes etc. Also just like classes, they can inherit, implement and override.
They are often used as namespaces for "static" functions and constants.

Because each object is a sole instance of its own (unnamed) class, that instance can be referred to. That's why we can do everything with an object that we can do with a regular value, i.e. we can pass an `object` to a method:

```scala
object SomeObject
object Test {
  println(SomeObject)
}
```

It's also important to remember that the sole instance of `object` is lazily initialized upon first access. 

### Primary constructor

Constructors in Scala work somewhat differently than in Java. Every Scala class has a *primary constructor*. The code of that constructor is simply the body of the class. For example:

```scala
class MyClass {
  println("Doing some work in primary constructor!")
}
```

Of course, class body may also contain members (methods, fields, etc.). This way it may actually become an arbitrary mix of class member definitions and pieces of primary constructor code:

```scala
class MyClass {
  println("Creating MyClass!")

  val name: String = { 
    println("Initializing name!")
    "John"
  }

  println("Finished creating MyClass!")
}
```

The order in which pieces of primary constructor code and member initializers execute reflects their order in the source code, i.e. `new MyClass` would print:

```
Creating MyClass!
Initializing name!
Finished creating MyClass!
```

**NOTE**: `objects` also have a primary constructor, but (for obvious reasons) cannot take constructor parameters

#### Primary constructor local variables

The fact that body of primary constructor is mixed with member declarations brings some annoyances: you cannot use local variables in primary constructor code. Every `val` or `var` that you declare will become a member of the class (with underlying field in bytecode) and not just a local variable:

```scala
class C {
  // that's a public member, not a local variable! 
  var x: Int = 0
}
```

This may not always that bad - the only problem with it is that there will be completely unnecessary field in the compiled bytecode that will hold a useless value. This may cause unwanted memory usage but if you don't care you can just declare it as `private` to not expose it outside of your class and be happy with it. However, there are some techniques that could still avoid the problem.

If you need a local variable during computation of initial value of a `val` or `var`, you can assign a block to that field and put the local variable inside that block:

```scala
class C {
  val someValue = {
    var localVar: Int = 0
    // some computations here ...
    someResultExpression
  }
}
```

OK, but what if we need a the same local variable in order to compute initial values for *two* `val`s? Here we can use multi-value assignment (see pattern matching discussion for more details):

```scala
class C {
  val (someValue, someOtherValue) = {
    var localVar: Int = 0
    // some computations here ...
    (firstResult, secondResult)
  }
}
```

This is not ideal (e.g. we cannot document each `val`) but still better than anything.

#### Primary constructor parameters

The primary constructor may, of course, take some parameters. For this, we use a syntax that looks as if the class itself "takes" some parameters:

```scala
class MyClass(myName: String) {
  println(s"My name is $myName")
}
```

Just like methods, the primary constructor may take multiple parameter lists.

The nice thing about primary constructor parameters is that they may be referred to anywhere in the class definition - not just in the constructor code. For example, we can define a method that uses primary constructor parameter:

```scala
class MyClass(myName: String) {
  def printName(): Unit = println(myName)
}
```

On bytecode level, Scala compiler will automatically turn such constructor parameters into class fields so that they can be accessed even after the primary constructor finished its execution.

Primary constructors may also have access modifiers. For example:

```scala
class MyClass private(myName: String)
```

This syntax may look somewhat odd when the constructor has no params and even more odd when the class has empty body:

```scala
// private primary constructor with no params on class with empty body
class MyClass private

// this looks like "private inheritance" but it's just private constructor
class MyOtherClass private extends Object
```

Access modifiers (`private`, `protected` etc) will be described in detail later in this section.

You can easily turn primary constructor parameters to also be members of your class by adding the `val` keyword:

```scala
class MyClass(val myName: String)
// now myName is a constructor param but can also be accessed from outside!
new MyClass().myName
```

Such constructor-parameter-members can have all the modifiers that normal class members can - they can be `private`, `final`, have annotations etc.

#### Auxiliary constructors

Apart from the primary constructor, you may also define *auxiliary constructors*. However - every auxiliary constructor must call some other (previously defined) constructor as its first instruction. This way the primary constructor must *always* be invoked eventually.

The syntax for auxiliary constructors is as follows:

```scala
class MyConnection(hostPort: String) {
  // simply delegate to primary constructor
  def this(host: String, port: Int) = this(s"$host:$port")

  def this(port: Int) = {
    // delegate to other constructor, but do something more afterwards
    this("localhost", port)
    println("No host given, assuming localhost!")
  }
}
```

Auxiliary constructors are similar to overloaded constructors in Java. However, note that in Java they are often used to provide default values for some other constructors' parameters. In Scala this can be achieved much easier with default parameters. 

For example, instead of writing:

```scala
class MyClass(name: String) {
  def this() = this("DefaultName")
}
```

we can simply do this:

```scala
class MyClass(name: String = "DefaultName")
```

The difference between the two is that the first variant is more usable from Java code - Java cannot easily call Scala methods or constructors which have default parameter values.

### Members

Class members are divided into two groups - *term members* and *type members*.
Term members are members which represent values - `val`s, `lazy val`s, `var`s, `def`s and `object`s.
Type members are members which represent types - (inner) `class`es, `trait`s, `type` aliases and abstract `type`s.

#### `def` members - methods

These are just methods. We have already seen multiple examples of them.

#### `val` members

When a member of some class is a `val`, it represents a field and a method at the same time. Every `val` member is backed by a field in bytecode, but the field is private and every access to that field goes through an automatically generated getter - a method with the same name as the `val`.

For example, this class:

```scala
class Klass {
  val x = 5
}
```

yields bytecode equivalent to this Java class:

```java
public class Klass
    private final int x = 5;
    public int x() {
        return this.x;
    }
}
```

You can also request that `scalac` generates a Java-like getter for your `val` using the `@BeanProperty` or `@BooleanBeanProperty` annotation, which is nice if you want your class to be consumed by one of the many Java frameworks that use reflection to inspect JavaBean classes.

```scala
import scala.beans.BeanProperty

class Klass {
  // will cause getX() method to be generated
  @BeanProperty val x = 5

  // will cause isGood() method to be generated
  @BooleanBeanProperty val good = true
}
```

#### `lazy val` members

`lazy val`s are just like `val`s except that they are initialized lazily, when they're first referred to. The initialization logic is implemented in a thread-safe manner, so you can safely refer to a `lazy val` from multiple threads without worrying about seeing uninitialized value or running initialization code multiple times.

Thread safety is ensured by synchronization during initialization and volatile access (so, no unnecessary synchronization happens after the field has already been initialized).

#### `var` members

`var` members represent mutable fields. In bytecode, they are backed by a private field, a getter method (the same way as for `val`s) plus an additional *setter* method. The setter method uses peculiar name mangling - if your `var` is named `someVar`, its setter method will be `someVar_=`. This method is always used when someone assigns a new value to the `var`. Here's an example of Scala class with `var` member and a Java class that would compile to the same bytecode:

```scala
class Klass {
  var number: Int = 0
}
```

```java
public class Klass {
  private int number = 0;
  public number() { return this.number; }
  // number_$eq is a name-mangled version of number_=
  public number_$eq(int number) { this.number = number; }
}
```

The synthetic setter can be called directly, e.g.

```scala
val k = new Klass
// following two lines are equivalent
k.number = 10
k.number_=(10)
```

As with `val`s, you can use `@BeanProperty` and `@BooleanBeanProperty` to request generation of JavaBean-style getter and **setter** for your `var` member.

#### Manually defined accessors

If you have a `var`, you cannot affect how its accessor methods are implemented. However, you can define a getter and a setter without a `var` at all. For example:

```scala
class Klass {
  def number: Int = { /* get from somewhere */ }
  def number_=(n: Int): Unit = { /* set to somewhere */ }
}
```

If you define such a pair, you can use it as if it was a `var`:

```scala
val k = new Klass
k.number = 10
println(k.number)
k.number += 5
```

**NOTE**: `k.number = 10` is automatically translated to `k.number_=(10)` only if **both** getter and setter are declared. Just declaring the setter is not enough.

#### Inner classes

You can define a class inside another class (or object). Inner classes are bound to particular instance of its enclosing class (or object) and have access to its members.

Instances of an inner class can be created only when having an instance of the outer class. For example:

```scala
class Outer {
  val x = 5

  class Inner {
    // inner can access Outer's members
    println(x)
  }

  new Inner // will be bound to Outer's `this` instance
}

val outer = new Outer
new outer.Inner // will be bound to 'outer' object
```

#### Referring to outer instance

If an inner class or object needs to refer to its outer class instance, it can use the `Outer.this` syntax:

```scala
class Outer {
  class Inner {
    def printOuter(): Unit = println(s"My outer instance is ${Outer.this}")
  }
}
```

However, there is an additional syntax that allows us to give an alias to outer instance which can be used in inner instances:

```scala
class Outer { outer =>
  class Inner {
    def printOuter(): Unit = println(s"My outer instance is $outer")
  }
}
```
  
**NOTE**: this is one of the uses of more general syntax called *self-type annotation* but we won't discuss this in detail here

#### Path dependent types

Inner classes in Scala are also typed more strictly than in Java. Two instances of the same inner class bound to different instances of outer class have incompatible types with each other:

```scala
class Outer {
  class Inner
  def takeInner(inner: Inner): Unit = ???
}
```

then:

```
val o1 = new Outer
val o2 = new Outer

// error! - o1.Inner and o2.Inner are incompatible
o2.takeInner(new o1.Inner)
```

Also because of this strict behaviour of Scala, there's no such type as `Outer.Inner` (that's how you would refer to it in Java). If you write `o.Inner` then `o` must be a `val` (or `lazy val` or `object`) that is an instance of `Outer`. Such types are called *path-dependent types*.

If you want to express a type of inner class which doesn't care about the outer object, Scala lets you denote it as `Outer#Inner`, which means `Inner` instance of any `Outer` instance.

```scala
class Outer {
  class Inner
  def takeWhoeversInner(inner: Outer#Inner): Unit = ???
}
```

then:

```
val o1 = new Outer
val o2 = new Outer

// now fine
o2.takeWhoeversInner(new o1.Inner)
```

**NOTE**: the `Outer#Inner` syntax and more strict typing of inner classes applies also to Java classes when using them in Scala code.

#### Inner objects

Inner objects are also bound to its outer class instance. And since each object is a singleton, it means that there's exactly one inner object instance per outer class instance (i.e. a singleton in scope of its outer instance - there may be many instances overall).

It's important to remember that just like toplevel objects, inner objects are lazily initialized.

#### Inner classes and objects of objects

Objects can also have inner classes and objects - they work the same way as for outer classes, but of course typing constraints are no longer relevant because there's only one possible outer instance.

#### Note about `trait`s

Scala traits can also be inner to objects, classes and other traits and traits themselves can also have inner classes, objects and traits. This was not mentioned earlier because traits require a separate chapter of their own.

### Companion objects

**TODO**

### Inheritance and overriding

**TODO**

Implementing table:

```
   abstract def  val  var
concrete
def         yes  no   no 
val         yes  yes  no 
lazy val    yes  yes  no
var         yes  no   yes
object      yes  yes  no
```

Overriding table:

```
 overridden def  val  lazy val  var  object
overriding
def         yes  no   no        no   no
val         yes  yes  no        no   no
lazy val    yes  no   yes       no   no
var         pair no   no        no   no
object      yes  yes  no        no   no
```

### Type members

**TODO**

* constructors - primary, auxiliary
 * *local variables annoyance*
* generics, existentials
* methods, full syntax
* abstract members, overriding
* vals, lazy vals, vars, *bean properties*
* inner classes and objects
* no statics, companion objects, import annoyance
* inner classes and objects
 * self alias

