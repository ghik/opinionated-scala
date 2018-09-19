# Scala macros

Macros are a very controversial feature of Scala. They have been introduced to the language back in 2013 in
version 2.10 as an experimental feature because of their tight coupling to the architecture of Scala compiler itself
with many of its low-level technical details, quirks, limitations and accidental complexity. For this reason, they
are rather hard to use because to write macros correctly and safely requires knowledge of some rather non-obvious,
not very well documented internals regarding how Scala programs are represented and processed in the compiler.

Macros have never left their experimental state and in theory they have never become a fully supported language feature.
However, they turned out to be very powerful and useful and since their inception they have become a basis for many
popular, foundational libraries. [Shapeless](https://github.com/milessabin/shapeless) is an example of far-fetched
exploitation of macros that practially makes _magic_ possible in Scala. For this reason, macros have _de facto_ become
a standard part of the language.

## Types of macros

Macros, in their most general sense are _transformations_ that a programmer can define to be done _during compilation_
on some parts of the program. In other words, macros are a _metaprogramming_ facility. Macros can also _generate_ code
 - not only as a transformation of other pieces of code but also based on compile-time reflection on types defined anywhere
 in the program. In practice, this might mean transforming/generating an expression but also an entire class/object definition
 or even a type (e.g. one being inferred by the compiler somewhere).
 
 ## `def` macros
 
 `def` macros are the most important macro flavor. They are macros which are visible to the type system as if they were
 regular methods. However, rather than in runtime, they are invoked during compilation. Each macro invocation is responsible
 for generating a piece of code that will replace the macro invocation and ultimately end up being compiled to bytecode.
 
 **TODO**
 * show an example of `def` macro
 * macro definition, implementation and invocation, separate compilation
 * introduce `Context`, `blackbox` at this point
 * quick explanation of `c.Tree` - note that it must be **typechecked** before being passed to macro
 * recursive macro expansion
 * macro bundles
 * blackbox and whitebox macros - IDE (not)friendliness
 
