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
* constructors - primary, auxiliary
* generics
* vals, lazy vals, vars, bean properties
* methods, full syntax
* abstract members
* no statics, companion objects