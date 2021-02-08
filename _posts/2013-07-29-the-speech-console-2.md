---
layout: post
title: The Speech Console
date: 2013-07-29 13:19:43.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Apps
- Speech
- Web console
tags: []
meta:
  _edit_last: '43713691'
  geo_public: '0'
  draftfeedback_requests: a:2:{s:25:"habla.computing@gmail.com";a:3:{s:3:"key";s:13:"51ed6b2156d68";s:4:"time";s:10:"1374513953";s:7:"user_id";s:8:"43713691";}s:32:"habla-computing@googlegroups.com";a:3:{s:3:"key";s:13:"51efa39e7ff24";s:4:"time";s:10:"1374659486";s:7:"user_id";s:8:"43713691";}}
  draft_feedback: |-
    a:1:{s:32:"habla-computing@googlegroups.com";a:2:{i:0;a:2:{s:4:"time";s:10:"1374665921";s:7:"content";s:767:" Pondría las frases "A process-oriented..." y "A DSL for..." entre comillas y quizás en un tamaño más grande o negrita.

    Haría una analogía con las "rooms" de un chat, en el párrafo de "The Speech Server as a structured communication infrastructure".

    "and this interaction [may/could] be closed immediately"

    Reduciría un poco el apartado "Speech as a language for programming..." quizás cortando en el segundo párrafo.

    Me gusta la analogía que hacía Isabel de cargar un cartucho en la consola a la hora de componer el sistema con twitter.Program. Igual se podría comentar (igual no :)

    Y en general, me parece que está muy bien, pero tengo dudas de que el que lo lea (que ya sabemos que no tiene ni idea de Speech) se pueda perder en algún punto...
    ";}i:1;a:2:{s:4:"time";s:10:"1374668670";s:7:"content";s:629:"Comentarios imm:

     1 - Esto no es un post de la consola sino un nuevo post de Speech.

    2 - Como post de Speech, no sé si hay segmentos muy especializados que lo entenderán, la mayor parte de la gente no lo creo. Si cabe es más complicado que hasta ahora.

    Creo que el problema es que la gente no vemos inmediatamente qué es comunicarse y no vemos que cuando usamos una aplicación estemos haciendo actos de comunicación regulados entre los participantes, y sin embargo tú lo das por sentado.

    3 - En relación con la pregunta por qué usar Speech  no se indica nada de la proximidad entre la funcionalidad y el lenguaje??

    ";}}}
  _publicize_pending: '1'
author: Juan Manuel Serrano
permalink: "/2013/07/29/the-speech-console-2/"
---
During these months, we have tried to explain Speech using different strategies and metaphors, with varying results. For instance, we have defined Speech as

<strong>*"A process-oriented programming language"*</strong>

or as

<strong>*"A DSL for programming the business logic of social applications"*</strong>

These definitions are precise and correct, but not very effective. In fact, the common reaction from the audience to these definitions is "WAT" ;)

*<span style="line-height:1.5;">Juan: </span>*<span style="line-height:1.5;">In this post, I will try to shed some light on these definitions with the help of ... THE SPEECH CONSOLE!</span>

*You: wat*

<i>
</i>Well, let's try it.

## *What is the Speech Console?*
Think of chat clients, such as IRC clients, instant messengers, etc. These tools allow people to communicate about any topic, almost in real-time. The Speech console can be thought of a chat client, in the sense that its purpose is essentially to allow users to communicate and engage in conversation. And, consequently, you can similarly think of the speech virtual machine as a kind of chat server which enables and mediates these communications. In fact, if we launch the Speech server with the minimum configuration possible, what we obtain is something similar to a chat server. The following program, which you can find in the *<a href="http://www.speechlang.org/documentation.php?s=GetStarted">Getting started</a> *section of the speechlang.org site, precisely do that:

```scala
object Main extends App{
  object Test extends org.hablapps.speech.web.PlainSystem
  Test.launch
}
```

Running this program will make the Speech server available at the localhost:8111 default address. Then, you can point the browser to the URL localhost:8111/console to get access to the Speech console. The following video illustrates this process and a sample session with two users.

[youtube=http://www.youtube.com/watch?feature=player_embedded&amp;v=zrE0ufY7y78]

&nbsp;

## *The Speech Server as a structured communication infrastructure*
But the Speech Virtual Machine is much more than a simple chat server, of course. Even with its minimal configuration, the Speech server allows users to structure their communications around a hierarchy of interaction contexts (akin to *chat rooms*), and their activity in terms of a hierarchy of roles played within those interactions. Thus, besides saying arbitrary things, the previous video showed how users can *set up* new interaction contexts, *join* and *assign* other users to them, say things within those contexts, and, eventually, *leave* and *close* the interactions.

The *set up*, *close*,* join*,* leave*, etc., message types are standard declarations provided by Speech that allow users to modify the interaction space. Within the Speech console, you can get help on these commands by typing "help say". But, besides saying things, users can also see what is happening (e.g. which interactions are taking place? which roles do I play?, etc.). Typing "help see" will give you explanations on how to observe the particular way in which interactions are structured within the Speech server at a given moment.

## *Speech as a language for programming a communication infrastructure*
But the major difference between the Speech server and a simple communication infrastructure is not its ability to hold structured conversations, but the fact that *it can be programmed*. To understand in which sense the Speech server can be programmed, note that the previous minimum configuration represents a state of anarchy: people can say what they want, and structure their conversations the way they like; moreover, there are no rules: someone may set up a new interaction, and this interaction could be closed immediately by any other one.

Now, think of the way people communicate in a given context. First, the shape of interactions and roles that people play, as well as the types of things they say, are commonly constrained to certain types. And the things that they can say, see and do depend on the kind of role they play, as well as on the specific circumstances in which they attempt to do it. So, people's interactions are commonly shaped and ruled ... at least to some extent, and a Speech program precisely encodes this rules so that they can be interpreted at runtime by the Speech virtual machine.

## *Simulating Twitter interaction through the Speech console*
<span style="line-height:1.5;">For instance, which are the constraints imposed by twitter on user interaction? which norms are enforced? </span><span style="line-height:1.5;">Basically, interactions within the Twitter </span><em style="color:#444444;line-height:1.5;">community*<span style="line-height:1.5;"> are shaped around member </span><em style="color:#444444;line-height:1.5;">accounts*<span style="line-height:1.5;"> and </span><em style="color:#444444;line-height:1.5;">lists*<span style="line-height:1.5;">; interacting users can only be </span><em style="color:#444444;line-height:1.5;">guests*<span style="line-height:1.5;"> or registered </span><em style="color:#444444;line-height:1.5;">tweeters*<span style="line-height:1.5;">, who can be </span><em style="color:#444444;line-height:1.5;">followers *<span style="line-height:1.5;">of other tweeters and be </span><em style="color:#444444;line-height:1.5;">listed*<span style="line-height:1.5;">; concerning the things they say, guests can only </span><em style="color:#444444;line-height:1.5;">set up new accounts*<span style="line-height:1.5;">, whereas tweeters can only </span><em style="color:#444444;line-height:1.5;">tweet*<span style="line-height:1.5;"> messages whose lengths are constrained to 140 chars, </span><em style="color:#444444;line-height:1.5;">re-tweet *<span style="line-height:1.5;">others' messages, join other tweeter accounts as followers (i.e. </span><em style="color:#444444;line-height:1.5;">follow*<span style="line-height:1.5;"> other users), etc.</span><em style="color:#444444;line-height:1.5;"> *<span style="line-height:1.5;">Concerning norms, tweeters can only follow other users if they are not blocked by them; tweets issued within some account are automatically notified to their followers; etc. These and other norms and rules are part of the specification of Twitter as a communication network, and these norms and types of interactions, roles and message types, can easily be </span><a style="line-height:1.5;" href="https://github.com/hablapps/app-twitter">programmed in Speech</a><span style="line-height:1.5;">.</span>

Let's suppose that the *org.hablapps.twitter.Program*  trait* *implements these types and norms; then, you can tell the Speech server that it must manage user interaction according to the structure and rules of twitter, with a simple mixin composition:

```scala
import org.hablapps.{ speech, twitter }
object TwitterWeb extends App {
  object System extends speech.web.PlainSystem with twitter.Program {
  System.launch
}
```

Once you execute this program, you can launch the Speech console and test different twitter scenarios. The following video shows the following one:


* First, a new twitter community is created (named "habla")
* Then, a guest user enters the community and decides to register himself as a Twitter user (@serrano)
* Another twitter account is created for tweeter @morena; this time the account is private
* @morena follows @serrano, so that she receives whatever serrano tweets within his account
* @serrano attempts to follow @morena, but her account is private, so the "following" declaration is kept pending for approval
* @morena allows @serrano to follow her
* Eventually, @morena decides to unfollow @serrano and she also fires his follower role within her account

[youtube=http://youtu.be/r9cOrUxtTW4]

<em style="color:#000000;font-size:1.8em;line-height:1.5em;">In sum ...*

In the light of all this, what can we say about the purpose and proposition value of Speech? First, the Speech console shows that Speech is only suitable for programming software which aim at managing user interaction. More specifically, Speech allows programmers to implement the structure and rules that govern the interactions of users within the application domain, i.e. the joint activities (or processes) carried out by people within a given social context. Thus, we can think of Speech as a (social) <strong>process-oriented programming language</strong>. Moreover, Speech is not a general purpose language but a domain specific language: in fact, Speech is a language that allows us to program a structured communication infrastructure, not a general purpose machine. Last, since the business logic of social applications largely deal with the kind of interaction requirements addressed by Speech, it can be described as a <strong>DSL for programming the business logic of social applications. </strong>

Why should we program social applications in Speech? First, because programmers don't have to start from scratch but from a programmable communication infrastructure (i.e. less code needed); and, second, because the Scala embedding of Speech allows us to implement the structure and rules of interactions in a very concise and understandable way. We strive to achieving minimum levels of accidental complexity, so that functional requirements can be implemented in the most direct way possible. In next posts, we will tell you about new improvements in the Speech embedding.

Have a great holidays!

