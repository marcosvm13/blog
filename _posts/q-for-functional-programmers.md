# Q For (Mortal) Functional Programmers

## About this series

This series of posts describes our experience while learning
[*q*](https://code.kx.com/q/learn/startingkdb/language/) and *kdb+* by Kx
Systems. Although q is also a functional programming language, it has many
features (highlighting its capabilities for array processing) that make it
different from other conventional languages such as Haskell, OCaml, etc. Given
this situation, we will try to provide an overview of q basics, connecting the
missing pieces to our previous knowledge on functional programming, using Scala
and Spark as vehicles. We hope it also works in the opposite direction, so q
programmers can benefit from this article as well.

It is worth noting that this is the first of two posts that will focus on q and
kdb, respectively. Since we know that the line that divides q and kdb is
unclear, we want to clarify that the first post introduces q as a
general-purpose programming language, from the functional programmer
perspective, while the second one emphasizes persistence and interaction with
the file system, to make the data engineer happy. In this sense, data structures
such as tables and the q-sql interface will be unavoidable included in both
articles. Hopefully, each post will emphasize different aspects from them, to
avoid the overlapping of content. Having said so, we are ready to go now!

## Why q?

Each day, our small team spend some time experimenting with new technologies,
and we are especially interested on functional languages. A colleague from the
Scala community pointed us towards q. The following items summarise the reasons
why we decided to give it a go:

- Q is fast, sooo fast
- Q is a functional query language
- Q is a well-founded language that relies on APL
- Q is highly demanded in the financial industry
- Q is quite a challenge!

After a few months of reading q material and coding, we can confirm that none of
the previous items is a myth. Although we feel that we still have a long way to
master q/kdb, we are confident that we have now a good perspective on how hard
it is to learn this language. In fact, reflecting this experience is perhaps the
major contribution of this post.

From now on, we will introduce q from different perspectives: as yet another
functional language, as an array processing language and finally, as a query
language. We will use examples from trading to guide the explanations, where we
will show q and Scala snippets side by side. We want to remark that our
intention is not to provide a comparison of these languages, but rather
introducing q in terms of a more conventional functional language, for merely
didactic purposes. Indeed, connecting new knowledge to existing one has proven
to be a useful tool for learning.

## Q as a functional language

Q relies on the shoulders of Kenneth E. Iverson and his (Turing awarded) work on
*A Programming Language* (APL) that started more than six decades ago. Iverson
emphasises the [importance of
notation](https://dl.acm.org/doi/pdf/10.1145/358896.358899) to concentrate on
more advanced problems, and finds in programming languages the idoneous setting
to make math notation universal, executable and unambiguous. In accordance with
such spirit, most of the APL primitives are still available on q.

We cannot find a better overview for q as the one contained in the first pages
of [Q Tips](https://www.q-tips.net): *"Q is an interpreted, dynamic,
event-driven, functional, array programming language."*. That long definition
let us infer why learning q becomes such a challenge. Letting the eternal war
between interpreted/compiled and dynamic/static aside (compiled and static
always win, right? :), this section put focus on the *functional* feature, where
we functional programmers can benefit from, since we have walked this road
before.

Although there is an emerging interest on [taking q and kdb beyond financial
services](https://www.efinancialcareers.co.uk/news/2017/05/kdbq-banking-alternatives),
I think it is fair to say that nowadays, most of q-related job positions fall
under this umbrella. Thereby, we find it convenient to use different trading
indicators to guide the explanations. In this sense, we must say that most of
kdb examples and tutorials revolve around them, so it is sometimes helpful to
have an specialist nearby. But do not worry about it, we will try to keep it
very easy. In fact, we will start by calculating the max and min price of an
instrument.

The q language allows us to calculate the max of two numbers by means of the
operator `|`. We show how to use it in the following snippet, extracted from a
q REPL session, where `q)` acts as the default prompt:
```q
q)3|2
3
```
Scala also has a REPL as part of its ecosystem, where the default prompt is
`scala> `. We could translate the very same logic into Scala using `Math.max`:
```scala
scala> import Math.max
scala> max(3, 2)
val res0: Int = 3
```
As can be seen, we need to import the `Math` module in order to get such
functionality. This can be extrapolated to many other math operators, which are
loaded by default in q, mainly becauses of its bias towards analytics.

It would not be difficult to supply an infix alias for `Math.max` in Scala
(although we adopt `||` instead of `|`, since the latter is already associated
to the *bitwise or* operator):
```scala
scala> implicit class IntExt(x: Int) {
     |   def ||(y: Int): Int = max(x, y)
     | }

scala> 3 || 2
val res1: Int = 3
```
However, it is perhaps more interesting to follow the opposite path and move
`|` to its prefix notation, using a familiar syntax for Haskellers, where we
place the operator in parentheses:
```q
q)(|)[3;2]
3
```
As can be seen, the arguments are separated by semicolons and surrounded by
square brackets. Have you noticed the lack of space characters among them? Q is
really committed to shortness and invites the programmer to limit them. This
subtle difference has a considerable impact while reading q code, although
eventually, you get used to it.

In the previous snippets, we have just produced an output and we have let the
REPL to show it, but we could have assigned a name to the resulting value. The q
notation to introduce a variable `x` is as follows:
```q
q)x:3|2
q)x
3
```
Which we translate into Scala by means of a `var` instead of a`val`, since q
variables can be reassigned:
```scala
scala> var x: Int = max(3, 2)
scala> x
val res0: Int = 3
```
Despite q embraces immutability for a wide range of situations, it is clearly
not a pure functional language. It neither put the same level of pressure on
avoiding mutable state as the one exhibited by Scala.

Another important aspect from the previous snippets are the variable types. We
can see that the Scala version indicates that the type of `x` is `Int`, although
we could have removed it and let the type inferencer work for us. In the case of
q, which is a dynamic language, we could use the `type` primitive to find it:
```q
q)type x
-7h
```
Where you hoping to find something more familiar? Welcome to q! The number 7
indicates that the value of this type is a long. On its part, the negative
symbol sugggests an atomic value, ie. not a list of longs. The final `h` just
indicates that the value that `type` returns has *short* as type. You can find
the complete relation between numbers and types in [this
section](https://code.kx.com/q/basics/datatypes/) from the official
documentation.

Once we know how to get the max value of two numbers, we will extend it in order
to calculate the max price of a given list, which corresponds to our first
instrument. To do so, we will generate a list of random numbers to play with. In
particular, we will generate a different price for each intraday second,
starting at 09:00 and closing at 17:30. How many seconds are there in such
interval?
```q
q)7h$17:30:00-09:00:00
30600
```
We clumsily adapt the previous expression into Scala as follows:
```scala
scala> import scala.concurrent.duration._
import scala.concurrent.duration._

scala> (17.hours+30.minutes - 9.hours).toSeconds
val res18: Long = 30600
```
There are several aspects to discuss here, but we start emphasising that q
primitives for dates and times are just superb. Creating, operating and casting
different units of times is a clean, elegant and intuitive task. In the example
above, we just create the opening and closing seconds and then we subtract them.
See that `7h$` at the beginning? We use the `$` operator to cast an expression
to another type. As you already know, `7` is associated to the *long* type, so
we are casting the subtraction result to its numeric form. Dates and times in
Scala and the JVM are object of discussion, where we can find a vast field of
non-interoperable libraries that make the hell out of a programmer, so we will
not go into further detail around dates. However, we can see that the Scala
expression includes parenthesis to determine associativity, so we will take the
opportunity to discuss it, along with operator precedence.

At first sight, we could infer that `x$y-z` interprets the subtraction before
the casting due to `-` having a higher precedence. But this is not the case. In
fact, no operator has higher precedende than another, since q will always
interpret expressions from right to left. For instance, `2*3+1` returns `8`.  If
you combine this right-biased way of interpretating with the fact that it is
possible to introduce variable names at any point of an expression, you can find
yourself spending a non-negligible amount of time trying to understand why the
following expression does return `4`:
```q
q)x:0
q)x*3+2-x:1
4
```
Notice that the second line rewrites `x` at the very beginning, I mean, at the
rightmost expression. We show now the Scala analogous:
```scala
scala> var x = 0
var x: Int = 0

scala> x = 1; x * (3 + (2 - x))
val res23: Int = 4
```

lists
iterators
lambdas/def/unit
dictionaries
apply
curry

## Q as an array processing language

## Q as a query language

## Takeaways

