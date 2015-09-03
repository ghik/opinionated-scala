*Implicits* are an important, unique and controversial Scala feature. They are a low-level language mechanism that underlies many hihger-level features and are responsible for much of Scala's expresiveness, conciseness and type safety.

Awesome, but what are they, actually?

The name "implicits" by itself doesn't mean anything and encompasses two independent but closely related features *implicit parameters* and *implicit conversions*. You'll also hear about *implicit classes* or *implicit views* which are a form of implicit conversions.

### Implicit parameters

**TODO**

* last parameter list of a method or constructor may be marked as *implicit*, using the `implicit` keyword - this means that if we don't supply these parameters to a method call, the compiler will try to automatically find appropriate values
* only values marked by themselves as `implicit` are eligible to be passed as implicit parameters
* implicit `def`s that take parameters themselves are seen as implicit function values, unless the parameters of the `def` are themselves implicit - this would mean a chained implicit (an implicit which depends on other implicits)
* implicit parameters are themselves seen as implicit values inside the method body
* implicits are searched for in current scope or, when not found, in the *implicit scope* which consists of companion objects of classes and traits that occur in the type of the implicit, or their base classes and traits
* beware of shadowing!
* overloading resolution
* the type of an implicit should be explicit

Implicit parameter usages:
* static dependency injection, e.g. `ExecutionContext`
* additional type information, e.g. `ClassTag`
* additional type constraints, e.g. `<:<`
* type classes - later

Implicit conversions:
* as part of "implicit parameters" feature, the compiler looks for implicits when it needs something to pass as an implicit parameter
* as part of "implicit conversions" feature, the compiler also looks for implicit values of type `A => B` when it has value of type `A` in a context where value of type `B` is required
* implicit `def`s are implicit conversions by eta-expansion
* additionally, as part of "implicit conversions" feature, the compiler looks for implicit conversions which would allow it to call some otherwise unresolved member of some type - extensions methods
* `implicit class` syntax, value classes
* extension methods - actually more powerful than regular methods

Convenience implicit conversions - why not cool:
* we usually want the conversion to work in only one specific context, but it will pollute everything
* alternative: extension methods and "converters"
* extension methods and overloading
* implicit conversion to controlled type - magnet pattern example
* still annoying - implicit conversions don't chain
* refactor method into generic and implicit conversion stops working

Type classes:
* the idea in Haskell - `Monoid` example
* a way to do polymorphism
* scala encoding using implicits, ops implicit class
* `Ordering`, `Numeric`
* more powerful than inheritance
 * we can define typeclass instances for arbitrary types, not classes
 * we can define typeclass instances for types not defined by us
 * we can express complex rules for type class instances and dependencies between typeclasses
* `Foldable`, higher-kinds and ops