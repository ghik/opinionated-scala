# Handling data in functional way

In this chapter we're going to explore some basic ways of processing data in a functional way. Most of what we are going to show here has some orthogonal counterpart in object oriented programming. Scala tries to blend functional and object oriented programming in a single language but it is not necessarily a good idea to mix these two styles. That's why we're going to try to provide some guidelines on when to use functional and object oriented approach and make the distinction as clear as possible.

## Some principles of functional programming

### Immutability

Functional programming is a lot about ability to reason abstractly about programs. Therefore, in functional design, we like to put as much constraints on our data and code as possible while still retaining its ability to do what we want.

One of such essential constraints that functional programming leverages is **immutability**. When we're sure that our data structures are immutable, we can relieve ourselves from much pain of debugging and tracking possibly very unpredictable mutations in our program. Code that uses only immutable data is much easier to refactor, test, parallelize and sometimes even optimize for performance.

*Ok, but if all data is immutable then how do we, for example, update some map with a key-value pair?*

When working in immutable data structures, "modifying" does not mutate the data in place but rather creates completely new piece of data based on the original data. For example, putting a key-value pair to an immutable map actually creates a new map based on the original map with one more entry added to it.

*Ok, but doesn't this require a lot of copying and is bad for performance?*

Immutable data structures are indeed usually slower than mutable ones, but it's not as bad as it may seem. Implementations of immutable data structures use some very clever tricks to share as much data between consecutive instances as possible. This way they mostly avoid copying. For example, updating an immutable map in Scala with a key-value pair is still a nearly constant-time operation. Such data structures are often called *persistent data structures*. We'll talk more about immutable collections in a separate chapter. For now we just want to outline a general value of immutability.

Despite the fact that working with immutable data usually seems to be slower than with mutable data, there are often situations where it's the mutability that hurts performance.

Java is *very* bad at immutability. Working with immutable data in Java requires so much boilerplate that usually we just give up and introduce some mutable state. In particular, all standard Java collections are mutable. Or rather should we say - they can never be assumed to be immutable. The example below shows how mutability may hurt:

```java
public class Tweet {
    private List<String> hashTags;

    public void setHashTags(List<String> hashTags) {
        this.hashTags = new ArrayList<>(hashTags);
    }
}
```

As you can see, the setter always makes a defensive copy of the list it accepts as the argument. If it wants to be really safe, it really must do it, because we cannot be sure if some other code has access to this list and may mutate it outside of our control. This is a good example where it's the mutability that forces us to copy a lot of data. If the list was immutable, we wouldn't need it.

**Immutable data can be safely shared.** 

That property is especially useful when sharing between multiple threads. This relieves us from much pain stemming from handling concurrent changes of mutable state and tricky synchronization issues that come with it.

### Separation of code and data

Apart from immutability of data, you'll also see that in functional design, there is usually a very clear separation of code and data. Definitions of data types and functions that work on them are completely separate. You could say that in functional languages, the API of data types lies "outside" of them and can be different in various contexts.

Note that this is quite the opposite of what object-oriented programming is proposing. When we create a class, put some fields in it and some methods that work on these fields, we strongly tangle some data with some code.

We're not saying that any of the approaches is better than the another - each has its own advantages. Functional approach provides very clear separation of concerns while the object-oriented approach gives us nice encapsulation of data. However, Scala has the advantage over Java here because it allows us to use *both* ways while Java pretty much only supports the object-oriented approach. Of course, with great power comes great responsibility - having two significantly different methods available for us within a single language, we must be very careful not to create a lot of mess by mixing these approaches.

## Basic immutability and code-data separation in Scala

### Tail recursion

We say that a recursive function is *tail recursive* when the recursive call is the *very last* thing which that function does. Here's an example:

```scala
def printElementsFrom(arr: Array[String], idx: Int): Unit = {
  val len = arr.length
  if(idx < length) {
    println(arr(idx))
    printElementsFrom(arr, idx + 1)
  }
}
```

The function above prints elements of an array starting at index `idx`. In particular, we could use it to print all elements of an array by calling it with `idx == 0`. Note how the recursive invocation is the last thing the function does. When the recursive invocation finishes, its "parent" invocation has nothing more to do.

Scala is able to use this nice property to perform the so-called *tail call optimization*. In standard recursion, every recursive call of the function has its own local variables and other data on the program stack. If the recursion is very deep, we may get a `StackOverflowError` or at least use a lot of memory.

But with tail recursion, we can avoid this. Recursive call is the last thing the function does, so it no longer needs its local variables at that point. This way the recursive call can reuse stack space allocated by its "parent" invocation for its own local variables and prevent the stack from growing.

Note that this effectively makes our function work just like a loop - it simply iterates and prints every element of the array - all in small, constant amount of memory used. If you look into the bytecode, you'll see that the tail-recursive function is compiled to the same bytecode as plain `while` loop.

However, a `while` loop would require mutable variable. We don't have any of these in our tail-recursive function. We've just written a loop that works on completely immutable data and is as fast as a `while` loop!
Note also that this can be faster than using `arr.foreach(println)` - the `foreach` requires an anonymous function to be created which introduces runtime overhead.

As you can see, tail recursion is the most basic tool to retain immutability of data in our code. It can also be much more readable than plain old variable-mutating imperative code. We would actually recommend tail recursive function as a default option when having to write a loop. This is fairly easy also thanks to the fact that Scala has local functions.

#### The `@tailrec` annotation

Scala has a nice annotation that can help you avoid accidentally writing non-tail-recursive functions where you intended to use tail recursion. Simply annotate your method with `@tailrec` and if the compiler detects that your method is not tail-recursive, it will give you an error.

#### Indirect tail recursion and trampolining

**TODO**

### Immutable records - case classes

The most basic piece of compound data is a *record* - something that takes a bunch of values of various types, gives each of them a name and wraps them together into a single object. A record that represents a postal address (say city, zipcode, street name and building number) is a very simple example.

Of course, we care about immutability so our records will also be immutable - if we want to modify one of the fields inside a record, we actually create a new record that is a copy of the old one but with this single field changed. Also, it is completely natural to compare records for equality or use them as keys in maps. We would also expect them to have a nice string representation if there's a need to print them during e.g. debugging.

Java is absolutely *terrible* at defining immutable records. It requires a ridiculous amount of boilerplate to meet all the expectations listed above. For example, in order to create a two-field immutable record that holds name and surname, we would need to write a serious amount of code:

```java
import java.util.Objects;

public final class Person {
    private final String name;
    private final String surname;

    public Person(String name, String surname) {
        this.name = name;
        this.surname = surname;
    }

    public String getName() {
        return name;
    }

    public String getSurname() {
        return surname;
    }

    public Person withName(String newName) {
        return new Person(newName, surname);
    }

    public Person withSurname(String newSurname) {
        return new Person(name, newSurname);
    }

    @Override
    public boolean equals(Object other) {
        if(other instanceof Person) {
            Person otherPerson = (Person)other;
            return Objects.equals(name, otherPerson.name) &&
                   Objects.equals(surname, otherPerson.surname);
        } else {
            return false;
        }
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, surname);
    }

    @Override
    public String toString() {
        return "Person(" + name + ", " + surname + ")";
    }
}
```

This is a horrendous amount of boilerplate code. Here's a Scala equivalent:

```scala
case class Person(name: String, surname: String)
```

That's it. One line and we meet all the requirements stated earlier. That single line also provides some more features which Java simply has no equivalent of. We'll talk about them later. For now, let's explain what exactly is a *case* class.

Declaring some class as `case class` causes the following to happen:
* Companion object is automatically created for the class. It contains autogenerated `apply` method which allows us to create instances without the `new` keyword:

    ```scala
    val person = Person("Fred", "Kowalski")
    ```

    Actually, every companion object of a case class automatically extends one of the `Function` traits, depending on the number of fields. Our `Person` class has two fields of type `String` and so the `Person` object extends `Function2[String,String,Person]`, i.e. `(String, String) => Person`.

* Fields of the case class are automatically visible as public members:

    ```scala
    val person = Person("Fred", "Kowalski")
    println(s"Hi, ${person.name}!")
    ```

* The class has automatically generated `equals`, `hashCode` and `toString`.
* The class also has automatically generated `copy` method which allows us to make copies of instances with some of the fields changed:

    ```scala
    val person = Person("Fred", "Kowalski")
    val otherPerson = person.copy(surname = "Nowak")
    ```

  The `copy` method is especially useful when the class has a lot of fields.

* We can *pattern-match* against the class. We will soon explain in detail what that means. For the sake of completeness, we'll mention that this is allowed thanks to the automatically generated magic `unapply` method in the companion object. Don't worry about what that means now, though.
* Every case class also extends one of the Scala `Product` traits (`Product1` to `Product22`). This causes the class to have the `productArity`, `productElement`, `productIterator` and `productPrefix` methods. However, these are not very interesting, so we won't be elaborating on them.

### Pattern matching

Pattern matching is one of the flagship features of every functional programming language and one of the most powerful features of Scala.

Pattern matching is closely related to case classes. In previous section, we mentioned that we can pattern match *against* them. Now we're going to show what that means.

Pattern matching essentially allows us to check if some value has a specific type and shape (it "matches a particular pattern") and deconstruct that value according to that pattern at the same time (with concise syntax). Let's define a slightly more complex case class for personal data:

```scala
case class Address(city: String, zipcode: String, street: String, number: Int)
case class Person(name: String, surname: String, address: Address)
```

In the simplest form of pattern matching, we simply deconstruct an instance of `Person` by giving its parts some identifiers and doing something with them.

```scala
val person = Person(...)
person match {
  case Person(name, surname, address) =>
    println(s"Hello, $name $surname!")
}
```

In this case, we use fields of the `person` object to print a message. But as everything else in Scala, a pattern match is an expression that evaluates to some value. The above snippet could be rewritten as:

```scala
val message = person match {
  case Person(name, surname, address) =>
    s"Hello, $name, $surname"
}
println(message)
```

We have given the parts of `person` object identifiers `name`, `surname` and `address` - exactly the same names as in the definition of `Person` case class. This is natural in this case, but it's not required. We could have used any names we liked.

##### Wildcard patterns

You may have also noticed that we didn't use the `address` in our pattern match. In such case, we may completely omit the name for an unused value and replace it with *wildcard*:

```scala
val message = person match {
  case Person(name, surname, _) =>
    s"Hello, $name $surname"
}
```

##### Matching literal values

The above pattern match does not actually test any type or structure - we know that `person` is a `Person` so it cannot have different type or structure that our pattern expects. In other words, it's impossible for the above pattern match to fail.

But that's of course not always the case. We can require our `person` object to have more specific structure. For instance, we may require that the `name` is *exactly* "John" - by using a fixed literal value instead of an identifier that would match anything:

```scala
val message = person match {
  case Person("John", _, _) =>
    s"Hi, Johnny!"
}
```

We may also extract our literal to a constant:

```scala
val TheName = "John"
val message = person match {
  case Person(TheName, _, _) =>
    s"Hi, Johnny!"
}
```

However, be careful with this! There's a peculiar syntactic rule hidden in there. If we changed the name of our constant to `theName` (lowercase), it will no longer work. Instead of matching exactly the value inside `theName` constant, it would match *any* value and simply give it *identifier* `theName`. Such identifier would just shadow the constant that we defined:

```scala
val theName = "John"
val message = person match {
  // this will match anything, not just "John"
  // theName in the pattern is a completely new identifier which shadows the one outside
  case Person(theName, _, _) =>
    s"Hi, Johnny!"
}
```

Scala decides what to do solely based on whether the identifier starts with uppercase or lowercase letter. This may be very confusing if you're not aware of that detail.

It is generally recommended to use uppercase-starting names for external constants. However, as a last resort, you can put the name of your constant inside backticks. This will force the compiler back into treating the identifier as reference to a constant:

```scala
val theName = "John"
val message = person match {
  // this will match only "John" thanks to backticks
  case Person(`theName`, _, _) =>
    s"Hi, Johnny!"
}
```

##### When matching fails

Looking at previous example - what will happen if the name isn't "John"? In such case the pattern match will fail by throwing a `MatchError` (which, contrary to its name, is actually a `RuntimeException`). That's not something we're happy about. What we'd want is to do something special when the name is "John", but do something more generic when it isn't. We can easily achieve this by combining two previous patterns:

```scala
val message = person match {
  case Person("John", _, _) =>
    s"Hi, Johnny!"
  case Person(name, surname, _) =>
    s"Hello, $name $surname!"
}
```

When we have multiple cases in the pattern match, Scala will check one after another until it finds the first that matches. All other ones will be ignored, even if some of them would match, too. In other words, the order of cases in pattern matching is important. We generally put more specific cases above the more generic ones. The `MatchError` will be thrown only when none of the cases matches.

Note that this also means that we don't need to put any `break` statements at the end of each case, like we do in the `switch` statement in C or Java. Scala doesn't even have the `break` statement. Unlike in `switch` statement, there are also no problems with declaring local variables inside pattern matching cases. Each case body has its own scope.

##### Empty case bodies

If you recall the first chapter of this tutorial, you'll remember that there was a confusing pitfall with the `if` statements which don't have the `else` clause. The Scala compiler silently infers `else ()` in such situations, which may cause very confusing compilation errors.

Pattern matching has a similar pitfall - if you forget to implement the body of any of the cases by putting nothing after the `=>` sign, the compiler will silently insert `()` in there and you may run into exactly the same problems as with the `if` statement (e.g. the type of pattern match may be confusingly inferred as `Any`).

##### Alternatives

Imagine we want to handle "Johnny" in the same way as "John". In the most straightforward way, we would simply add one more case to the pattern match to do this:

```scala
val message = person match {
  case Person("John", _, _) =>
    s"Hi, Johnny!"
  case Person("Johnny", _, _) =>
    s"Hi, Johnny!"
  case Person(name, surname, _) =>
    s"Hello, $name $surname!"
}
```

However, we are repeating ourselves this way - we have two cases with the same body, which is not nice. Fortunately, we can use the `|` operator to combine two patterns into one - an alternative of its arguments:

```scala
val message = person match {
  case Person("John", _, _) | Person("Johnny", _, _) =>
    s"Hi, Johnny!"
  case Person(name, surname, _) =>
    s"Hello, $name $surname!"
}
```

Unfortunately, the above will not work if we wanted to give an identifier to the person's surname or address. For instance, this will fail with compilation error:

```scala
val message = person match {
  // error!
  case Person("John", surname, _) | Person("Johnny", surname, _) =>
    s"Hi, Johnny $surname!"
  case Person(name, surname, _) =>
    s"Hello, $name $surname!"
}
```

However, this example is simple enough so that we can redeem ourselves a bit. Notice that the two patterns that we combined with the `|` operator differ only by the name literal. We can use this to simplify our pattern:

```scala
val message = person match {
  case Person("John" | "Johnny", surname, _) =>
    s"Hi, Johnny $surname!"
  case Person(name, surname, _) =>
    s"Hello, $name $surname!"
}
```

The `|` operator can be used anywhere in the pattern.

##### Nesting patterns

We have deconstructed an instance of `Person` case class, but it also has another case class as one of its fields - the `Address`. So far, we ignored it, but what if we wanted to deconstruct it, too? We can do this with no problem - patterns can be arbitrarily nested and combined:

```scala
val message = person match {
  case Person(name, surname, Address(city, _, _, _)) =>
    s"Hello, $name $surname from $city!"
}
```

##### Bind operator

In above example, deconstruction of `Address` gives us nice syntax to access `city`. But what if we wanted to access both the `city` and the address itself? There's no identifier assigned to an address. Fortunately, we can give it one without destroying the pattern. To do that, we use the *bind* operator:

```scala
val message = person match {
  // we have identifiers for both the address and the city
  case Person(name, surname, addr@Address(city, _, _, _)) =>
    val formattedAddress = formatAddress(addr)
    s"Hello, $name $surname from $city! It seems that you live at $formattedAddress."
}
```

Bind operator can be used with any pattern on its right side. For example, it could be an alternative of patterns to which we give a common name.

##### `instanceof`-like patterns

So far our patterns only worked with case classes (to check for particular structure) or literal values (to check for equality). However, pattern matching can be used to perform arbitrary runtime type checking. This is similar to Java's `instanceof` operator.

For example, let's assume we just deserialized some object that came from a network and we don't know its type. We just know that it may be an `Int` or `String` or a `Boolean`. Pattern matching allows us to perform runtime tests for these types in concise syntax:

```scala
def handleInt(i: Int) = ...
def handleString(s: String) = ...
def handleBoolean(b: Boolean) = ...

val any: Any = getFromNetwork()
any match {
  case i: Int => handleInt(i)
  case s: String => handleString(s)
  case b: Boolean => handleBoolean(b)
  case _ => throw new Exception("Can't handle this value!")
}
```

Notice how each case does two things at once - it first checks whether matched value has particular type and if it has, it gives it a new identifier which is already aware of that type. This is something you would usually do in two separate steps in Java - first you would test the type with `instanceof` operator and then cast the value to that type.

The example above shows dedicated syntax (similar to type ascription) for `instanceof`-like patterns. However, runtime type check is also "hidden" inside every other pattern match. For example, nothing stops us from matching our completely untyped `any` value against a case class:

```scala
def handlePerson(name: String, surname: String) = ...

val any: Any = getFromNetwork()
any match {
  case Person(name, surname, _) => handlePerson(name, surname)
  ...
}
```

In this example, Scala will first test whether `any` is actually a `Person` and only then it will proceed to deconstruct it against the pattern.

**WARNING** Pattern matching used to perform runtime type checks is probably the easiest way to abuse its power. It gives us a nice syntax for doing an `instanceof` and type cast at the same time, but it should be avoided just as usage of `instanceof` is avoided in Java. Runtime type checks ("typecasing") are an enemy of type safety and nice abstractions. They go against ability to reason abstractly about code and thus make it much harder to change and maintain. Sometimes they're unavoidable (like when deserializing stuff from network), but having too much of them usually indicates bad design.

Runtime type checking with pattern matching is also limited in the same way as `instanceof` checks are limited in Java by the type erasure. That means you can't check e.g. if your object is a `List[String]` - you can only check if it's a `List[_]` (list of anything). Only the *class* information is retained in runtime, not the full type.

##### Guards

Every pattern matching case may be *guarded* by an arbitrary logical expression which can access deconstructed values. Here's how to do it:

```scala
val message = person match {
  case Person(name, surname, _) if name == surname =>
    s"Why did your parents give you the same name as your surname, $name?"
}
```

### Partial functions

Having a function `A => B` we usually assume that the function is safe to call on any argument of type `A` (or at least we would like to). However, this is not always the case - the function may throw an exception. For example:

```scala
val fun: String => Int =
  _.toInt
```

Calling `fun("abc")` will throw a `NumberFormatException`. We clearly see that the function is properly defined only for some of the possible values that can be technically passed to it. In this case we can pass any string value to `fun` but it will only return result for these strings which are correct representations of integer numbers.

Scala has a specialized function type which allows us to ask whether it is defined for given argument or not: `PartialFunction`.

`PartialFunction` inherits `Function1`, so it has a regular `apply` method, but it also has the `isDefinedAt` operation which allows us to ask whether that partial function is defined for particular value or not.

`PartialFunction` also exposes some convenience methods:

* `applyOrElse` - this is like `apply` but allows us to provide a fallback function that will be used if the partial function is not defined for particular value.
* `orElse` - combines two partial functions together into a single partial function. The second partial function serves as a fallback which is used when the first function is not defined for some value.
* `lift` - transforms a `PartialFunction[A,B]` into a `Function[A,Option[B]]`. An `Option[B]` may be one of two things - either `Some[B]` which contains the `B` value returned by original partial function or `None` if the partial function was not defined for particular value. We'll talk about the `Option` later in more detail.

Scala also provides a dedicated syntax for defining `PartialFunction`s. It's based on the pattern matching syntax. Here's how we would redefine our string-to-integer parsing function as a partial function:

```scala
val fun: PartialFunction[String,Int] = {
  // let's ignore negative numbers for simplicity
  case str if str.forall(_.isDigit) => 
    str.toInt
}
```

As you can see, the partial function body is like a pattern match, but without the `<expr> match` part at the beginning. You can think of it as if the argument of partial function was pattern-matched against our list of `case` definitions. If none of the cases match, the partial function will say that it's not defined for that argument.

Now, if we want to use `0` as a default value for the result (in case the string is not a proper integer representation), we could do it like this:

```scala
val string: String = ...
val parsed: Int = fun.applyOrElse(string, _ => 0)
```

Note that `PartialFunction` is still a `Function` so you can use the partial function syntax when using various higher-order functions. For example:

```scala
case class Person(name: String, surname: String)
val listOfPersons: List[Person] = ...
val surnamesOfJohnnyOrEmpty = listOfPersons.map {
  case Person("Johnny" | "John", surname) => surname
  case _ => ""
}
```

However, be careful - the `map` method that we have used is not aware that its argument is a `PartialFunction` and it's unable to check whether it is defined or not for particular argument. Therefore, if you pass partial functions where regular functions are expected, make sure that the PF is *always* defined. This way it's not going to be "partial" anymore, but the point is that we can still use the nice pattern matching like syntax.

#### Partial function syntax for multi argument functions

Sometimes it's also worth to know that Scala will also accept partial function syntax where multi-argument functions are expected. For example:

```scala
def takeFunctionTwo(fun: (Int,String) => String) = ???
takeFunctionTwo {
  case (int, string) => /* do something */
}
```

### Pattern assignment

Let's look at this simple code:

```scala
val x = someVeryLongExpression()
println(x)
```

Although there's no real point in writing it like this, technically this code does the same thing as:

```scala
someVeryLongExpression() match {
  case x => println(x)
}
```

Now, let's assume the pattern match from above is a little more complicated, e.g.

```scala
somePerson() match {
  case Person(name, surname) => println(s"Hi, $name $surname!")
}
```

Now the pattern is not as trivial as before - we extract two values (name and surname) from a case class. However, we can still go back to the previous syntax:

```scala
val Person(name, surname) = somePerson()
println(s"Hi, $name $surname!")
```

This is a very handy syntactic feature of Scala. You can use pretty much any pattern on the left side of an assignment. However, you have to be careful - if the value on the right side of assignment does not match the pattern, a `MatchError` will be thrown.

### Extractors

**TODO**

### Working with tuples

**TODO**

### Algebraic data types

**TODO**

#### Common ADTs from standard library

**TODO**

* `List`
* `Option`
* `Either`
* `Try`

### For comprehensions
