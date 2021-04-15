# Q: The Tool of Functional Thought

## About this series

This series of posts describes our experience while learning
[*q*](https://code.kx.com/q/learn/startingkdb/language/) and *kdb+* by Kx
Systems. Although q is also a functional programming language, it has many
features that make it different from other conventional languages such as
Haskell, OCaml, etc. Given this situation, we'll try to provide an overview of
q basics, connecting the missing pieces to our previous knowledge on functional
programming, using Scala and Spark to guide the explanations. We hope it also
works in the opposite direction, so q programmers can also benefit from this
introduction, which will be divided into the following posts:

1. Q as an (impure) functional language
2. Q as an array processing language
3. Q as a query language for kdb+

The first post introduces q as a functional language, showing the main q
features that are already familiar to the conventional functional programmer
which could be used from the very first day. The second post will put focus on
q as an array processing language, which is probably the main source of q
weirdness, so we'll try to connect it to existing theory on the functional
paradigm. Finally, the last post will introduce q and kdb+ as a query language
and a column-oriented database, respectively, to make the data engineer happy.

### Why q?

Each day, we set aside a time to experiment with new technologies, and we are
especially interested on functional languages. A colleague from the Scala
community pointed us towards q. The following items summarise the alleged
language benefits why we decided to give it a go:

- Q is fast, sooo fast
- Q is a functional query language
- Q is a well-founded language that relies on APL
- Q is highly demanded in the financial industry
- Q is quite a challenge

After a few months of reading q material and coding, we can confirm that none
of the previous items is a myth. Although we feel that we still have a long way
to master q/kdb, we are confident that we have now a good perspective on how
hard it is to learn this language. In fact, reflecting this experience is
perhaps the major contribution of this series of posts. Having said so, we're
ready to go now!

