---
layout: post
title: Yo Dawg, We Put an Algebra in Your Coalgebra
date: 2016-10-11 16:08:09.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- algebra
- coalgebra
- Embedded DSLs
- machine
- Scala
- Type Class
tags: []
meta:
  _wpcom_is_markdown: '1'
  geo_public: '0'
  _publicize_job_id: '27737752747'
  draftfeedback_requests: a:2:{s:13:"57fb776c3e408";a:3:{s:3:"key";s:13:"57fb776c3e408";s:4:"time";s:10:"1476097900";s:7:"user_id";s:8:"35663564";}s:13:"57fb77f03ad97";a:4:{s:3:"key";s:13:"57fb77f03ad97";s:4:"time";s:10:"1476098032";s:7:"user_id";s:8:"35663564";s:7:"revoked";s:1:"1";}}
  _edit_last: '35663564'
  _oembed_d076fa6cd436b4cec3434fd1c17c91b0: "{{unknown}}"
  _oembed_cf981a696f61707910799f6fed076921: "{{unknown}}"
  _oembed_b81ecd4a5ce0d054243473e1bcd249bd: "{{unknown}}"
  _oembed_853c3f0075b49ccf2ac20dd4d0af0d81: "{{unknown}}"
author: Jesus Lopez-Gonzalez
permalink: "/2016/10/11/yo-dawg-we-put-an-algebra-in-your-coalgebra/"
---
As Dan Piponi suggested in <a href="http://blog.sigfpe.com/2014/05/cofree-meets-free.html">Cofree Meets Free</a>, we may think of *coalgebraic things* as machines with buttons. In this post, we take this metaphor seriously and show how we can use algebras to model the Input/Output interface of the machine, i.e. its buttons. Prior to that, we'll make a brief introduction on coalgebras as they are usually shown, namely as F-coalgebras.

## What are F-coalgebras?
*F-coalgebra* (or functor-coalgebra) is just a reversed version of the more popular concept of <a href="https://www.schoolofhaskell.com/user/bartosz/understanding-algebras">*F-algebra*</a>, both of them belonging to the mystical world of Category Theory. The most widespread representation of an F-algebra is

```scala
type Algebra[F[_], X] = F[X] =&gt; X
```

(using *Scala* here) Paraphrasing Bartosz Milewski, "It always amazes me how much you can do with so little". I believe that its dual counterpart

```scala
type Coalgebra[F[_], X] = X =&gt; F[X]
```

deserves the very same amazingness, so today we'll put focus on them.

Given the previous representation, we notice that F-coalgebras are composed of a carrier `X`, a functor `F[_]` and a structure `X =&#062; F[X]` itself. What can we do with such a thing? Since we are just software developer muggles (vs matemagicians), we need familiar abstractions to deal with coalgebras. Therefore, we like to think of them as machines with buttons, which know how to forward a particular state (maybe requiring some input) to the next one (maybe attaching some output along) by pressing the aforementioned buttons. Now, let's find out some examples of mainstream machines that we, as functional programmers, already know:

```scala
// Generator Machine (Streams)
type GeneratorF[A, S] = (A, S)
type Generator[A, S]  = Coalgebra[GeneratorF[A, ?], S]

// Mealy Automata Machine
type AutomataF[I, S] = I =&gt; (Boolean, S)
type Automata[I, S]  = Coalgebra[AutomataF[I, ?], S]

// Lens Machine
type LensF[A, S] = (A, A =&gt; S)
type Lens[A, S]  = Coalgebra[LensF[A, ?], S]
```

Firstly, let's expand `Generator[A, S]` into `S =&#062; (A, S)` which is something easier to deal with. Indeed, it's just a function that, given an initial state `S`, it returns both the head `A` and the tail `S` associated to that original state. It's the simplest specification of a generator machine that one could find! Given a concrete specification and once provided an initial state, we could build a standard `Stream` of `A`s.

Secondly, we showed a Mealy `Automata`. Again, let's turn `Automata[I, S]` into `S =&#062; I =&#062; (Boolean, S)` to see it clearer: given the current state `S` and any input `I` we can determine both the finality `Boolean` condition and the new state `S`.

Finally, we saw `Lens`. Notice that the type parameters are reversed if we compare this lens with the "official" representation (eg. *lens*, *Monocle*, etc.). This is just to provide homogeneity with the rest of machines, where the state `S` is kept as the last parameter. As usual, let's expand `Lens[A, S]` to obtain `S =&#062; (A, A =&#062; S)`. This tell us that given an initial state `S`, we could either *get* the smaller piece `A` or *set* the whole state with a brand new `A`.

So far, we have seen the typical representation for some prominent coalgebras. On the other hand, we claimed that we like to think of those coalgebras as machines with buttons that let us make them work. That machine abstraction seems nice, but I agree it's difficult to see those buttons right now. So, let's find them!

## Coalgebras as machines? Then, show me the buttons!
As promised, we'll dive into F-coalgebras to find some buttons. I anticipate that those buttons are kind of special, since they could require some input in order to be pressed and they could return some output after that action. We're going to use `Lens` as a guiding example but we'll show the final derivation for our three machines at the end as well. So, we start from this representation:

```scala
type Lens[A, S] = S =&gt; (A, (A =&gt; S))
```

If we apply basic math, we can split this representation into a tuple, getting an isomorphic one:

```scala
type Lens[A, S] = (S =&gt; A, S =&gt; A =&gt; S)
```

Trust me when I say that every element in this tuple corresponds with an input-output button, but we still have to make them uniform. First of all, we're going to flip the function at the second position, so the input for that button stays in the left hand side:

```scala
type Lens[A, S] = (S =&gt; A, A =&gt; S =&gt; S)
```

Our button at the first position has no input, but we can create an artificial one to make the input slot uniform:

```scala
type Lens[A, S] = (Unit =&gt; S =&gt; A, A =&gt; S =&gt; S)
```

Once provided the input for the buttons, we reach different situations. On the first button there is `S =&#062; A` which is a kind of observation where the state remains as is. However, in the second button, there is `S =&#062; S` which is clearly a state transformation with no output attached to it. If we return the original state along with the observed output in the first button and provide an artificial output for the second one, we get our uniform buttons, both with an input, an output and the resulting state.

```scala
type Lens[A, S] = (Unit =&gt; S =&gt; (S, A), A =&gt; S =&gt; (S, Unit))
```

If we squint a bit, we can find an old good friend hidden in the right hand side of our buttons, the *State* monad, leading us to a new representation where both tuple elements are *Kleisli* arrows:

```scala
type Lens[A, S] = (Unit =&gt; State[S, A], A =&gt; State[S, Unit])
```

Finally, we can achieve a final step, aiming at both naming the buttons and being closer to an object-oriented mindset:

```scala
trait Lens[A, S] {
  def get(): State[S, A]
  def set(a: A): State[S, Unit]
}
```

So here we are! We have turned an F-coalgebra into a trait that represents a machine where buttons (get &amp; set) are certainly determined. Obviously, pressing a button is synonym for invoking a method belonging to that machine. The returning value represents the state transformation that we must apply over the current state to make it advance. If we apply the same derivation to streams and automata we get similar representations:

```scala
trait Generator[A, S] {
  def head(): State[S, A]
  def tail(): State[S, Unit]
}

trait Automata[I, S] {
  def next(i: I): State[S, Boolean]
}
```

We're glad we found our buttons, so we can reinforce the machine intuition, but *stranger things* have happened along the way... The coalgebraic *Upside Down* world is not quite far from the algebraic one.

## Buttons are Algebras
In the previous section we made a derivation from the Lens F-coalgebra to a trait Lens where buttons are made explicit. However, that representation was mixing state and input-output concerns. If we go a step further, we can decouple both aspects by abstracting the state away from the specification, to obtain:

```scala
trait LensAlg[A, P[_]] {
  def get(): P[A]
  def set(a: A): P[Unit]
}

type Lens[A, S] = LensAlg[A, State[S, ?]]
```

So, lenses can be understood as a state-based interpretation of a particular Input/Output algebra. We can distinguish in this kind of specification between two components: the IO interface and the state transition component. Why would we want to define our lenses, or any other coalgebra, in this way? One advantage is that, once we get this representation, where input-output buttons are completely isolated, we can make machine programs that are completely decoupled from the state component, and just depend on the input-output interface. Take *modify*, a standard lens method, as an example:

```scala
def modify[A, P[_]](
    f: A =&gt; A)(implicit
    P: LensAlg[A, P],
    M: Monad[P]): P[Unit] =
  P.get &gt;&gt;= (P.set compose f)
```

Notice that although `modify` constrains `P` to be monadic, this restriction could be different in other scenarios, as we can see with `gets`, where `Functor` is powerful enough to fulfil the programmer needs:

```scala
def gets[A, B, P[_]](
    f: A =&gt; B)(implicit
    P: LensAlg[A, P],
    F: Functor[P]): P[B] =
  P.get map f
```

These programs are absolutely declarative since nothing has been said about `P[_]` yet, except for the fundamental constraints. Indeed, this way of programming should be pretty familiar for a functional programmer: the step that abstracted the state away led us to a (Higher Kinded) <a href="https://www.cs.utexas.edu/~wcook/Drafts/2012/ecoop2012.pdf">object-algebra</a> interface, which is just an alternative way of representing algebras (as F-algebras are).

## Ongoing Work
We started this post talking about F-coalgebras, `type Coalgebra[F[_], X] = X =&#062; F[X]`, and then we turned our lens coalgebra example into a new representation where buttons and state transformation concerns are clearly identified (rather than being hidden into the functor 'F'). Indeed, we may tentatively put forward IO-coalgebras as a particular class of coalgebras, and define lenses as follows:

```scala
type IOCoalgebra[IOAlg[_[_]], Step[_, _], S] = IOAlg[Step[S, ?]]
type Lens[A, S] = IOCoalgebra[LensAlg[A, ?], State, S]
```

As we said in the previous section, this representation empowers us to use the existing algebraic knowledge to deal with coalgebras. So, although we started our journey aiming at the specification of machines, we were brought back to the algebraic world! So, which is the connection between both worlds? In principle, what we suggest is that coalgebras might be viewed as state-based interpretations of algebras. Now, whether any F-Coalgebra can be represented as an IO-Coalgebra is something that has to be shown. And, additionally, we should also identify the constraints in the IOCoalgebra definition that allows us to prove that the resulting formula is actually a coalgebra.

On future posts, we'll be talking about cofree coalgebras as universal machines. As we will see, those cofree machines exploit the button intuition to simulate any other machine in different contexts. By now, we'd be really grateful to receive any kind of feedback to discuss the proposed connection between languages and machines. Hope you enjoyed reading!

