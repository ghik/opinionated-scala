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

Pattern matching essentially allows us to check if some valeu has a specific type and shape (it "matches a particular pattern") and deconstruct that value according to that pattern at the same time (with concise syntax). Let's define a slightly more complex case class for personal data:

```scala
case class Address(city: String, zipcode: String, street: String, number: Int)
case class Person(name: String, surname: String, address: Address)
```

Pattern matching
* case class matching
* nested patterns
* ignoring with underscore
* guards
* matching literals
* bind operator
* instanceof patterns
* multi-value assignments
* partial functions
* patterns in for comprehensions
* extractors?

Standard case classes:
* tuples

Algebraic data types:
* show in Haskell
* the same in Scala
* `sealed` trait
* about pattern matching ADTs and safety
* separation of code and data

Standard types:
* `List`
* `Option`
* `Either`
* `Try`
