# Dissection of Hello World

```scala
object Main {
  def main(args: Array[String]): Unit = {
    println("Hello")
  }
}
```

## Objects

Scala's `object` creates a singleton. In above example, the identifier `Main` refers to the singleton instance itself, not to its class. This means that `Main` can be used as a regular value, e.g. it can be passed as an argument to some function.

## Method definition

Our singleton class has one method called `main`. As we can see, method definition syntax differs from Java:
* method declaration or definition is denoted by the `def` keyword, followed by name of the method
* arguments are declared with `name: Type` syntax
* formal argument list is followed by return type declaration, in our example it is the `: Unit` part
* return type declaration is followed by `=` sign and a method body

### The `Unit` type

Scala's `Unit` type is roughly equivalent to Java's `void`, i.e. it is used to denote methods that do not return any meaningful value. Scala methods that return `Unit` are seen by Java as methods with `void` return type and vice versa - Java methods with `void` return type as seen by Scala as methods returning `Unit`. 

However, there is an important difference between Java's `void` and Scala's `Unit`: the `Unit` type actually *has* a value. That value is called "unit" and is denoted in Scala as `()`. Every method which has `Unit` return type actually returns `()`. So you could theoretically create a (useless) local variable of type `Unit` and assign something to it, e.g.

```scala
val foo: Unit = ()
val bar: Unit = println("stuff")
```

I haven't shown how to declare local variables yet, but I hope the above example is self-explanatory. We also see that the `println` method has `Unit` return type.

## Method body

The body of our `main` method looks mostly similar to how you would do it in Java - we have a block with some code inside. The only minor difference is that the `=` sign is required before the body.
However, the apparent similarity is a bit coincidental in our example.

### Blocks

Blocks, denoted by curly braces, in Scala look similar to blocks in Java, but are treated a bit differently. Java blocks are strictly pieces of *code* - that is, you can't assign a block to a variable or pass a block as a method parameter. In Scala, however, this is possible, because blocks are *expressions*, i.e. they evaluate to some value.

The value "returned" from a block is the value of last expression in that block. For example:

```scala
{
  println("something")
  handleStuff()
  42
}
```

The block above will evaluate to `42` and therefore, can be assigned to variable, passed as argument, etc.

### Method body is an expression

In general, method definition expects an *expression* after the `=` sign. The reason why we were able to put a block as a body of `main` is because a block is also an expression. But since our block has only one statement inside, we could shorten our `main` method definition:

```scala
def main(args: Array[String]): Unit = println("Hello")
```

Note that this is possible also thanks to the fact that `println` actually returns a value and can be "assigned" as a return expression of a method.

All of this also explains why the `=` sign is used. It more clearly denotes the fact that method body is an expression which always evaluates to something rather than a block of imperative code which may optionally return a value.

## Type inference

Scala is able to tell the return type of a method based on its body. For our `main` method, Scala compiler is able to deduce that since `println` returns `Unit`, then `main` also returns `Unit`. Thanks to that, we can omit the return type declaration and make the code even shorter:

```scala
def main(args: Array[String]) = println("Hello")
```

Type inference is one of the key features of Scala that allows writing concise code. As we will see later, it works in many other contexts, not just in method definitions.

## Procedure syntax

There is also one more syntax variant in which we could write our `main` method. It looks like this:

```scala
def main(args: Array[String]) {
  println("Hello")
}
```

We have omitted return type declaration and the `=` sign and put our method body in a block. Such syntax can be used for methods which return `Unit` (procedures). However, it is not recommended and will probably be removed from future versions of scala. We present it here for the sake of completeness - if you're going to read existing Scala code, you'll probably see this syntax frequently.

## Parameter list

We have declared our `main` method to take a single parameter named `args` which is an array of strings. There are a few differences from Java that are important here:
* Obviously, parameter declaration syntax is different with type coming after the name
* Arrays in Java are treated specially, but in Scala `Array` is simply a class that takes a type parameter, just like e.g. `java.util.List[String]`. Scala uses the square brackets `[]` to denote type parameters, as opposed to Java which uses angle brackets `<>`.

## Semicolons

You may have noticed that our `println("Hello")` call does not end with a semicolon. Semicolons are optional in Scala. They are typically used only to explicitly separate two statements inside a block when they are on the same line, e.g. `{ println("Hello"); println("World") }`.

## The `Predef` object

You may recall that in Java, there is no way to define "global" methods. Every method must be a member of some class. In Scala, we have the same rule - every method must come from a class or object. But you may have also noticed that we called our `println` method like it was global.

The `println` method actually comes from an object `scala.Predef` in the Scala standard library which contains some basic utilities like console operations. This object is treated specially by the Scala compiler - all its members are automatically visible everywhere. That is why we were able to call `println` directly, without having to write `Predef.println` or `scala.Predef.println`.

## Runtime

If you're not using an IDE or build tool (which you definitely should), you can compile our example by saving it in a `Main.scala` file (file name is arbitrary) and invoking `scalac Main.scala`. We will dig a bit into the bytecode to see how Scala compiler encodes `object`s.

You will see that two classfiles have been generated: `Main.class` and `Main$.class`. The `Main$` is the class that actually implements the singleton. Let's see what's inside it:

```
$ javap -private -c Main\$
Compiled from "Main.scala"
public final class Main$ {
  public static final Main$ MODULE$;

  public static {};                                                                                                                                                                               
    Code:                                                                                                                                                                                         
       0: new           #2                  // class Main$                                                                                                                                        
       3: invokespecial #12                 // Method "<init>":()V                                                                                                                                
       6: return                                                                                                                                                                                  
                                                                                                                                                                                                  
  public void main(java.lang.String[]);
    Code:
       0: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
       3: ldc           #21                 // String Hello
       5: invokevirtual #25                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
       8: return

  private Main$();
    Code:
       0: aload_0
       1: invokespecial #29                 // Method java/lang/Object."<init>":()V
       4: aload_0
       5: putstatic     #31                 // Field MODULE$:LMain$;
       8: return
}
```

We can see that:
* The static initializer of `Main$` invokes constructor.
* The constructor assigns itself to the `MODULE$` static field. This is the singleton instance of our class.
* The `main` method is a regular method, not a static one.

However, there is also a second class generated - `Main`. Let's see what's inside.

```
$ javap -private -c Main
Compiled from "Main.scala"
public final class Main {
  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #16                 // Field Main$.MODULE$:LMain$;
       3: aload_0
       4: invokevirtual #18                 // Method Main$.main:([Ljava/lang/String;)V
       7: return
}
```

We can see that the `Main` class also contains the `main` method. However, this one is static and simply forwards the call to `Main$.MODULE$.main`. Thanks to this static forwarder generated by `scalac`, we can run our program simply by writing:

```
$ scala Main
Hello
```
