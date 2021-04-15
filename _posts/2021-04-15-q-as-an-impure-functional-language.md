---
layout: post
title: "Q: The Tool of Functional Thought. Part I"
author: Jesús López-González
intro: "Q as an (Impure) Functional Language"
image: /img/trilogia-espejismo.jpg
intro_image_ratio: is-16by9
toc: true
---

Q was implemented by [Arthur
Whitney](https://queue.acm.org/detail.cfm?id=1531242) (Kx Systems), having its
first appearence in 2003. It relies on the shoulders of Kenneth E. Iverson and
his (Turing awarded) work on *A Programming Language* (APL) that started more
than six decades ago. Iverson emphasises the [importance of
notation](https://dl.acm.org/doi/pdf/10.1145/358896.358899) to concentrate on
more advanced problems, and finds in programming languages the idoneous setting
to make math notation universal, executable and unambiguous. In accordance with
such spirit, most of the APL primitives are still available on q.

We cannot find a better overview for q as the one contained in the first pages
of [Q Tips](https://www.q-tips.net): *"Q is an interpreted, dynamic,
event-driven, functional, array programming language."*. That long definition
lets us infer why learning q becomes such a challenge. Letting the eternal war
between interpreted/compiled and dynamic/static aside (compiled and static
always win, right? :), this article focuses on the *functional* feature, where
we functional programmers can benefit from, since we have walked this road
before.

Although there is an emerging interest on [taking q and kdb beyond financial
services](https://www.efinancialcareers.co.uk/news/2017/05/kdbq-banking-alternatives),
we won't be so original here and will use a trading indicator as an example. In
this sense, we'll try to keep it very easy.

> Most of the q/kdb+ tutorials and code snippets revolve around trading
> examples, so it is sometimes helpful to have an specialist nearby.

Our indicator simply consists on calculating the max price of an instrument
(for instance: AAPL, AMZN, etc.) in the last year. We'll assume that the
instrument prices are organized as a kind of table containing all the working
days (Monday-Friday) within the last year. In turn, each of them contains a
price update for every second within the working hours (in particular, from
09:00 to 17:30). We split the problem in three steps, where each of them
corresponds to the following post sections:
1. Calculating the max of two numbers
2. Calculating the max price within a day
3. Calculating the max price within a whole year

The first one is just a first contact with the language, where q operators and
types basics are introduced. The second one serves us as an excuse to show the
date api, lists, lambda expressions and *iterators* (which are essentially
higher order functions over collections). Finally, the last section presents
dictionaries and more iterators, where we briefly suggest connections with
functors and monads.

Throughout these posts, we will show q and scala snippets side by side. We want
to remark that our intention isn't to provide a comparison of these languages,
though, but rather support our explanations by means of snippets from a more
conventional functional language such as Scala, for merely didactic purposes. We
encourage you to [install q](https://code.kx.com/q/learn/install/) and type the
expressions below on your own.

# Calculating the max of two numbers

The q language allows us to calculate the max of two numbers by means of the
operator `|`. We show how to use it in the following snippet, extracted from a
q REPL session, where `q)` acts as the default prompt:
```q
q)3|2
3
```
Scala also has a REPL as part of its ecosystem, where the default prompt is
`scala> `. We could translate the very same logic into Scala using the `max`
method from `Integer`:
```scala
scala> 3 max 2
val res0: Int = 3
```

> We could move `|` to its prefix notation, using a familiar syntax for
> Haskellers, where we place the operator symbol in parentheses:
> ```q
> q)(|)[3;2]
> 3
> ```
> As can be seen, the arguments are separated by semicolons and surrounded by
> square brackets. As we'll see later, q supplies many flavours of syntactic
> sugar while invoking functions beyond these ones, but we just wanted to remark
> that the operator `|` could behave as any other function. This notion will
> become relevant later on.

In the previous snippets, we've just produced an output and we've let the REPL
to show it, but we could have assigned a name to the resulting value. The q
notation to introduce a variable `x` is as follows:
```q
q)x:3|2
q)x
3
```
Which we translate into Scala by means of a `var` instead of a`val`, since q
variables can be reassigned:
```scala
scala> var x: Int = 3 max 2
scala> x
val res2: Int = 3
```
Although q embraces immutability for a wide range of situations, it is clearly
not a pure functional language. It neither puts the same level of pressure on
avoiding mutable state as the one exhibited by Scala.

> A remarkably interesting q feature is that variables can be introduced at any
> point. For instance the expression `x:3|y:2` introduces `y` *on the fly* and
> just returns its value to keep going. We'd need two different statements to do
> that in Scala, as in `var y = 2; var x = 3 || y`.

Another important aspect from the previous snippets are the variable types. We
can see that the Scala version indicates that the type of `x` is `Int`, although
we could have removed it and let the type inferencer work for us. In the case of
q, which is a dynamic language, we could use the `type` primitive to find it:
```q
q)type x
-7h
```
Where you hoping to find something more familiar? Welcome to q! The number 7
indicates that the value of this type is a long; the negative symbol sugggests
an atomic value, i.e. not a list of longs; and the final `h` just manifests that
the value that `type` returns has *short* as type. You can find the complete
relation between numbers and types in [this
section](https://code.kx.com/q/basics/datatypes/) from the official
documentation.

# Calculating the max price within a day

Instead of assuming a predefined input list to play with, we find it convenient
to show how to generate a list of random prices. Once generated, we'll move on
to the actual calculation of the max value within it. Finally, we'll introduce a
variation of the problem, where we'll calculate the max value as long as it
doesn't exceed a given limit.

## Generating random prices for a working day

As mentioned before, we assume that the input includes a price update for every
single second between the range that goes from 09:00 to 17:30. How many seconds
are there in such interval?
```q
q)n:7h$17:30:00-09:00:00
30600
```
> Have you noticed the lack of space characters within the q expressions? Q is
> really committed to shortness and encourages the programmer to limit them.
> This subtle difference has a considerable impact while reading q code,
> although eventually, you get used to it.

We clumsily adapt the previous expression into Scala using the *duration*
interface:
```scala
scala> import scala.concurrent.duration._

scala> val n = (17.hours+30.minutes - 9.hours).toSeconds
val n: Long = 30600
```
There are several aspects to discuss here, but we start emphasising that q
primitives for dates and times are just superb. Creating, operating and casting
different units of times is a clean, elegant and intuitive task. In the example
above, we just create the opening and closing seconds and then we subtract them.
We use the resulting output to calculate the number of seconds in the interval.
The most striking feature of the q snippet is perhaps the `7h$` at the
beginning, which somehow corresponds to the `toSeconds` invocation from the
Scala definition. In this sense, we use the `$` operator to cast an expression
to another type. As you already know, `7` is associated to the *long* type, so
we are casting the subtraction result to its numeric form.

> At first sight, we could infer that `x$y-z` interprets the subtraction before
> the casting due to `-` having a higher precedence. But this is not the case.
> In fact, no q operator has higher precedende than another, since it will
> always interpret expressions from right to left. For instance, `2*3+1` returns
> `8`. If you combine this right-biased interpretation with the fact that it is
> possible to introduce variable names at any point of an expression, you can
> find yourself spending a non-negligible amount of time trying to understand
> why the following expression does return `4`:
> ```q
> q)x:0
> q)x*3+2-x:1
> 4
> ```
> Notice that the second line rewrites `x` at the very beginning, remember, at
> the rightmost expression. We show the Scala analogous to remark this aspect:
> ```scala
> scala> var x = 0
> var x: Int = 0
> 
> scala> x = 1; x * (3 + (2 - x))
> val res3: Int = 4
> ```
> Again, this way of interpreting code is yet another hurdle that makes q
> difficult to read for newbies, but eventually, you get used to it. Before
> moving on, we want to clarify that q programmers can change associativity by
> using parenthesis, for instance: `(2*3)+1`, although it's more idiomatic to
> avoid them and reorder the code, if possible.

Once we know the number of random prices that the intraday list will contain,
which we assigned to the variable `n`, it is time to generate them. To do so, we
simply use the `?` operator:
```q
q)prices:n?1000f
231.8545 102.0847 974.3216 673.6161 404.1387 626.0377 211.9141 604.1371 52.77..
```
Now, we show what we have considered its Scala counterpart by means of
`util.Random.nextFloat`:
```scala
scala> val prices = List.fill(n)(nextFloat).map(_ * 1000)
val prices: Seq[Float] = List(618.7332, 216.10922, 481.55737, 257.13562, 95.020..
```
The previous snippets generate `n` float numbers in the range that goes from
zero to one thousand.

> Note that the generation of random numbers by means of `?` is quite simple but
> impure, since this operation is not referentially transparent. We avoid
> introducing seeds in the Scala version to keep the comparison more direct.
> Indeed, we haven't seen references to seeds in our brief experience as q
> programmers.

A nice feature from both q and Scala is that most of times the output reflects
the very same code that we need to build such value. For instance, we can
generate a list of floats using the very same notation:
```q
q)231.8545 102.0847 974.3216
231.8545 102.0847 974.3216
```
whose associated type is `9h`, meaning a list of floats, and that we adapt to
Scala as follows:
```scala
scala> List(231.8545f, 102.0847f, 974.3216f)
val res4: List[Float] = List(231.8545, 102.0847, 974.3216)
```
Having generated a completely crazy list of intraday prices that nobody should
invest in, we will finally proceed to calculate its higher value.

> We recommend the *Q Tips* book to find a more realistic generation of random
> prices, which goes beyond the scope of this post.

## Finding the max value

As functional programmers, we would find the greatest value by *folding* the
list. In q, the analogous for this function is the so-called `over` *iterator*
(`/`).

> The notion of *iterator* in Scala refers to a different concept, namely
> generators. Indeed, we should understand [q
> iterators](https://code.kx.com/q/ref/iterators/) as a catalogue of
> higher-order functions over collections such as lists.

This operator takes the reducing function and the list itself as input
arguments, so we could get the highest price by passing the `|` operator as
reducer:
```q
q)(|/)prices
999.9987
```
We can get the analogous behaviour in Scala by using the `reduce` method and
`max` as reducer:
```scala
scala> prices.reduce(_ max _)
val res5: Float = 999.99884
```
Obviously, they don't lead to the very same output since each version produces
its own random numbers, but they are close enough!

As a pure and total functional programmer you might be missing the part of the
algebra that corresponds to the `Nil` (or empty list) case. In fact, `over`
would return `()` when we pass an empty list as second argument. This value
represents the empty list and I guess we could map it as a kind of Scala's `()`,
which corresponds to the unique instance for the *Unit* type. So, to a certain
extent, we could consider that `(+/)` returns either `()` or the greatest value,
the dynamic poor man's type for `Either[Unit, Float]`, [isomorphic to the
`Option`
type](https://bartoszmilewski.com/2015/01/13/simple-algebraic-data-types/). The
Scala version simply raises an exception when `reduce` is invoked from an empty
list. To make things safer, `over` can take an additional argument to
contemplate the `Nil` case as well, as we show in the following snippet:
```q
q)0|/prices
999.9987
```
which would produce `0` when prices correspond to an empty list. From the Scala
viewpoint, we can use the pure `fold` method instead of `reduce` (which is
actually implemented as a `foldLeft`):
```scala
scala> prices.fold(0f)(max)
val res6: Float = 999.99884
```

> One of the fundamental pilars of APL is the *suggestivity* of notation, where
> Iverson emphasises the importance of inferring new behaviours from existing
> expressions. In this sense, we could guess that by passing the operator that
> calculates the minumum of two values as an argument for `over`, we should be
> able to obtain the lowest price:
> ```q
> q)(&/)prices
> 0.0008079223
> ```
> The same suggestivity applies to Scala, where we can pass the proper operation
> as an argument to `reduce`:
> ```scala
> scala> prices.reduce(min)
> val res7: Float = 0.02861023
> ```

Finally, we must say that calculating the maximum value from a given list is so
common, that q supplies `max` as an alias for `+/`, as in `max prices`. Scala
does also supply the analogous alternative, as in `prices.max`.

## Finding the max value with an upper bound

Calculating the maximum and minimum prices is ok, but we could be interested in
implementing more sophisticated indicators. For instance, calculating the higher
price that don't exceed a given limit. At this point, one may wonder if `over`
is restricted to native predefined operators or if we could pass our own
operator as argument in order to implement that logic. Q, being a functional
language, provides support for lambda expressions, as we show next:
```q
q)0{[x;y]$[y<500f;x|y;x]}/prices
499.9798
```
We rely on the Scala adaptation to explain what is going on:
```scala
scala> prices.fold(0f)((x, y) => if (y < 500f) max(x, y) else x)
val res8: Float = 499.98163
```
As you can see, we replace `|` with the lambda expression that implements the
desired logic: getting the max of `x` and `y` as long as `y` is lower than
`500f`. The q lambda expression is surrounded by curly braces, where `[x;y]`
correspond to the input parameters, using a consistent notation with regard to
the argument passing style. The rest of the expression (`$[y<500f;x|y;x]`) acts
as the body, where the `$` operator is therefore the analogous for an `if`
statement.

> So far, we've seen that the character `$` serves as the casting operator and
> as the *if* statement. These kind of symbol overloading is very frequent and
> turns out to be a major barrier to start reading q code from experienced
> programmers. I guess that the same barrier applies for Scala newbies trying to
> understand the many intentions of a character such as `_`.

It's worth mentioning that when the parameter block is omitted, q will
understand names `x`, `y` and `z` as the first, second and third parameters,
respectively. It's quite similar to the Scala *placeholder* syntax (`_ + _`),
without the limitation of having to use each parameter exactly once. On its
part, q has the limitation of lacking additional names for functions with a
number of parameters greater than three. We apply this idea in the following
definition, where we parameterize the hardcoded limit as an additional argument
and assign the resulting function a name:
```q
q)lim:{$[y<z;x|y;x]}
```
The Scala counterpart would be:
```scala
scala> val lim: (Float, Float, Float) => Float = (x, y, z) => if (y < z) x max y else x
```
Unfortunately, we can exploit Scala placeholder syntax here.

Once we have defined the new name to get the maximum value which is in turn
lower than a given limit, we can modularise the previous logic:
```q
q)0 lim[;;500f]/prices	
499.9798
```
We can adapt this code into Scala as follows:
```scala
scala> prices.fold(0f)(lim(_, _, 500f))
val res10: Float = 499.88913
```
As you have probably guessed by the Scala definition, what q achieves in
`lim[;;500f]` is to fix the third argument to `500f` and return a function that
still expects the first and second ones, as determined by the lack of arguments
for such positions. In q, this technique is known as *projection*.

> Clearly, projection is somehow related to currification. In a way, q deploys
> both the currified and non-currified versions of every function. In this
> sense, `lim` can be invoked both as `lim[1;2;3]` and `lim[1][2][3]`. If we
> combine this currying notion with projection, the flexibility of function
> invocacion becomes astonishing. To illustrate it, we show further alternatives
> for the very same invocation:
> ```q
> lim[1][2;3]
> lim[;2;][1][3]
> lim[;;3][;2][1]
> lim[1;;3][2]
> ```

Before moving on to the next section, we'd like to clarify that the *limit*
logic could benefit from a different implementation. In fact, we think that the
following code is more idiomatic in q:
```q
q)max prices where prices<500f
499.9798
```
The previous code could be adapted into Scala as follows:
```scala
scala> prices.filter(_ < 500f).max
val res11: Float = 499.88913
```
However, the q approach is radically different in the shadows, as the next post
will show.

# Calculating the max price within a whole year

Again, we find it interesting to generate the random prices from scratch. In
fact, instead of simply using a longer list of prices, we'll produce a list of
prices associated to every working day, to keep data tidier. Once generated,
we'll move on to the actual calculation of the max value within the brand new
structure.

## Generating a whole year of random prices

First of all, we'll use the techniques that we've been learning on the previous
section to make the existing functions more reusable. For example, we adapt the
generation of the prices for a day as follows:
```q
q)rnd_prices:{(7h$y-x)?z}
```
As usual, we adapt the snippet into Scala:
```scala
scala> val rnd_prices: (Duration, Duration, Float) => List[Float] = {
     |   case (x, y, z) => List.fill((y-x).toSeconds)(nextFloat).map(_ * z)
     | }
```
As you can see, we've just parameterize the starting time, ending time and
higher price as `x`, `y` and `z`, respectively. This function will be reused to
generate different random prices for each working day.

Before moving on, we need to identify the working days within a range of dates.
We do so by means of the next function:
```q
q)working_days:{dates where((dates:x+til(y-x))mod 7)>1}
```
This time we avoid showing the Scala counterpart, since it would be far more
complex and doesn't add value from a didactic perspective. Anyway, the previous
function just keeps working days, those whose modulo 7 is greater than 1. To
understand why, we need to take into account that q dates start counting on
2000.01.01, which happened to be Saturday. We supply the starting and ending
dates that allow us to collect the working days from 2020 (such a wonderful
year, huh?):
```q
q)wds:working_days[2020.01.01;2021.01.01]
```
We'll assume that such list was generated in Scala somehow:
```scala
scala> val wds: List[Date] = ...
```

Now, it's time to associate the random prices for each working day. We do so by
means of the following expression, where two new features are introduced:
```q
q)prices:wds!{rnd_prices[09:00:00;17:30:00;1000f]}each wds
q)prices
2020.01.01| 447.9321 687.9944 491.4469 426.7794 650.8995 147.1279 440.327  37..
2020.01.02| 601.4124 818.0695 549.0516 985.867  387.0052 315.4341 338.4381 40..
2020.01.03| 811.1749 237.0332 220.7359 435.7565 190.276  35.80185 491.0418 82..
2020.01.06| 780.8859 5.676414 286.5235 149.7137 568.0527 916.0366 66.16259 46..
..
```
It's adapted into Scala as follows:
```scala
scala> val prices: Map[Date, List[Float]] =
     |   wds.zip(wds.map(_ => rnd_prices(9.hours, 17.hours+30.minutes, 1000f))).toMap

val prices: Map[Date,List[Float]] = Map(2020.01.01 -> List(977.59784, 185.63521,
586.2779, 221.09216, 775.3352, 645.992, 206.07281, 427.91003, 166.2563,
639.81836, 717.57886, 842.7385, 189.36241, 755.4852, 229.79778, 548.248,
472.32468, 383.4009, 920.29846, 211.65651, 132.54398, 514.35223, 135....
```
On the one hand, we remark the iterator `each`. It corresponds to the `map`
invocation on the Scala snippet, that we associate to the `Functor` typeclass.
In fact, they both allow us to apply a function over each element at the
collection. The mapper ignores the existing value, since the objective here is
to replace the date with the random prices.

> The Scala version could avoid the `zip` invocation and reuse `map` for putting
> dates and lists together:
> ```scala
> scala> val prices: Map[Date, List[Float]] =
>      |   wds.map((_, rnd_prices(9.hours, 17.hours+30.minutes, 1000f))).toMap
> ```
> However, we think that the original version has a better correspondence with
> the q one.

On the other hand, we must focus on the `!` operator as well, since it's
introducing a major abstraction from q: *dictionaries*. Although the inner
implementation details may be completely different, I find it fair to compare
dictionaries with maps. The `!` operator is adapted as a combination of `zip`,
to place keys with values together, and `toMap`, to turn the list of pairs into
an actual map. By using it, we end up with a collection where each working day
has a list of prices associated.

## Finding the max value

Given the generated prices, we think that there are two main approaches to
calculate the higher price: 
- Calculate the maximum price for each day and then calculate the maximum one
  among them
- Put all the prices together and just calculate the maximum one

The first approach is carried out in the next code:
```q
q)max max each prices
999.9998
```
that we can translate into Scala this way:
```scala
scala> prices.mapValues(_.max).max._2
val res12: Float = 999.9997
```
As suggested by the Scala expression, when we use the `each` iterator over a
dictionary, we are actually applying the mapper over the values, keeping the
keys as is. Once we've calculated the max value associated to each day, we can
invoke `max` directly over the resulting collection. In the Scala case, we need
to pick the value as a final step, since the involved key is also returned.

The second approach is implemented as follows:
```q
q)max raze prices
999.9998
```
It is adapted into Scala using `flatten`:
```scala
scala> prices.values.flatten.max
```
Indeed, `raze` just flattens the dictionaries of lists, and roughly corresponds
to the `join` monadic operation. If we take into account that `,` is the list
concatenation operator, it should be straightforward to understand the
implementation of `raze`:
```q
q)raze
,/
```
Indeed, it just uses the `over` iterator using `,` as reducer.

At this point we must say that the first approach is preferable, since it's more
modular and therefore parallelizable. In fact, there's a variant of `each` which
is referred to as `peach` that we could use to exploit such aspect. However, we
wanted to show the second approach since it's almost mandatory for a functional
programming-related post to make [*yet another
monad*](https://mvanier.livejournal.com/3917.html) reference, isn't it?

# Takeaways

* Q enjoys the common traits of functional languages: lambdas, higher-order
  functions, etc. with some nice features like projection.
* Q is not for the purest minds, given that side effects are the norm in certain
  situations, such as the generation of random numbers or the reassignment of
  variables.
* Although we acknowledge that q notation isn't for everyone, we find it very
  beautiful and extremely concise (even when we let the insane lack of space
  characters and operator overloading aside).

Before concluding, we must warn you that this post is far from reflecting the
real power of q. You should wait to our next post on array processing to see it!
It takes a while to experience the Zen, but you'll get a really powerful *tool
of thought* as a reward.

