## Packages and imports

### Files

In *Java*, every source file must contain at most one public class and the file must have the same name as that class. 

Scala has no such restriction. Scala source file may contain many classes, objects and traits and there are no restrictions in terms of their visibility. However, it is a good practice to retain the Java convention and put large classes in their dedicated source files. But since it's not uncommon for Scala classes to be defined in single line, it's good that the language allows keeping them together.

Also, if you recall *algebraic types*, you know that they rely on the `sealed` modifier which - when applied to a trait or class - requires that all direct subclasses must be defined in the same file. This way Scala uses files as containers for sealed class hierarchies.

### Package nesting

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

### Multiple packages in single file

As we already mentioned, Scala allows you to put many classes in the same file. But it goes even further with taking down file restrictions - you can define contents of multiple *packages* in a single file. For example:

```scala
package com.software.theproject

package utils {
  // contents of package `com.software.theproject.utils`

  package subutils {
    // contents of package `com.software.theproject.utils.subutils`
  }
}

package core {
  // contents of package `com.software.theproject.core`
}
```

### Packages vs directories

Standard Java convention says that source files should be placed in a directory that corresponds to package name. This is the same in Scala, but gets a bit more complicated for source files with multiple packages. In such case it seems natural to put the file in the package that contains every other subpackage defined in that file. For example, the file from previous section would be put into `com/software/theproject` directory.

### Package objects

In general, packages may contain only classes, objects, traits and other packages. It's impossible to put a method or field directly into a package.

However, Scala has a construct that somewhat simulates such situation - *package objects*.

Here's a sample definition of package object associated with the `com.software.theproject` package. The file is named `package.scala` and is in the `com/software/theproject` directory.

```scala
package com.software

package object theproject {
  def utilFunction(utilParam: String): Int = ???

  val Constant = "constant"
}
```

A package object is technically just an object named `` `package` `` (you can actually refer to it in code using ``com.software.theproject.`package` ``), but is treated a bit specially by the compiler.

All members of the package object are seen as if they are members of the package itself. For example, the `Constant` defined in above snippet can be accessed with `com.software.theproject.Constant`. Of course, you can also import it in the same way as a class would be imported.

Additionally, remember that package nesting and chained package clauses can make members of package automatically visible. This applies in the same way to package object members. For example:

```scala
package com.software.theproject
package core

object CoreUtil {
  // reference to `Constant` needs no import or qualification
  val ConstantUpper = Constant.toUpperCase
}
```

## Imports

Scala introduces a lot of new features to `import` constructs as compared to Java. We'll try to walk through them in this section.

#### Regular package member imports

The most common type of import you'll encounter in Java (and probably in Scala, too) is an `import` clause at the beginning of the file (just after `package` clause or clauses) which imports some class or interface (trait in Scala) from a package.

```scala
package com.software.theproject
package core

import java.util.List
import java.util.ArrayList
```

This looks the same in Java and Scala, except that you don't need semicolons.

#### Importing a package

Here's a first improvement that Scala introduces over Java: you can import a package *itself* and then refer to it just with its name:

```scala
package com.software.theproject
package core

import java.io

object CoreUtil {
  def newStringWriter = new io.StringWriter
}
```

#### Relative imports

You can also import relatively, i.e. from things that are already visible, e.g. thanks to some previous imports.

```scala
package com.software.theproject
package core

import java.io
import io.StringWriter

object CoreUtil {
  def newStringWriter = new StringWriter
}
```

#### Local imports

Imports can be used in arbitrary places in your source files, not just at the beginning of the file:

```scala
package com.software.theproject
package core

import java.io

object CoreUtil {
  import io.StringWriter // this import is visible only inside the `CoreUtil` object
  def newStringWriter = new StringWriter

  def newStringReader(str: Striong) = {
    import io.StringReader // this import is visible only inside the `newStringReader` method
    new StringReader(str)
  }
}
```

Local imports are very nice tool to minimize the impact of your imports and protect yourself from accidentally polluting too wide namespaces with imports that you need only in small, local context. That's especially important when using *wildcard* imports (see later).

#### Imports from values

Obviously, you can import things from packages and objects. But you can also import from `val`s and `lazy val`s:

```scala
package com.software.theproject
package core

import java.util.ArrayList

object CoreUtil {
  def createList(a: Int, b: Int, c: Int): ArrayList[Int] = {
    val result = new ArrayList[Int]
    import result.add
    add(a)
    add(b)
    add(c)
    result
  }
}
```

Imports from packages, objects and `val`s can be arbitrarily chained. For example if package `p` has `object x` which has a member `val y` which has a method `meth`, you can directly `import p.x.y.meth`.

#### Multi imports

You can import multiple identifiers in a single line:

```scala
import java.io.{StringWriter, StringReader}
```

#### Import aliases

You can import an identifier under a different name.

This is useful for example when you want to concisely refer to contents of some package or object, but you don't want to import everything from it:

```scala
import java.{lang => jl}

object Main {
  def takeJavaLong(n: jl.Long): Unit = ???
}
```

This is especially useful when direct import would introduce a name conflict (e.g. `Long` can be `scala.Long` or `java.lang.Long`).

#### Wildcard imports

Finally, you can import *all* members of some package, object, `val, etc. by using wildcard import:

```scala
import java.io._
```

Be careful with wildcard imports - you can easily write unmaintainable code with them. A good practice is to limit the usage of wildcard imports to local imports (e.g import all constants of some Java enum in a local code block).

You can join wildcard imports with import aliases to import everything, but some of the things under a different name:

```scala
// import everything from java.io, but StringReader under an alias SR
import java.io.{StringReader => SR, _}
```

There's also a special syntax that allows you to import everything *except* some things, e.g.

```scala
// import everything from java.io except StringWriter and StringReader
import java.io.{StringReader => _, StringWriter => _, _}```