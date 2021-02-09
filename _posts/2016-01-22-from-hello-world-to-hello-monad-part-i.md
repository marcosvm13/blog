---
layout: post
title: From "Hello, world!" to "Hello, monad!" (Part I)
date: 2016-01-22 19:10:04.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- functional programming
- Scala
tags:
- DSLs
- metaprogramming
meta:
  _edit_last: '43713691'
  geo_public: '0'
  _post_restored_from: a:3:{s:20:"restored_revision_id";i:612;s:16:"restored_by_user";i:43713691;s:13:"restored_time";i:1453485619;}
  _publicize_job_id: '19018039883'
  _wpcom_is_markdown: '1'
author: Juan Manuel Serrano
permalink: "/2016/01/22/from-hello-world-to-hello-monad-part-i/"
---
This is the first instalment of a series of posts about the essence of functional programming. The only purpose of this series is to illustrate the defining features of this style of programming using different examples of increasing complexity. We will start with the ubiquitous "Hello, world!" and will eventually arrive at ... (throat clearing) monads. But we won't argue that monads are the essence of functional programming, and, ultimately, do not intend these posts to be about monads. In fact, we will stumble upon monads without actually looking for them, much in the same spirit of Dan Piponi's "<a href="http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html">You Could Have Invented Monads! (And Maybe You Already Have.)</a>".

There is a major difference between Dan's post and these ones, however: we won't be using Haskell but Scala, a language which unlike Haskell is not purely functional, i.e. that allows us to write non-functional programs. But this feature won't be a drawback at all. On the contrary, we think that it will allow us to emphasise some aspects (e.g. the interpreters) that may go unnoticed using a language like Haskell. Let's start with our first example!

## Hello, functional world!
This is a possible way of writing the "Hello, world!" program in Scala:

```scala
object Example1{
  def hello(): Unit =
    println("Hello, world!")
}
```

which can be run in the Scala REPL as follows:

```scala
scala> Example1.hello()
Hello, world!
```

As you can see, when this function is run the message "Hello, world!" is printed in the console. This is called a *side effect*, and it was indeed the purpose of the program. But this also means that our implementation is not purely functional. Why? Because functional programming is all about writing *pure* functions: functions which receive some input, compute some values and do *nothing else*. In a strongly-typed language such as Scala, we can witness the non-functional character of some function as follows: if the function does nothing else than returning values of the type declared in its signature, then it's a pure function; otherwise, it's an *impure* function: its signature declares that it does one thing, but it also does something more behind the compiler's back. We may thus also say that impure functions work in the *black market*, beyond the reach of the compiler's type system (our best ally!).

But, if pure functions only allow us to return *values*, how can we then print something to the console? How can we then execute any kind of effect (read something from the database, invoke a web service, write to a file, etc.)? How can we then do something useful at all? The answer is that you can't do those things with pure functions alone. Functional programming rests upon a basic modularity principle which tells us to decompose our applications into two kinds of modules: (1) functional modules, made up of pure functions that are responsible for computing *what* has to be done, and (2) non-functional modules made up of impure functions or programs in charge of actually *doing* it. It's this latter kind of modules which will do the dirty job of actually interacting with the outside world and executing the desired effects; and it's the only responsibility of the functional modules to determine which effects have to be executed. When programming functionally, you should not forget this fact: impure functions will eventually be necessary. The only thing that functional programming mandates is to segregate impure functions, so that the *ivory tower* where pure functions live is as large as possible, and the *kitchen* where impure, side-effecting functions operate is reduced to the bare minimum (which, nonetheless, in many applications might be quite large).

This limitation on the things that functional programming can do is a self-imposed constraint that doesn't actually constrain its range of application domains. Indeed, we can apply the functional programming style to any kind of application domain you may think of. So, what about our impure "Hello, world!" program? Can we *purify* it? Sure we can. But then, how can we disentangle its pure and impure parts? Essentially, functional programming tells us to proceed as follows (line numbers refer to the code snippet bellow):


* First, we define a *data type *that allows us to describe the kinds of effects we are dealing with*. *In our "Hello, world!" example, we want to talk about *printing *strings somewhere, so we will define the `Print `data type (cf. line 8).
* Second, we implement a function which *computes* the particular desired effects in terms of an instance of the previous data type. Our program then will be pretty simple (line 12): it simply returns the value `Print("Hello, world!")`. Note that this function is pure!
* Third, we implement a function that receives an instance of the effect data type, and executes it in any way we want. In our example, we will implement a function that receives a `Print `value and executes a `println` instruction to write the desired message to the console (line 17). This function is thus impure!
* Last, we compose our program out of the previous two functions, obtaining, of course, an impure program (line 21).

The resulting program is implemented in the `Fun` module (pun intended):

```scala
object Example1{ 

  /* Functional purification */
  object Fun{

    // Language
    type IOProgram = Print
    case class Print(msg: String)

    // Program
    def pureHello(): IOProgram =
      Print("Hello, world!")

    // Interpreter
    def run(program: IOProgram): Unit =
      program match {
        case Print(msg) => println(msg)
      }

    // Composition
    def hello() = run(pureHello())
  }
}
```

The equivalent program to our initial impure solution `Example1.hello` is the (also impure) function `Example1.Fun.hello`. Both functions has the same functionality, i.e. they both do the same thing, and from the point of view of the functional requirements of our tiny application, they are both *correct.* However, they are far from being similar in terms of their reusability, composability, testability, and other non-functional requirements. We won't explain in this post why the functional solution offers better non-functional guarantees, but ultimately the reason lies behind its better modularisation: whereas the `Example1.hello `function is monolithic, its functional counterpart is made up of two parts: the `pureHello `function and the impure function `run`.

Now, an important remark concerning the above code: note that we defined the alias `IOProgram` for our effect data type, and that we used the labels *Language*, *Program* and *Interpreter *for the different parts of our functional solution. This is not accidental, and it points at the <a href="https://www.youtube.com/watch?v=hmX2s3pe_qk">interpreter pattern</a>, arguably the essence of functional programming:


* First, our effect data type can be regarded as the *language* we use to describe the desired effects of our application. As part of this language, we can have different types of single effects, such as the `Print` effect or instruction. Also, since languages are used to write programs, expressions in our effect language can be called *programs*, and we can use the word "program" to name the type of the overall effect language. In our case, since we are dealing with IO effects, `IOProgram` is a good name. Last, note that the purpose of our language is very specific: we want to be able to write programs that just build upon IO instructions, so `IOProgram` is actually a *domain-specific* language (DSL). Here there are some hand-crafted programs of our IO DSL:```scala
scala> import Example1.Fun._
import Example1.Fun._
scala> val program1: IOProgram = Print("hi!")
program1: Example1.Fun.IOProgram = Print(hi!)
scala> val program2: IOProgram = Print("dummy program!")
program2: Example1.Fun.IOProgram = Print(dummy program!)
```
* So, pure functions return programs: this means that functional programming is intimately related to metaprogramming! And when we say that functional programming is *declarative*, we mean that functional programs just *declare* or *describe* what has to be done in terms of expressions or values. To convince yourself that our `pureHello` function is really declarative (i.e. pure), just execute it on the REPL. You will see that the only thing that happens during its execution is that a new value is computed by the runtime system (note that the output that you'll see in the REPL is not a result of the `pureHello` execution, but of the REPL itself):```scala
scala> Example1.Fun.pureHello()
res1: Example1.Fun.IOProgram = Print(Hello, world!)
```
* Once we execute a pure function and obtain a program, what is left is to actually *run* that program. But our program is an expression, pure syntax, so we have to choose a particular *interpretation* before we can actually run it. In our case, we chose to interpret our `IOProgram`s in term of console instructions, but we are free to interpret it otherwise (file IO, socket, etc.). When you run the program computed by our `pureHello` function, you will actually see the intended side effects:```scala
scala> Example1.Fun.run(Example1.Fun.pureHello())
Hello, world!
```

So, this is basically the structure of functional programming-based applications: domain-specific languages, pure functions that return programs in those DSLs, and interpreters that execute those programs. Note that these interpreters may be implemented in such a way that our programs are not directly executed but are instead translated to programs of some lower-level intermediate language (in a pure way). Eventually, however, we will reach the "bare metal" and we will be able to observe some side effect in the real world.

The IO programs that we are able to express with our current definition of the type `IOProgram` are very simple. In fact, we can just create programs that write single messages. Accordingly, the range of impure functions that we can purify is pretty limited. In our next posts, we'll challenge our IO DSL with more complex and realistic scenarios, and will see how it has to be extended in order to cope with them.

<strong>Edit:</strong> All code from this post can be found <a href="https://github.com/hablapps/gist/blob/master/src/test/scala/hello-monads/partI.scala" target="_blank">here</a>.

