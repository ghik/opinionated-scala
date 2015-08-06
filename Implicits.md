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
* implicit values of type `A => B`
* remember that `implicit def`s are lifted to functions (*eta expansion*)
* when some types don't match, the compiler will try to use implicit conversions
* the compiler may also use implicit conversion when it looks for a member that doesn't exist in the original type
* implicit views, implicit classes, extension methods