---
layout: post
title: From "Hello, world!" to "Hello, monad!" (part III/III)
date: 2017-05-30 09:57:39.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded DSLs
- functional programming
tags:
- Free
- Monads
- Syntactic sugar
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '43713691'
  geo_public: '0'
  _publicize_job_id: '5586176996'
author: Juan Manuel Serrano
permalink: "/2017/05/30/from-hello-world-to-hello-monad-part-iiiiii/"
---
In the <a href="http://blog.hablapps.com/2016/01/22/from-hello-world-to-hello-monad-part-i/">first</a> part of this series, we saw how we can write the business logic of our applications as pure functions that return programs written in a custom domain-specific language (DSL). We also showed in <a href="https://blog.hablapps.com/2017/01/09/from-hello-world-to-hello-monad-part-iiiii/">part II</a> that no matter how complex our business logic is, we can always craft a DSL to express our intent. All this was illustrated using the "Fibonacci" example of purely functional programming, namely *IO programs*. We reproduce bellow the resulting design of the IO DSL and a sample IO program:

```scala
  // IO DSL

  sealed trait IOProgram[A]
  case class Single[A](e: IOProgram.Effect[A])
    extends IOProgram[A]
  case class Sequence[A, B](p1: IOProgram[A],
    p2: A =&gt; IOProgram[B]) extends IOProgram[B]
  case class Value[A](a: A) extends IOProgram[A]

  object IOProgram{
    sealed trait Effect[A]
    case class Write(s: String) extends Effect[Unit]
    case object Read extends Effect[String]
  }

  // Sample IO program

  def echo(): IOProgram[String] =
    Sequence(Single(Read()), (msg: String) =&gt;
      Sequence(Write(msg), (_ : Unit) =&gt;
        Value(msg)))
```

However, while this design is essentially correct from the point of view of the functional requirements of our little application, and from the point of view of illustrating the essence of functional programming, there are two major flaws concerning two important non-functional guarantees: readability and modularity. Let's start from the first one!

*Note: you can find the code for this post in this <a href="https://github.com/hablapps/gist/blob/master/src/test/scala/hello-monads/partIII.scala">repo</a>. *

## More sugar!
What's the problem with the little `echo` function we came up with? Well, this function being pure has an essential advantage: it simply declares <i>what</i> has to be done, and the task of actually executing those programs in any way we want is delegated to another part of the application - the interpreter. Thus, we could run our `echo()` IO program using the `println` and `readLine` methods of the `Console`; or using an asynchronous library using `Future` values; or test it without the need of mocking libraries with the help of custom state transformers in a type-safe way. Great, great, great! But ... who would ever want to write our pure functions using that syntax? We have to admit that the readability of our little program is poor ... to say the least. Let's fix it!

### Smart constructors for atomic programs
We start by adding some lifting methods that allow us to use *IO instructions* as if they were *programs *already:

```scala
object IOProgram {
  object Syntax{
    val read(): IOProgram[String] =
      Single(Read)
    def write(msg: String): IOProgram[Unit] =
      Single(Write(msg))
  }
}
```

### Smart constructors for complex programs
Next, let's introduce some smart constructors for sequencing programs. We will named them `flatMap` and `map` -- for reasons that will become clear very soon. As you can see in the following implementation, `flatMap` simply allow us to write sequential programs using an infix notation; and `map` allows us to write a special type of sequential program: one which runs some program, transforms its result using a given function, and then simply returns that transformed output.

```scala
sealed trait IOProgram[A]{
  def flatMap[B](f: A =&gt; IOProgram[B]): IOProgram[B] =
    Sequence(this, f)
  def map[B](f: A =&gt; B): IOProgram[B] =
    flatMap(f andThen Value.apply)
}
```

Using all these smart constructors we can already write our program in a more concise style:

```scala
import IOProgram.Syntax._

def echo: IOProgram[String] =
  read() flatMap { msg =&gt;
    write(msg) map { _ =&gt; msg }
  }
```

### Using for-comprehensions
We may agree that the above version using smart constructors represents an improvement, but, admittedly, it's far from the conciseness and readability of the initial impure version:

```scala
def echo(): String = {
  val msg: String = readLine
  println(msg)
  msg
}
```

For one thing at least: in case that our program consists of a long sequence of multiple subprograms, we will be forced to write a long sequence of nested indented `flatMap`s. But we can avoid this already using so-called *for-comprehensions*! This is a Scala feature which parallels Haskell's *do* notation and F#'s *computation expressions*. In all of these cases, the purpose is being able to write sequential programs more easily. Our little example can be written now as follows:

```scala
import IOProgram.Syntax._

def echo(): IOProgram[String] = for{
  msg &lt;- read()
  _ &lt;- write(msg)
} yield msg
```

For-comprehensions are desugared by the Scala compiler into a sequence of `flatMap`s and a last `map` expression. So, the above program and the `flatMap`-based program written in the last section are essentially identical.

## Hello, Monad!
Let's deal now with the second of our problems: the one concerning modularity. What's the problem with the little DSL to write IO programs we came up with? Basically, the problem is that, approximately, half of this data type is not related to input-output at all. Indeed, if we were to write a different DSL to write imperative programs dealing with file system effects (e.g. reading the content from some file, renaming it, etc.), we would almost write line by line half of its definition:

```scala
sealed trait FileSystemProgram[A]
case class Single[A](e: FileSystemProgram.Effect[A])
  extends FileSystemProgram[A]
case class Sequence[A, B](p1: FileSystemProgram[A],
  p2: A =&gt; FileSystemProgram[B]) extends FileSystemProgram[B]
case class Value[A](a: A) extends FileSystemProgram[A]

object FileSystemProgram{
  sealed abstract class Effect[_]
  case class ReadFile(path: String) extends Effect[String]
  case class DeleteFile(path: String) extends Effect[Unit]
  case class WriteFile(path: String, content: String)
    extends Effect[Unit]
}
```

The only remarkable change is related to the kinds of effects we are dealing with now: file system effects instead of IO effects. The definition of the DSL itself simply varies in the reference to the new kind of effect. This amount of redundancy is a clear signal of a lack of modularity. What we need is a generic data type that accounts for the common imperative features of both DSLs. We can try it as follows:

```scala
sealed trait ImperativeProgram[Effect[_],A]{
  def flatMap[B](f: A =&gt; ImperativeProgram[Effect,B]) =
    Sequence(this, f)
  def map[B](f: A =&gt; B) =
    flatMap(f andThen Value.apply)
}
case class Single[Effect[_],A](e: Effect[A])
  extends ImperativeProgram[Effect,A]
case class Sequence[Effect[_],A, B](
  p1: ImperativeProgram[Effect,A],
  p2: A =&gt; ImperativeProgram[Effect,B])
  extends ImperativeProgram[Effect,B]
case class Value[Effect[_],A](a: A)
  extends ImperativeProgram[Effect,A]
```

Note how the `Single` variant of the DSL now refers to a (type constructor) parameter `Effect[_]`. We can now reuse the `ImperativeProgram` generic DSL in a modular definition of our DSLs for IO and file system effects:

```scala
type IOProgram[A] =
  ImperativeProgram[IOProgram.Effect, A]

type FileSystemProgram[A] =
  ImperativeProgram[FileSystemProgram.Effect, A]
```

This `ImperativeProgram` generic DSL seems pretty powerful: indeed, it encodes the essence of imperative DSLs, and it is actually commonly known through a much more popular name: *Free Monad*! The definitions of `Free` that you will find in professional libraries such as <a href="https://github.com/typelevel/cats/blob/master/free/src/main/scala/cats/free/Free.scala#L13" target="_blank" rel="noopener noreferrer">cats</a>, <a href="https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Free.scala#L102" target="_blank" rel="noopener noreferrer">scalaz</a> or <a href="https://github.com/atnos-org/eff/blob/master/shared/src/main/scala/org/atnos/eff/Eff.scala#L50" target="_blank" rel="noopener noreferrer">eff</a> are not quite the same as the one obtained in this post, which is quite inefficient both in time and space (not to mention further modularity problems when combining different types of effects); but, the essence of free monads, namely, being able to define imperative programs given *any* type of effects represented by some type constructor is there. This substantially reduces the effort of defining an imperative DSL: first, program definition will collapse into a single type alias; second, we will get the `flatMap` and `map` operators for free; and, similarly, although not shown in this post, we will also be able to simplify the definition of monadic interpreters (those that translate the given free program into a specific monadic data type, such as a state transformation, asynchronous computation, etc.), amongst many other goodies.

## Conclusion: modularity all the way down!
We may say that the essence of functional programming is *modularity*. Indeed, the defining feature of functional programming, namely *pure functions*, is an application of this design principle: they let us compose our application out of two kinds of modules: pure functions themselves that declare what has to be done, and interpreters that specify a particular way of doing it. In particular, interpreters may behave as translators, so that the resulting interpretations are programs written in a lower-level DSL, that also need to be interpreted. Eventually, we will reach the "bare metal" and the interpreters will actually bring the effects into the real world (i.e. something will be written in the screen, a file will be read, a web service will be called, etc.).

But besides pure functions, functional programming is full of many additional modularity techniques: parametric polymorphism, type classes, higher-order functions, lazy evaluation, datatype generics, etc. All these techniques, which were first conceived in the functional programming community, basically aim at allowing us to write programs with extra levels of modularity. We saw an example in this post: instead of defining imperative DSLs for implementing Input/Output and File System programs in a monolithic way, we were able to abstract away their differences and package their common part in a super reusable definition: namely, the generic imperative DSL represented by the Free monad. How did we do that? Basically, using parametric polymorphism (higher-kinds generics, in particular), and generalised algebraic data types (GADTs). But functional programming is so rich in abstractions and modularity techniques, that we may have even achieved a similar modular result using type classes instead of GADTs (in a style known as *finally tagless*). And this is actually what we will see in our next post. Stay tuned!

