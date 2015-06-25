## Modularization and object orientation

### Packages and visibility

#### Files

In *Java*, every source file must contain at most one public class and the file must have the same name as that class. 

Scala has no such restriction. Scala source file may contain many classes, objects and traits and there are no restrictions in terms of their visibility. However, it is a good practice to retain the Java convention and put large classes in their dedicated source files. But since it's not uncommon for Scala classes to be defined in single line, it's good that the language allows keeping them together.

Also, if you recall *algebraic types*, you know that they rely on the `sealed` modifier which - when applied to a trait or class - requires that all direct subclasses must be defined in the same file. This way Scala uses files as containers for sealed class hierarchies.

#### Package nesting

We'll look back at *Java* once again. In Scala's predecessor, packages seem to have a hierarchical structure (e.g. `com.google`, `com.google.guava`). But it's mostly just their names that form some kind of hierarchy. Packages themselves don't nest, i.e. `com.google.guava` is not contained in `com.google` - it's a completely separate package. This means that full import is required when some classes from the two packages want to refer to each other. Also, classes in `com.google` have no access to package-protected definitions in `com.google.guava` and vice versa.

Scala by default behaves the same way, but it is possible to change it. Everything depends on how the package is declared at the beginning of `.scala` file. 

Let's look at an example. Suppose our project's source code is split into two primary packages, `com.software.theproject.core` and `com.software.theproject.gui`. First package contains the `com.software.theproject.core.Utils` class which is used class `com.software.theproject.gui.PrettyWindow` class from the other package. Here's how `PrettyWindow.scala` is defined:

```scala
package com.software.theproject.gui

import com.software.theproject.core.Utils

class PrettyWindow {
  Utils.failWith("Not implemented yet!")
}
```

In order to use `Utils`, `PrettyWindow` needs a full, explicit import. But in the snippet below, this is no longer the case:

```scala
package com.software.theproject
package gui

import core.Utils

class PrettyWindow {
  Utils.failWith("Not implemented yet!")
}
```

As you can see, we have split the package clause into two separate lines. Such construct is called *chained package clause*. `PrettyWindow` is still in the same package as before (`com.software.theproject.gui`) but now it can directly access anything from `com.software` without import. That's why we were able to shorten `import com.software.theproject.core.Utils` to just `import core.Utils` - the `core` package is visible directly as a member of `com.software`.

In other words, splitting the package clause into multiple lines allows us to see identifiers from all intermediate packages (each line of package clause represents "intermediate" package), not just from exactly the full package that our file is defined in. 

Almost all real-life projects in Java or Scala have their dedicated toplevel package, i.e. every source file in that project is (directly or not) contained in that package. For example, in our snippet it would be `com.software.theproject`. Common usage of chained package clauses is to split the package clause at that toplevel package, just like we did it in above snippet.

* no file naming restrictions
* nested packages
* access qualifiers, package-private and protected-private
* package objects

### Imports
* package imports
* relative imports
* local imports
* wildcard imports
* imports from objects and vals
* multi imports
* import aliases
* wildcard imports with exclusions

### Objects and classes

As we have already seen in many examples in this tutorial - classes in Scala are defined in a standard way - with the `class` keyword and instantiated with the `new` keyword:

```scala
// we can omit braces if class has empty body
class MyClass
// we can omit parens just like for methods with no params
new MyClass
```

#### Primary constructor

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

On bytecode level, Scala compiler will automatically turn such constructor parameters into class fields so that they can be accessed even after the primary constructor finished its execution (you can imagine them to be `private[this] val`s).

Primary constructors may also have access modifiers. For example:

```scala
class MyClass private(myName: String)
```

This syntax may look somewhat odd when the constructor has no params and even more odd when also the class has empty body but is otherwise perfectly valid:

```scala
// private primary constructor with no params on class with empty body
class MyClass private
```

You can easily turn primary constructor parameters to also be members of your class by adding the `val` keyword:

```scala
class MyClass(val myName: String)
// now myName is a constructor param but can also be accessed from outside!
new MyClass().myName
```

Such constructor-parameter-members can have all the modifiers that normal class members can - they can be `private`, `final`, etc.

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

#### Class members

Class members are divided into two groups - *term members* and *type members*.
Term members are members which represent values - `val`s, `lazy val`s, `var`s, `def`s and `object`s.
Type members are members which represent types - (inner) `class`es, `trait`s, `type` aliases and abstract `type`s.

* `def` members

These are just methods. We have already seen multiple examples of them.

* `val` members

When a member of some class is a `val`, it represents a field and a method at the same time. Every `val` member is backed by a field in bytecode, but every access to that field goes through an automatically generated getter - a method with the same name as the `val`.

For example, this class:

```scala
class Klass {
  val x = 5
}
```

yields bytecode equivalent to this Java class:

```java
public class Klass
    private int x = 5;
    public int x() {
        return this.x;
    }
}
```

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

* traits
 * comparison to Java 8 interfaces
 * init code, but no constr params
 * multiple inheritance
 * overriding and linearization
 * initialization order problems, early initialization
 * the `abstract override`
