---
layout: post
title: Updating immutable objects in generic contexts
date: 2013-03-07 11:26:17.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- case classes
- immutability
- Macros
- Scala
- Speech
- Type Class
- Updatable
tags: []
meta:
  _edit_last: '43713691'
  draftfeedback_requests: a:1:{s:13:"5135dc82cfb35";a:3:{s:3:"key";s:13:"5135dc82cfb35";s:4:"time";s:10:"1362484354";s:7:"user_id";s:8:"43713691";}}
  draft_feedback: "a:1:{s:13:\"5135dc82cfb35\";a:6:{i:0;a:2:{s:4:\"time\";s:10:\"1362487399\";s:7:\"content\";s:454:\"typo:
    (dthink of the Speech member attribute as a kind of standard attribute) --> think
    of the Speech....\n\nthe most \"ugly\" part --> the \"ugliest\" part\n\nThe Speech
    layer is populated by several abstract types with dozens of standard attributes,
    and making the application programmer to provide getters and setters for *then
    \  --->  *them\n\n--------------------\nEstá muy bien. Fácil de entender, incluso
    para mí, que no tengo ni idea de escala. Congrats!\";}i:1;a:2:{s:4:\"time\";s:10:\"1362488374\";s:7:\"content\";s:1143:\"Categoría:
    añadir\n\nEstá todo bien, se entiende muy bien y seguro que les abre los ojos
    a muchos en signo de aprobación!\n\nTypos:\n\n[...] in popular scala libraries
    such as scalaz [...] -> scala / Scala\n[...] case classes and elliminating all
    this boilerplate? [...] -> elliminating / eliminating\n[...] you can find it on
    Github. [...] -> Github / GitHub\n[...] think of twitter: [...] -> twitter / Twitter\n[...]
    The following snippet uses the twitter [...] -> twitter / Twitter\n[...] The twitter
    layer simply [...] -> twitter / Twitter\n[...] (dthink of the Speech member attribute
    as a kind of standard attribute) [...] -> dthink / think\n[...] of the twitter
    types – using case classes [...] -> twitter / Twitter\n[...] Now, this is the
    most “ugly” part: [...] -> the most \"ugly\" / the \"ugliest\"\n[...] how the
    Speech and twitter layers [...] -> twitter / Twitter\n[...] If you want to elliminate
    that warning, [...] -> elliminate / eliminate\n[...] applications of the updatable
    package in latter posts, [...] -> latter / other (latter es \"último\", cuando
    te refieres a \"el primero de... el último de... sería mejor elegir otra palabra)\";}i:2;a:2:{s:4:\"time\";s:10:\"1362492251\";s:7:\"content\";s:447:\"Hola
    Juanma!,\n\nen primer lugar hay algunos errores en palabras. Creo haber visto
    un \"elliminate\" y un \"dthink\".\n\nPor otro lado he estado leyendo el artículo
    y creo que está bastante bien eso de poner el problema, una posible solución y
    luego meter los updatables como una buena alternativa. No soy un experto en estos
    follones pero tras leerlo he entendido prácticamente todo.\n\nFelicitaciones desde
    el otro lado de la pared.\n\nUn saludo,\n\nDavid\";}i:3;a:2:{s:4:\"time\";s:10:\"1362495492\";s:7:\"content\";s:339:\"Creo
    que está muy bien. La única pega, que puede dar la impresión de que utilizar las
    case classes no es realmente tan pesado (por la pinta que tienen los snippets).\n
    \nY alguna corrección ortográfica:\n- \"def macros\" => \"the macros\"?\n- \"dthink\"
    => \"think\"\n- \"The weakBuilder macro generate\" => \"generateS\"\n- \"ellimilate\"
    => \"eliminate\"\";}i:4;a:2:{s:4:\"time\";s:10:\"1362495945\";s:7:\"content\";s:203:\"Está
    muy bien, me ha gustado bastante. El único typo que creo que hay es \"is tough
    work\". ¿No sería \"is a tough work\"? Por lo demás, mi inglés no da para más,
    me parece que está muy bien escrito.\";}i:5;a:2:{s:4:\"time\";s:10:\"1362496075\";s:7:\"content\";s:20:\"
    No entiendo nada!!!\";}}}"
  _publicize_pending: '1'
author: Juan Manuel Serrano
permalink: "/2013/03/07/updating-immutable-objects-in-generic-contexts/"
---
Immutability is one of the hallmarks of functional design, and writing idiomatic programs in Scala highly relies on manipulating immutable objects. Now, if we don't have mutable fields (aka *vars*) ... how can we update objects in a convenient way? Scala provides so-called *case classes* which have a *copy* method with the required functionality. And we can also use *lenses*, a higher-level abstraction that you can find in popular Scala libraries such as <a href="https://github.com/scalaz/scalaz">scalaz</a> and <a href="https://github.com/milessabin/shapeless">shapeless</a> (you can find a macro-based implementation in the <a href="https://github.com/retronym/macrocosm">macrocosm</a> project as well). Nevertheless, all these implementations build some way or another upon case classes as the basic updating mechanism.

Now, sometimes writing case classes for your specification traits is cumbersome, since it involves a lot of boilerplate. And this problem is specially exacerbated in the presence of inheritance hierarchies, where traits get also polluted with getters and setters. Wouldn't it be nice if we found some way of automatically deriving case classes and eliminating all this boilerplate? Well, this is a question for macros, and *<a href="http://docs.scala-lang.org/overviews/macros/typemacros.html">type macros</a>, *in particular. But type macros are still a pre-release feature of Scala. So, what can be done with <a href="http://docs.scala-lang.org/overviews/macros/overview.html">*def macros*</a> alone? We have developed a library that exploits def macros in combination with reflective calls to eliminate the need of writing implementation classes. And it allows the programmer to update immutable objects in generic contexts with a minimum overhead. This library is called <a href="https://github.com/hablapps/updatable"><strong>org.hablapps.updatable</strong></a> and you can find it on GitHub. Before explaining its functionality, though, let's illustrate the problem with a simple example, and let's solve it using case classes.

### <strong>*The problem ...*</strong>
We will illustrate the kind of updating problem we have to deal with by considering a design problem in the implementation of Speech itself. Among other things, our DSL offers to programmers an abstract layer which implements generic types and state transformations that can be reused across any kind of social domain. For instance, the layer includes *interaction contexts *and *agent roles*, and the *play* transformation which adds a new agent role within some context. We want to implement interaction contexts and agent roles as immutable objects and be able to reuse the *play *transformation as-is, across any application domain. For instance, think of Twitter: there you find accounts, followers, tweeters, and many other concepts. We can think of accounts as the contexts where tweeters interact with their followers; and following someone would involve *playing *a new follower role within their account. As another example, think of courses as contexts of interaction for student and teacher agents, and some student enrolling some course: this action can also be implemented with the help of the *play *action.

### *<strong>... Solved using case classes</strong>*
Our design problem can be understood as a particular example of the*<a href="http://www.scala-lang.org/sites/default/files/odersky/ScalableComponent.pdf"> family polymorphism</a> *problem, which can be easily solved in Scala using abstract types and the *cake **pattern. *Accordingly, the Speech abstraction layer can be understood as a family of types which vary together covariantly in each application layer. In particular, our implementation will be structured in three basic layers:


* An abstract layer (the *S**peech* layer) which provides generic implementations of interactions contexts, agents, and generic transformations, in terms of traits and generic methods.
* An application layer which provides specific implementations of domain-dependent concepts in terms of traits that extends the corresponding generic traits.
* Another application layer which provides the implementation of domain-dependent traits, in terms of case classes.

The following snippet represents an implementation sketch of the first layer:

```scala
trait Speech {
  trait Interaction[This <: Interaction[This]] { self: This =>
    type Member <: Agent[Member]
    def member: Set[Member]
    def member_=(agent: Set[Member]): This
  }

  trait Agent[This <: Agent[This]] { self: This =>
  }

  def play[I <: Interaction[I]](i: I)(a: i.Member): I =
    i.member = i.member + a
}
```

Here, the Speech layer just implements two traits for the *Interaction *and *Agent *types, as well as the *play *transformation. Note that the *play *method must work for any type of interaction and agent, and we don't want to forget the exact type of interaction once we call the method. Hence, the method is parameterized with respect to some interaction type *I*. Now, the agent to be played within that context must be compatible with the interaction type, i.e. we can play *followers *within Twitter accounts, but not *students. * To account for this constraint, we declare an abstract type *Member *in the *Interaction *trait and exploit dependent types in the *play *signature. How do we add the new member agent? We need a setter, of course. And this setter must also return the specific type of the interaction **(again, to avoid type information loss). For that purpose, the trait is parameterized with the *This *parameter, following the standard <a href="http://www.scala-lang.org/node/6649#comment-27573">solution</a> to this problem. Last, note the updated sentence in the *play *method: it's as if *member *was a *var*. But it's not, it's simply that we named the getter and setter according to the *var *convention.

How do we reuse this abstract layer? The following snippet uses the Twitter domain to illustrate reuse of the Speech layer.

```scala
trait Twitter extends Speech {

  trait Account extends Interaction[Account] {
    type Member = Follower
  }

  def Account(members: Set[Follower] = Set()): Account

  trait Follower extends Agent[Follower] {
  }

  def Follower(): Follower
}
```

The Twitter layer simply extends the Speech traits and sets the abstract members to the desired values. Of course, a real implementation will include additional *domain-dependent* attributes, methods, etc., to the *Account *and *Follower* traits (think of the Speech *member* attribute as a kind of *standard* attribute). Note that we also included factory methods for the *Account* and *Follower* types. In a real implementation, it is more than likely that we will need them. And we don't want to commit to any specific implementation class, so we declare them abstract. The next portion of the cake will provide the implementations of the Twitter types - using case classes:

```scala
  trait TwitterImpl { self: Twitter =>

    private case class AccountClass(member: Set[Follower]) extends Account {
      def member_=(agent: Set[Follower]) = copy(member = agent)
    }

    def Account(members: Set[Follower] = Set()): Account = AccountClass(members)

    private case class FollowerClass() extends Follower {
    }

    def Follower(): Follower = FollowerClass()
  }
```

Now, this is the "ugliest" part: we had to provide case classes for all the application traits, and the getters/setters for all of their attributes (standard and non-standard). In this simple example, we just have the "member" attribute, but we may have dozens in a real implementation. This implementation layer must also provide implementations for factory methods, which happen to be the only way to create new entities (note the *private* declaration of case classes).

The following snippet exercises the above implementation:

```scala
  object s extends Twitter with TwitterImpl
  import s._

  val (a, f1, f2) = (Account(), Follower(), Follower())

  // test _=
  assert((a.member = Set()).member == Set())
  assert((a.member = Set(f1, f2)).member == Set(f1, f2))

  // test play
  val a1 = play(a)(f1)
  assert(a1.member == Set(f1))
```

##  *... <strong>Solved using the </strong>*org.hablapps.updatable*<strong> package</strong>*
The major structural change to the above implementation is that we don't need the *case* *class* layer. Thus, we may qualify the following implementation as *trait-oriented*. Let's see how the Speech and Twitter layers are modified:

```scala
trait Speech {
  trait Interaction {
    type Member <: Agent
    val member: Set[Member]
  }

  implicit val Interaction = weakBuilder[Interaction]

  trait Agent {
  }

  implicit val Agent = weakBuilder[Agent]

  def play[I <: Interaction: Builder](i: I)(a: i.Member): I =
    i.member := i.member + a
}
```

The first noticeable change is that ... we don't need getters and setters! We just declared our attributes using *val*s. And the implementation of the *play *method has not been excessively complicated: we just substituted the "=" operator for the new operator ":=", and included through its signature evidence that the type parameter *I* has an implementation of the *Builder* type class. Instances of this type class can be understood as factories that allow programmers to instantiate and update objects of the specified type in a very convenient way. In particular, the Builder type class enables an implicit macro conversion which gives access to the ":=" operator. All this in a type-safe way.  In a sense, builders play the same role as case classes played in the previous implementation. But there is a crucial difference: builders are created automatically through the *builder *macro, as shown in the following snippet of the second layer:

```scala
  trait Twitter extends Speech {
    trait Account extends Interaction {
      type Member = Follower
    }

    implicit val Account = builder[Account]

    trait Follower extends Agent {
    }

    implicit val Follower = builder[Follower]
  }
```

The only difference in this layer with respect to the case class implementation is that no method factories are needed, since builders play that role. Now, if you come back to the previous snippet you will also notice *weakBuilder* invocations for types *Interaction *and *Agent*. Certainly, we don't need strict builders for these types, since they are "abstract". However, builders also provide attribute reifications, and we certainly want an unique reification for the *member *attribute. The *weakBuilder *macro generates the corresponding reifications. The following snippet shows how to access reified attributes, and mimic the functionality included in the case class implementation.

```scala
object s extends Twitter
import s._

// test reifications
assert(Account.attributes == List(Account._member))

// create instances
val (a, f1, f2) = (Account(), Follower(), Follower())

// test _=
assert(a.member == Set())
assert(((a.member += f2).member -= f2).member == Set())

// test play
val a1 = play(a)(f1)
assert(a1.member == Set(f1))

println("ok!")
```

Note that the factory method provided by the *Account* builder include default parameters as well. These default parameters are defined through the *Default* type class. The companion object of this type class comes equipped with default values for common Scala types, but you can also provide default values for your own specific types. As you can see, the default value defined for types *Set[_] *is the empty set.

Concerning the rest of the snippet, we also illustrated the use of the '+=' and '-=' operators. Basically, these operators allow the programmer to specify updates of multivalued attributes specifying only just the element to be added or removed to the collection. To be able to use these operators, the* *type constructor of the attribute type must implement the *Modifi**able *type class. Currently, the updatable package offers modifiable instances for ***Option *and **any kind of *Traversable.*

## ... <strong>*But be careful with non-"final" attributes*</strong>
Let's suppose that we changed slightly the signature of the *play* method:

```scala
def playAll[I <: Interaction: Builder](i: I)(ags: Set[i.Member]): I =
  i.member := ags
```

Is this type-safe? Certainly not, since the actual type *I *may have refined the *member *attribute to a proper *Set *subtype. For instance, actual type *I *may have overridden the *member *declaration to a *ListSet,* while actual argument *ags *may be a *HashSet. *The source of this problem is that the *member* **attribute is not "final", in the sense that it can be overridden. We will consider an attribute as "final" if every component which is part of its declared type is a final class or refers to an abstract type.

We may have forbidden non-final attributes to be used as part of update sentences, but this would rule out the above implementation of the *play method*, which is perfectly safe: in that case, there was no problem because the '+' operator is defined by the different subtypes of the trait *Set**. *So, we ended up deciding to just emit a warning if non-final attributes are used by updating sentences *in a generic context*. If you want to eliminate that warning, you can always make the attribute declaration final with the help of new auxiliary abstract types. For instance, look at the following snippet: the *member* declaration now refers to a new abstract type *MemberCol[_]*, which forces us to change the declaration of the *playAll *method in such a way that the actual type of the attribute must now be taken into account.

```scala
trait Interaction {
  type MemberCol[x] <: Set[x]
  type Member <: Agent
  val member: MemberCol[Member]
}

def playAll[I <: Interaction: Builder](i: I)(ags: i.MemberCol[i.Member]): I =
  i.member := ags
```

<strong>UPDATE: </strong>the above snippet has been changed to fix a mistake detected by Eugene Burmako. Thanks Eugene!

## <strong>*In hindsight ...*</strong>
We spent a considerable amount of time in the design and implementation of the updatable package, but it was worth it. The *Speech *layer is populated by several abstract types with dozens of standard attributes, and making the application programmer to provide getters and setters for them, for each of the application types, is a tough work.

But we also found the updatable library useful for other parts of the Speech platform: for instance, we exploit it to facilitate the serialization of JSON objects, so that we can automatically generate a serializer for buildable types (i.e. instances of the *Builder *type class). We will tell you about this and other applications of the updatable package in following posts, paying particular attention to macro issues.

But there is still a lot that could be done ... besides fixing bugs, of course ;). For instance, we currently require traits to have all of its type members defined in order to generate a builder for it, and it would be nice to relax this constraint. Also, we may extend the updating operator := to cope with nested updates (similarly to what you can achieve with lenses). And we may add support for Union-like types, try to use type macros, etc. We warmly welcome any comment, suggestion for new functionality, corrections, ... and any other kind of help. Enjoy it!

