---
layout: post
title: From “Hello, world!” to “Hello, monad!” (Part II/III)
date: 2017-01-09 17:49:35.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Embedded DSLs
- functional programming
- Scala
tags: []
meta:
  _wpcom_is_markdown: '1'
  _thumbnail_id: '65'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '572276751'
author: Javier Fuentes
permalink: "/2017/01/09/from-hello-world-to-hello-monad-part-iiiii/"
---
In the <a href="http://blog.hablapps.com/2016/01/22/from-hello-world-to-hello-monad-part-i/">first part</a> of this series, we set forth the essence of functional programming, namely, *being declarative*. This was illustrated with the ubiquitous "Hello, world!" example, a ridiculously simple program which, nonetheless, allowed us to introduce the major concepts involved in purely functional programming: declarative functions, languages, and interpreters. It's time now to show that this approach actually scales up for larger and more complex programs.

In the following paragraphs, you'll find a series of impure IO programs that represent purification challenges. Compare to the simple "Hello, world!" program, these new programs feature additional IO instructions, and an increasingly complex control flow structure. These extra levels of complexity will force us to enrich the simple `IOProgram` DSL and interpreter that we created to purify the simple "Hello, world!" program. For easy of reference, we reproduce that initial purification bellow:

```scala
object HelloWorld{
  /* Impure program */
  def helloWorld: String =
    println(&quot;Hello, world!&quot;)

  /* Functional solution */
  object Fun {
    // Language
    type IOProgram = Print
    case class Print(msg: String)

    // Pure function
    def pureHello(): IOProgram =
      Print(&quot;Hello, world!&quot;)

    // Interpreter
    def run(program: IOProgram): Unit =
      program match {
        case Print(msg) =&gt; println(msg)
      }

    // Impure program (modularised)
    def hello() = run(pureHello())
  }
}
```

## Say what?
Our first challenge consists in purifying the following impure program:

```scala
def sayWhat: String =
  readLine
```

Similarly to the "Hello, world!" program, the "sayWhat" program consists of a single IO instruction. In this case, when the program is run it will immediately block until we type something in the console. Then, it will return the string typed, rather than `Unit`:

```scala
scala&gt; SayWhat.sayWhat
(type &quot;something&quot;)
res0: String = &quot;something&quot;
```

In order to purify any program we have to return pure values that represent a description of the logic we want to accomplish. In order to describe this logic, we use the IOProgram DSL, but the current version of this DSL does only offer a `Write` instruction, so we have to extend it with a new `Read` command:

```scala
type IOProgram[A] = IOEffect[A]

sealed trait IOEffect[A]
case class Write(msg: String) extends IOEffect[Unit]
case object Read extends IOEffect[String]
```

There are several things going on here:


* We chose to use an ADT (Algebraic Data Type) to represent our instructions. We created an *effect Language* to perform IO operations, composed by two instructions, one to read from and the other to write to the console. And our programs consist of either one of these instructions.
* The new `Read` instruction is a `case object` because it is a singleton instance; there can only exist one and only one instance of `Read` as it has no arguments.
* Another thing to point out is that we parameterized our `IOEffect` and `IOProgram` ADTs. We did that because we need to store the return type of our instructions somewhere in order to be able to implement the interpreter. So in this case we use a *phantom type* to carry that information over. Thus, the `IOEffect` algebraic data type is what is known as a Generalised Algebraic Data type (GADT).

We got it. Now we can express the impure "sayWhat" program in a pure fashion as follows:

```scala
def pureSayWhat: IOProgram[String] = Read
```

As simple as that. But things become more complicated when we try to update our interpreter:

```scala
def run(program: IOProgram): ??? = ...
```

Now our programs are parameterized so we need to change the signature a little bit; remember that the return type is stored in the program type parameter. We can use good old pattern matching to know which instruction we are dealing with (Scala's support for pattern matching GADTs suffices in this case):

```scala
def run[A](program: IOProgram[A]): A =
  program match {
    case Write(msg) =&gt; println(msg)
    case Read =&gt; readLine
  }
```

The only thing left to do is reimplementing in a modular fashion our equivalent impure function. I'll leave the complete example below:

```scala
object SayWhat {
  /* Impure program */
  def sayWhat: String = readLine

  /* Functional solution */
  object Fun {
    // Language
    type IOProgram[A] = IOEffect[A]

    sealed trait IOEffect[A]
    case class Write(msg: String) extends IOEffect[Unit]
    case object Read extends IOEffect[String]

    // Pure Program
    def pureSayWhat: IOProgram[String] = Read

    // Interpreter
    def run[A](program: IOProgram[A]): A =
      program match {
        case Write(msg) =&gt; println(msg)
        case Read =&gt; readLine
      }

    // Composition
    def sayWhat: String = run(pureSayWhat)
  }
}
```

## Say What? (reloaded)
We'll start now building programs with more than one instruction. In this case we are going to print something to the console and then read the user's input.

### Impure program
```scala
def helloSayWhat: String = {
  println(&quot;Hello, say something:&quot;)
  readLine
}
```

As you can see, this is a common *imperative* program which can be read out aloud as follows: "first, do this; next, do this".

### Pure function and language
So far, our program definition has been just a type alias of a single instruction, but now we want our programs to be able to represent two-instructions programs as well. It turns out our program definition must be also an ADT:

```scala
sealed trait IOProgram[A]
case class Single[A](e: IOEffect[A]) extends IOProgram[A]
case class Sequence[A, B](e1: IOProgram[A], e2: IOProgram[B]) extends IOProgram[B]

def pureHelloSayWhat: IOProgram[String] =
  Sequence(
    Single(Write(&quot;Hello, say something:&quot;)),
    Single(Read))
```

As you can see, our programs can now be made up of just a `Single` instruction or a `Sequence` of two programs.

### Interpreter
We must now change the interpreter accordingly. In particular, we need two interpreters, one for programs and one for effects:

```scala
def run[A](program: IOProgram[A]): A =
  program match {
    case Single(e) =&gt; runEffect(e)
    case Sequence(p1, p2) =&gt;
      runProgram(p1) ; runProgram(p2)
  }

def runEffect[A](effect: IOEffect[A]): A =
  effect match {
    case Write(msg) =&gt; println(msg)
    case Read =&gt; readLine
  }
```

### Composition
The only thing left is to rewrite the impure program in a modular fashion:

```scala
def sayWhat: String = run(pureHelloSayWhat)
```

## Echo, echo!
In our next program we'll complicate the control flow a little bit.

### Impure program
```scala
def echo: Unit = {
  val read: String = readLine
  println(read)
}
```

Note that the `println` instruction is writing the result of the `read` operation. This is a behaviour we can't describe with our program representation yet. The problem is that the `Sequence` case doesn't allow us to use the result of the first program. We thus need somehow to represent context-dependent programs, i.e. programs that depend on the results of previous ones. Let's fix that.

### Pure function and language
```scala
sealed trait IOProgram[A]
case class Single[A](e: IOEffect[A]) extends IOProgram[A]
case class Sequence[A, B](e1: IOProgram[A],
  e2: A =&gt; IOProgram[B]) extends IOProgram[B]

def pureEcho: IOProgram[Unit] =
  Sequence(
    Single(Read), read =&gt;
    Single(Write(read)) )
```

As simple as that, the new version of `Sequence` carries as its second parameter a program that is allowed to depend on a value of type `A`, i.e. the type of values returned when the first program is interpreted. Of course, the intention is that the interpreter will apply this function to that precise value, as will be shown in the next section. By the way, does the signature of the new version of `Sequence` ring a bell?

### Interpreter
As commented previously, the new version of the interpreter will simply need to modify the way in which sequenced programs are executed:

```scala
def runProgram[A](program: IOProgram[A]): A =
  program match {
    case Single(e) =&gt; runEffect(e)
    case Sequence(p, next) =&gt;
      val res = runProgram(p)
      runProgram(next(res))
  }
```

### Composition
The last thing to do is to reimplement the impure function in a modular way by applying the interpreter to the result of the pure function.

```scala
def echo: Unit = runProgram(pureEcho)
```

## On pure values
There are still some impure IO programs we can't represent with our current `IOProgram` ADT. In particular, think of imperative programs structured as follows: *"Do this program; then, do this other program, possible taking into account the result of the last program; etc.; finally, return this value, possible taking into account the results of the last steps."*. It's the last step which can't be represented. For instance, let's consider the following program.

### Impure program
```scala
def echo(): String = {
  val read: String = readLine
  println(read)
  read
}
```

This program is similar to the last one, but this time we return the *String* read, i.e., a pure value. So let's add this new functionality to our ADT.

### Pure function and language
```scala
sealed trait IOProgram[A]
case class Single[A](e: IOEffect[A]) extends IOProgram[A]
case class Sequence[A, B](e1: IOProgram[A],
  e2: A =&gt; IOProgram[B]) extends IOProgram[B]
case class Value[A](a: A) extends IOProgram[A]

def pureEcho: IOProgram[String] =
  Sequence(
    Single(Read), read =&gt;
      Sequence(
        Write(read), _ =&gt;
          Value(read)))
```

This is the final form of our ADT, whereby a program can be one of three:


* Single: A single instruction.
* `Sequence`: A sequence of context-dependent programs.
* `Value`: A pure value (e.g. a `String`, `Int`, `MyFancyClass`, etc.)

### Interpreter
In order to update the interpreter, we just have to deal with our new type of IO programs.

```scala
def runProgram[A](program: IOProgram[A]): A =
  program match {
    case Single(e) =&gt; runEffect(e)
    case Sequence(p, next) =&gt;
      val res = runProgram(p)
      runProgram(next(res))
    case Value(a) =&gt; a
  }
```

As you can see, this interpretation is fairly easy: a program `Value(a)` just means "returns a", which is what our interpreter does.

### Composition
Last, we compose interpreter and pure function as usual to obtain a modular version of the original impure program:

```scala
def echo: String = runProgram(pureEcho)
```

## Conclusion
This post aimed at showing that no matter how complex you impure programs are, you can always design a DSL to represent those programs in a purely declarative way. In our case, the DSL for building IO programs we ended up with is pretty expressive. In fact, we can represent any kind of imperative control flow with it. Try it!

There are, however, two major flaws we have still to deal with. First, we have to admit that the readability of programs written in the final `IOProgram` DSL is ... poor, to say the least. Second, there is a lot of boilerplate involved in the design of the `IOProgram` type. Indeed, no matter the type of DSL we are dealing with (based on IO instructions, File system operations, Web service calls, etc.), if we need imperative features, we will need to copy &amp; paste the same `Sequence` and `Value` cases. We leave the solution to these problems for the next and last post of this series!

<strong>Edit:</strong> All code from this post can be found <a href="https://github.com/hablapps/gist/blob/master/src/test/scala/hello-monads/partII.scala" target="_blank">here</a>.

