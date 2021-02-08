---
layout: post
title: Lens, State Is Your Father
date: 2016-11-10 15:37:02.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- algebra
- coalgebra
- Lens
- Optics
- State
- Type Class
tags: []
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '35663564'
  geo_public: '0'
  _publicize_job_id: '28770050931'
  draftfeedback_requests: a:3:{s:36:"juanmanuel.serrano.hidalgo@gmail.com";a:3:{s:3:"key";s:13:"58123161e4ebd";s:4:"time";s:10:"1477587297";s:7:"user_id";s:8:"35663564";}s:21:"javierfs.89@gmail.com";a:3:{s:3:"key";s:13:"58123162cf7a5";s:4:"time";s:10:"1477587298";s:7:"user_id";s:8:"35663564";}s:13:"5823505d0b016";a:3:{s:3:"key";s:13:"5823505d0b016";s:4:"time";s:10:"1478709341";s:7:"user_id";s:8:"35663564";}}
author: Jesus Lopez-Gonzalez
permalink: "/2016/11/10/lens-state-is-your-father/"
---
In our <a href="https://blog.hablapps.com/2016/10/11/yo-dawg-we-put-an-algebra-in-your-coalgebra/">last post</a>, we introduced `IOCoalgebra`s as an alternative way of representing *coalgebras* from an algebraic viewpoint, where `Lens` was used as a guiding example. In fact, lens is an abstraction that belongs to the group of *Optics*, a great source of fascinating machines. We encourage you to watch this nice <a href="https://www.youtube.com/watch?v=6nyGVgGEKdA">introduction to optics</a> because we'll show more optic examples under the `IOCoalgebra` perspective. While doing so, we'll find out that this new representation let us identify and clarify some connections between optics and `State`. Finally, those connections will be analyzed in a real-world setting, specifically, in the *state module* from <a href="https://github.com/julien-truffaut/Monocle/">Monocle</a>. Let us not waste time, there is plenty of work to do!

*(*) All the encodings associated to this post have been collected <a href="https://github.com/hablapps/gist/blob/master/src/test/scala/LensStateIsYourFather.scala">here</a>, where the same sectioning structure is followed.*

## Optics as Coalgebras
First of all, let's recall the `IOCoalgebra` type constructor:

```scala
type IOCoalgebra[IOAlg[_[_]], Step[_, _], S] = IOAlg[Step[S, ?]]
```

As you can see, it receives three type arguments: the object algebra interface, the state-based action or step, and the state type. Once provided, coalgebras are defined as a state-based interpretation of the specified algebra. Take <a href="https://github.com/julien-truffaut/Monocle/blob/master/core/shared/src/main/scala/monocle/package.scala#L7">`Lens`</a> as an example of IOCoalgebra:

```scala
trait LensAlg[A, P[_]] {
  def get: P[A]
  def set(a: A): P[Unit]
}

type IOLens[S, A] = IOCoalgebra[LensAlg[A, ?[_]], State, S]
```

*(*) This is a simple Lens, in opposition to a <a href="https://github.com/julien-truffaut/Monocle/blob/master/core/shared/src/main/scala/monocle/Lens.scala#L33">polymorphic</a> one. We will only consider simple optics for the rest of the article.*

If we expand `IOLens[S, A]`, we get `LensAlg[A, State[S, ?]]`, which is a perfectly valid representation for lenses as demonstrated by the following isomorphism:

```scala
def lensIso[S, A] = new (Lens[S, A] &lt;=&gt; IOLens[S, A]) {

  def from: IOLens[S, A] =&gt; Lens[S, A] =
    ioln =&gt; Lens[S, A](ioln.get.eval)(a =&gt; ioln.set(a).exec)

  def to: Lens[S, A] =&gt; IOLens[S, A] = ln =&gt; new IOLens[S, A] {
    def get: State[S, A] = State.gets(ln.get)
    def set(a: A): State[S, Unit] = State.modify(ln.set(a))
  }
}
```

We'll see more details about lenses in later sections but, for now let's keep diving through other optics, starting with <a href="https://github.com/julien-truffaut/Monocle/blob/master/core/shared/src/main/scala/monocle/package.scala#L5">`Optional`</a>:

```scala
trait OptionalAlg[A, P[_]] {
  def getOption: P[Option[A]]
  def set(a: A): P[Unit]
}

type IOOptional[S, A] = IOCoalgebra[OptionalAlg[A, ?[_]], State, S]
```

This optic just replaces IOLens' `get` with `getOption`, stating that it's not always possible to return the inner value, and thence the resulting `Option[A]`. As far as we are concerned, there aren't more significant changes, given that `State` is used as step as well. Thereby, we can move on to <a href="https://github.com/julien-truffaut/Monocle/blob/master/core/shared/src/main/scala/monocle/package.scala#L3">`Setter`s</a>:

```scala
trait SetterAlg[A, P[_]] {
  def modify(f: A =&gt; A): P[Unit]
}

type IOSetter[S, A] = IOCoalgebra[SetterAlg[A, ?[_]], State, S]
```

In fact, this is a kind of relaxed lens that has lost the ability to "get" the focus, but is still able to update it. Notice that `set` can be automatically derived in terms of `modify`. Again, `State` is perfectly fine to model the step associated to this optic. Finally, there is <a href="https://github.com/julien-truffaut/Monocle/blob/master/core/shared/src/main/scala/monocle/Getter.scala#L14">`Getter`</a>:

```scala
trait GetterAlg[A, P[_]] {
  def get: P[A]
}

type IOGetter[S, A] = IOCoalgebra[GetterAlg[A, ?[_]], Reader, S]
```

This new optic is pretty much like a lens where the `set` method has been taken off, and it only remains `get`. Although we could use `State` to represent the state-based action, we'll take another path here. Since there isn't a real state in the background that we need to thread, ie. we can only "get" the inner value, `Reader` could be used as step instead. As an additional observation, realize that `LensAlg` could have been implemented as a combination of `GetterAlg` and `SetterAlg`.

There are still more optics in the wild, such as `Fold` and `Traversal`, but we're currently working on their corresponding IOCoalgebra representation. However, the ones that have already been shown are good enough to establish some relations between optics and the state monad.

## Optics and State Connections
Dealing with lenses and dealing with state feels like doing very similar things. In both settings there is a state that could be queried and updated. However, if we want to go deeper with this connection, we need to compare apples to apples. So, what's the algebra for `State`? Indeed, this algebra is very well known, it's named `MonadState`:

```scala
trait MonadState[F[_], S] extends Monad[F] {
  def get: F[S]
  def put(s: S): F[Unit]

  def gets[A](f: S =&gt; A): F[A] =
    map(get)(f)

  def modify(f: S =&gt; S): F[Unit] =
    bind(get)(f andThen put)
}
```

This `MonadState` version is a simplification of what we may find in a library such as *scalaz* or *cats*. The algebra is parametrized with two types: the state-based action `F` and the state `S` itself. If we look inside the typeclass, we find two abstract methods: `get` to obtain the current state and `put` to overwrite it, given a new one passed as argument. Those abstract methods, in combination with the fact that `MonadState` inherits `Monad`, let us implement `gets` and `modify` as derived methods. This sounds familiar, doesn't it? It's just the lens algebra along with the program examples that we used in our last post! Putting it all together:

```scala
trait LensAlg[A, P[_]] {
  def get: P[A]
  def set(a: A): P[Unit]

  def gets[B](
      f: A =&gt; B)(implicit
      F: Functor[P]): P[B] =
    get map f

  def modify(
      f: A =&gt; A)(implicit
      M: Monad[P]): P[Unit] =
    get &gt;&gt;= (f andThen set)
}
```

*(*) Notice that we could have had `LensAlg` extending `Monad` as well, but this decoupling seems nicer to us, since each program requires only the exact level of power to proceed. For instance, `Functor` is powerful enough to implement `gets`, so no `Monad` evidence is needed.*

Apparently, the only difference among `LensAlg` and `MonadState` lies in the way we use the additional type parameter. On the one hand, `LensAlg` has a type parameter `A`, which we understand as the focus or inner state contextualized within an outer state. On the other hand, we tend to think of `MonadState`'s `S` parameter as the unique global state where focus is put. Thereby, types instantiating this typeclass usually make reference to that type parameter, as one could appreciate in the `State` <a href="//github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/StateT.scala#L177)">instance</a> for `MonadState`. However, we could avoid that common practice and use a different type as companion. In fact, by applying this idea in the previous instance, we get a new lens representation:

```scala
type MSLens[S, A] = MonadState[State[S, ?], A]
```

*(*) The isomorphism between `IOLens` and `MSLens` is almost trivial, given the similarities among their algebras. Indeed, you can check it <a href="https://github.com/hablapps/gist/blob/master/src/test/scala/LensStateIsYourFather.scala#L135">here</a>.*

Lastly, we can't forget about one of the most essential elements conforming an algebra: its laws. <a href="http://www.cs.ox.ac.uk/people/jeremy.gibbons/publications/utp-monads.pdf">`MonadState` laws</a> are fairly known in the functional programming community. However, the laws associated to our `LensAlg` aren't clear. Luckily, we don't have to start this work from scratch, since <a href="http://sebfisch.github.io/research/pub/Fischer+MPC15.pdf">lens laws</a> are a good starting point. Despite the similarity between both packages of laws (look at their names!) we have still to formalize this connection. Probably, this task will shed even more light on this section.

## Monocle and State
Connections between optics and state have already been identified. Proof of this can be found in <a href="https://github.com/julien-truffaut/Monocle">Monocle</a>, the most popular Scala optic library nowadays, which includes a <a href="https://github.com/julien-truffaut/Monocle/tree/master/state/shared/src/main/scala/monocle/state">*state module*</a> containing facilities to combine some optics with State. What follows is a simplification (removes polymorphic stuff) of the class that provides conversions from lens actions to state ones:

```scala
class StateLensOps[S, A](lens: Lens[S, A]) {
  def toState: State[S, A] = ...
  def mod(f: A =&gt; A): State[S, A] = ...
  def assign(a: A): State[S, A] = ...
  ...
}
```

For instance, `mod` is a shortcut for applying `lens.modify` over the standing outer state and returning the resulting inner value. The next snippet, extracted from Monocle (type annotations were added for clarity), shows this method in action:

```scala
case class Person(name: String, age: Int)
val _age: Lens[Person, Int] = GenLens[Person](_.age)
val p: Person = Person(&quot;John&quot;, 30)

test(&quot;mod&quot;) {
  val increment: State[Person, Int] = _age mod (_ + 1)

  increment.run(p) shouldEqual ((Person(&quot;John&quot;, 31), 31))
}
```

That said, how can we harness from our optic representation to analyze this module? Well, first of all, it would be nice to carry out the same exercise from the `IOLens` perspective:

```scala
case class Person(name: String, age: Int)
val _ioage: IOLens[Person, Int] =
  IOLens(_.age)(age =&gt; _.copy(age = age))
val p: Person = Person(&quot;John&quot;, 30)

test(&quot;mod&quot;) {
  val increment: State[Person, Int] =
    (_ioage modify (_ + 1)) &gt;&gt; (_ioage get)

  increment.run(p) shouldEqual ((Person(&quot;John&quot;, 31), 31))
}
```

Leaving aside the different types returned by `_age mod (_ + 1)` and `_ioage modify (_ + 1)`, we could say that both instructions are pretty much the same. However, `mod` is an action located in an external state module while `modify` is just a primitive belonging to `IOLens`. Is this a mere coincidence? To answer this question, we have formalized this kind of connections in a table:

<table>
<thead>
<tr>
<th>Monocle State-Lens Action</th>
<th>IOLens Action</th>
<th>Return Type</th>
</tr>
</thead>
<tbody>
<tr>
<td>`toState`</td>
<td>`get`</td>
<td>`State[S, A]`</td>
</tr>
<tr>
<td>?</td>
<td>`set(a: A)`</td>
<td>`State[S, Unit]`</td>
</tr>
<tr>
<td>?</td>
<td>`gets(f: A ⇒ B)`</td>
<td>`State[S, B]`</td>
</tr>
<tr>
<td>?</td>
<td>`modify(f: A ⇒ A)`</td>
<td>`State[S, Unit]`</td>
</tr>
<tr>
<td>`mod(f: A ⇒ A)`</td>
<td>?</td>
<td>`State[S, A]`</td>
</tr>
<tr>
<td>`modo(f: A ⇒ A)`</td>
<td>?</td>
<td>`State[S, A]`</td>
</tr>
<tr>
<td>`assign(a: A)`</td>
<td>?</td>
<td>`State[S, A]`</td>
</tr>
<tr>
<td>`assigno(a: A)`</td>
<td>?</td>
<td>`State[S, A]`</td>
</tr>
</tbody>
</table>
What this table tells us is how the actions correspond to each other. For instance, the first raw shows that *toState* (from Monocle) corresponds directly with *get* (from our IOLens), both generating a program whose type is `State[S, A]`. The second raw contains a new element *?*, which informs us that there's no corresponding action for *set* in Monocle. Given the multitude of gaps in the table, we could determine that we're dealing with such different stuff, but if you squint your eyes, it's not hard to appreciate that `mod(o)` and `assign(o)` are very close to `modify` and `set`, respectively. In fact, as we saw while defining `increment`, `mod` is just a combination of `get` and `modify`. So, it seems to exist a strong connection between the `IOLens` primitives and the actions that could be placed in the state module for lenses. The obvious question to be asked now is: Is there such a connection between the state module and other optics? In fact, Monocle also provides facilities to combine `State` and `Optional`s, so we can create the same table for it:

<table>
<thead>
<tr>
<th>Monocle State-Optional Action</th>
<th>IOOptional Action</th>
<th>Return Type</th>
</tr>
</thead>
<tbody>
<tr>
<td>`toState`</td>
<td>`getOption`</td>
<td>`State[S, Option[A]]`</td>
</tr>
<tr>
<td>?</td>
<td>`set(a: A)`</td>
<td>`State[S, Unit]`</td>
</tr>
<tr>
<td>?</td>
<td>`gets(f: A ⇒ B)`</td>
<td>`State[S, Option[B]]`</td>
</tr>
<tr>
<td>?</td>
<td>`modify(f: A ⇒ A)`</td>
<td>`State[S, Unit]`</td>
</tr>
<tr>
<td>`modo(f: A ⇒ A)`</td>
<td>?</td>
<td>`State[S, Option[A]]`</td>
</tr>
<tr>
<td>`assigno(a: A)`</td>
<td>?</td>
<td>`State[S, Option[A]]`</td>
</tr>
</tbody>
</table>
Again, the results are very similar to the ones we extracted from `IOLens`. In fact, we claim that any `IOCoalgebra`-based optic which can be interpreted into `State` may contain a representative in the state module, and the actions that the module may include for each of them are just its associated primitives and derived methods. But, what about `Getter`s, where both `State` and `Reader` are suitable instances? Well, the `State` part is clear, we can add a new representative for `Getter` in the state module. However, the interesting insight comes with `Reader`: identifying new interpretations means identifying new modules. In this sense, we could consider including a new module *reader* in the library. Obviously, we could fulfill that module by following the same ideas that we showed for *state*.

To sum up, by following this approach, we have obtained a framework to systematically determine:


* The appropriateness of including a new module. 
* The optics that it may support.
* The methods it may contain for every optic.

This is a nice help, isn't it?

## Discussion and Ongoing Work
Today, we have seen that `IOCoalgebra`s served us two purposes, both of them involving understandability. First of all, we have identified an unexpected connection between `Lens` and the `State Monad`. In fact, we have defined `Lens` in terms of `MonadState`, so we had to explain `Lens` who was his biological father, and that was tough for her! Secondly, we have described a systematic process to create and fulfill Monocle's peripheral modules, such as *state*. In this sense, if we go one step further, we could think of those peripheral modules as particular interpretations of our optic algebras. This perspective makes the aforementioned process entirely dispensable, since optic instances would replace the module itself. As a result, logic wouldn't end up being contaminated with new names such as `assign` or `mod`, when all they really mean is `set` and `modify`, respectively.

As we mentioned before, we still have to translate other optics into their corresponding `IOCoalgebra` representation and identify the laws associated to the algebras. Besides, we focused on simple optics, but we should contemplate the polymorphic nature of optics to analyze its implications in the global picture. Anyway, optics are just a source of very low-level machines that conform one of the first steps in the pursue of our general objective, which is programming larger machines, ie. reactive systems, by combining smaller ones. It's precisely within this context where our optics, in combination with many other machines from here and there, should shine. In this sense, there's still a lot of work to do, but at least we could see that isolating algebras from state concerns has turned out to be a nice design pattern.

