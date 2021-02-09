---
layout: post
title: Algebras for the Masses!
date: 2017-02-20 18:47:51.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: []
tags: []
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '35663564'
  geo_public: '0'
  draftfeedback_requests: a:1:{s:13:"5874f2cfcde2a";a:3:{s:3:"key";s:13:"5874f2cfcde2a";s:4:"time";s:10:"1484059343";s:7:"user_id";s:8:"35663564";}}
  _publicize_job_id: '2066845134'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15919025;s:51:"https://twitter.com/jeslg/status/833734944396603392";}}
  _publicize_done_16092269: '1'
  _wpas_done_15919025: '1'
  publicize_twitter_user: jeslg
author: Jesus Lopez-Gonzalez
permalink: "/2017/02/20/algebras-for-the-masses/"
---
According to Wikipedia, *"an <a href="https://en.wikipedia.org/wiki/Algebraic_structure">Algebraic Structure</a> is a set with one or more finitary operations defined on it that satisfies a list of axioms"*. From a programming perspective, that sounds like a bunch of methods defined on a type. In fact, we can find many of those algebras represented as *type classes* in libraries such as *scalaz* or *cats*. This way of representing algebras is pretty related to <a href="https://www.cs.utexas.edu/~wcook/Drafts/2012/ecoop2012.pdf">*object algebras*</a>. However, it's quite common to hear about <a href="https://www.schoolofhaskell.com/user/bartosz/understanding-algebras">*F-algebras*</a> as well, an abstraction that arises from the field of *Category Theory*. Today, we'll see not only that both representations are isomorphic, but also how to systematically generate conversions between them. To validate those transformations, we'll scratch the surface of <a href="https://github.com/slamdata/matryoshka">Matryoshka</a> to fold several expressions with the aforementioned algebra representations. So here we go!

## Algebras and Their Representations
Undoubtedly, one of the most widespread algebraic structures in the functional programming community is *monoid*. Despite its simplicity, it turns out to be <a href="http://repository.upenn.edu/cgi/viewcontent.cgi?article=1773&context=cis_papers">very powerful</a>. Typically, in Scala type class libraries, monoid is represented as follows:

```scala
trait OMonoid[A] {
  def mzero(): A
  def mappend(a1: A, a2: A): A
}
```

This type class is what is known as the *object algebra interface*, an interface of an abstract factory to create expressions. It contains two methods: `mzero` and `mappend` which correspond with the two operations that describe this particular algebra. Once we have created the algebra interface, we could provide instances for it, that are also known as *object algebras*. A common monoid instance is *sum*:

```scala
val sumOMonoid: OMonoid[Int] = new OMonoid[Int] {
  def mzero(): Int = 0
  def mappend(a1: Int, a2: Int): Int = a1 + a2
}
```

Once we have shown object algebra fundamentals, it's time to focus on F-algebras. This is a really simple abstraction that consists of a *Functor* `F[_]`, a *carrier* type `A` and an algebra *structure* (the function itself):

```scala
type FAlgebra[F[_], A] = F[A] => A
```

At first glance, this looks very different from the object algebra approach for monoids. However, as we will see, the translation is completely natural. Indeed, this representation just packs all the algebra operations into a unique function. Thereby, the major challenge here is to identify the corresponding functor for monoids, which is an Algebraic Data Type with a representative for every operation conforming the algebra. We refer to it as the algebra signature:

```scala
sealed trait Σ[A]
case class MZero[A]() extends Σ[A]
case class MAppend[A](a1: A, a2: A) extends Σ[A]
```

Once the functor is defined, we can modularize Monoid as an F-algebra:

```scala
type FMonoid[A] = FAlgebra[Σ, A]
```

Finally, we could provide a *sum* instance for the brand new monoid representation, as we did with the previous approach:

```scala
val sumFMonoid: FMonoid[Int] = {
  case Mzero() => 0
  case Mappend(a1, a2) => a1 + a2
}
```

We claim that `sumFMonoid` is isomorphic to `sumOMonoid`. In order to provide such an evidence, we show the isomorphism between `OMonoid` and `FMonoid`:

```scala
val monoidIso = new (OMonoid <~> FMonoid) {

  val to = new (OMonoid ~> FMonoid) {
    def apply[A](omonoid: OMonoid[A]) = {
      case Mzero() => omonoid.mzero
      case Mappend(a1, a2) => omonoid.mappend(a1, a2)
    }
  }

  val from = new (FMonoid ~> Monoid) {
    def apply[A](fmonoid: FAlgebra[Σ, A]) = new FMonoid[A] {
      def mzero = fmonoid(Mzero())
      def mappend(a1: A, a2: A) = fmonoid(Mappend(a1, a2))
    }
  }
}
```

*(*) Notice that we have ignored the monoid laws along the article for simplicity, but keep in mind that they constitute a fundamental part of every algebra.*

Given this situation, the question we should be asking is: *"What is the best algebra representation for us?"* Sadly, there's no clear answer to this. On the one hand, there is F-algebra. Undoubtedly, this representation is more modular. In fact, it is used in projects such as <a href="https://github.com/slamdata/matryoshka/blob/master/core/shared/src/main/scala/matryoshka/package.scala#L55">Matryoshka</a>, a library of recursion-schemes that is able to generate fixed points for any `Functor`, or define a generic <a href="https://github.com/slamdata/matryoshka/blob/master/core/shared/src/main/scala/matryoshka/Recursive.scala#L35">catamorphism</a> (or `fold`) *once and for all*, which works for any F-algebra. On the other hand, there is the object algebra representation, closer to the widespread *programming interfaces*, that we can find in libraries such as <a href="https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Monoid.scala#L20">scalaz</a> or <a href="https://github.com/typelevel/cats/blob/155f7f534993c30d6e757de990330ac796dad5da/kernel/src/main/scala/cats/kernel/Monoid.scala#L11">cats</a>. Although not as modular as F-algebras, this representation is powerful enough to interpret algebra expressions with little effort. See <a href="http://www.cs.ox.ac.uk/jeremy.gibbons/publications/embedding.pdf">this paper</a> on shallow embedding to get a better intuition on that. Therefore, both representations do appear in prominent libraries of the functional programming community. Wouldn't it be nice to have them coexisting?

## Macro `@algebra` to Provide Conversions
As we have seen in the previous section, turning object algebras into F-algebras (and viceversa) is straightforward. Besides, we noticed that both algebra representations are used in everyday programming. For all these reasons, we decided to code an <a href="https://github.com/hablapps/azucar/blob/master/src/main/scala/macros/algebra.scala#L7">experimental macro</a> (`@algebra`) to make both representations live together. The macro annotation can be applied to an object algebra Interface:

```scala
@algebra trait OMonoid[A] {
  def mzero(): A
  def mappend(a1: A, a2: A): A
}
```

This annotation removes boilerplate by automatically <a href="https://gist.github.com/jeslg/bf1163433698be5e50375dab93e5e075">generating some F-algebra encodings</a>. They should enable us to translate object algebras wherever an F-algebra is required. To check that behaviour, we're going to invoke a Matryoshka *catamorphism* (`cata`) that requires an F-algebra as input parameter, but we'll be implementing object algebras instead. Besides, we'll be using the <a href="https://github.com/slamdata/matryoshka#introduction">tiny language</a> (num literals and multiplications) that is used in Matryoshka's introduction, so we recommend the reader to glance at it before moving ahead. Then, you should be able to appreciate that `Expr` leads to the following object algebra interface:

```scala
@algebra trait ExprAlg[A] {
  def num(value: Long): A
  def mul(l: A, r: A): A
}
```

First of all, <a href="https://gist.github.com/jeslg/9f9534a3a37b772e953d727a4e661039">our macro has to generate</a> the corresponding `Expr` signature. So, our auto generated companion for `ExprAlg` will contain:

```scala
sealed abstract class Σ[_]
case class Num[A](value: Long) extends Σ[A]
case class Mul[A](l: A, r: A) extends Σ[A]
```

In <a href="https://github.com/slamdata/matryoshka#algebras">this section</a> from Matryoshka's introduction, we see that it's required to provide a functor for `Expr` prior to apply a `cata`. Our macro is able to derive the Functor instance for `Σ`, so we don't have to worry about that. The document shows also an `eval` F-algebra, that we can translate easily to an object algebra:

```scala
implicit def eval = new Expr[Long] {
  def num(value: Long) = value
  def mul(l: Long, r: Long) = l * r
}
```

Notice that we marked it as `implicit`, because it will be necessary for the next task, which is invoking the catamorphism over an expression. Firstly, we need to declare the expression to be folded, I mean, evaluated. We can copy `someExpr` as is, and it will compile smoothly, since the `Mul` and `Num` case classes are generated by the macro as well:

```scala
def someExpr[T](implicit T: Corecursive.Aux[T, Σ]): T =
  Mul(Num[T](2).embed, Mul(Num[T](3).embed,
    Num[T](4).embed).embed).embed
```

Finally, we can invoke the `cata`. As we noted previously, it requires an F-algebra as input. Thereby, we use the `FAlgebra` summoner, generated by the macro, that detects the implicit `eval` and turns it into a compatible F-algebra to feed the function.

```scala
someExpr[Mu[Σ]].cata(FAlgebra[Long]) // ⇒ 24
```

To sum up, we applied our macro annotation to `ExprAlg` to generate some utilities to deal with F-algebras. Then, we defined our `eval` as an object algebra. As the generated encodings knew how to turn it into a F-algebra, we could invoke Matryoshka's `cata` with this algebra safely. Thus, we reach our objective of making both representations coexist nicely.

## Future Work
Today, we have seen `OMonoid[A]` and `ExprAlg[A]` as algebra examples, both demanding a concrete type parameter. However, there are algebras that are parametrized by a type constructor. Take `Monad[F[_]]` as an example. In this particular situation, we can't generate isomorphisms with F-algebras as we know them. Instead, we have to deal with F-algebras for <a href="http://www.timphilipwilliams.com/posts/2013-01-16-fixing-gadts.html">Higher Order Functors</a>. Our macro `@algebra` is able to detect GADTs and generate the corresponding encodings. This is still very experimental, but you can find <a href="https://github.com/hablapps/azucar/blob/master/src/test/scala/hk/AlgebraTest.scala">an example here</a>.

By now, we have placed `@algebra` in <a href="https://github.com/hablapps/azucar">*azucar*</a> (spanish word for "sugar"), a library where we plan to deploy more utilities to deal with (co)algebras. If you have some feedback or suggestion to improve it, we'd be very glad to hear from you. Anyway, we hope you've enjoyed reading!

