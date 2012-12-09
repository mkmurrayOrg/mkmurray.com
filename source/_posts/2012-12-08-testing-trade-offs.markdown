---
layout: post
title: "Testing Trade-offs"
date: 2012-12-08 21:11
comments: true
categories:
 - Code Organization
 - Mocking
 - Ruby
 - Testing
---

Last week our dev team at Extend Health watched a [RubyConf 2012 video of a talk
entitled Boundaries](http://www.youtube.com/watch?v=yTkzNHF6rMs) by [Gary
Bernhardt](https://twitter.com/garybernhardt) of [Destroy All
Software](https://www.destroyallsoftware.com/screencasts) (or perhaps even
better known for the infamous [WAT lightning talk at CodeMash
2012](https://www.destroyallsoftware.com/talks/wat)). Gary proposes a very
interesting code architecture that marries the individual benefits of
immutability & mutability, functional programming & imperative object-oriented
programming, isolated unit testing & integration testing. He discusses the pros
& cons of each code design & testing decision and the trade-offs that we end up
dealing with. He suggests a potential solution that makes virtually no trade-off
and attempts to harness the advantages of each of these methodologies that
appear to be at odds with one another. I feel the idea has a lot of merit and I
highly encourage everyone to watch his presentation to get the full context and
a better logical progression of his proposal than what I can provide.

Isolated Unit Testing vs. Integration Testing
---------------------------------------------

A common practice in unit testing classes and objects is to isolate the targeted
class from its dependencies so that you can focus solely on its responsibilities
and domain logic independent of any implementation details of the dependencies.
This is typically accomplished by the use of stubs and mocks, which attempt to
control and monitor the interactions with and data return from dependencies.
There are a ton of gains afforded by this style of testing as Gary points out,
but one very large criticism of this testing methodology is that it doesn't
exercise the code in the same way it would run in production. It is not exactly
rare to have these isolated unit tests successfully pass and yet still have
problems in production.

Integration testing tends to better emulate production because it maintains the
interaction relationships between objects in addition to acquiring and passing
the data around the system in the same way. The criticism Gary puts forth of an
integrated testing strategy is that it is very slow as you begin to attempt to
cover all the code paths through the system. Consider trying to cover all
logical branches through the system, including all branches of
try/catch/finally, conditional, and looping structures. Gary suggests the growth
in code to cover these scenarios is 2^n^, where `n` is the number of branches.

Many try to get the benefits of both types of testing in order to compensate for
each strategy's shortcomings. However, they still write code that doesn't play
to each testing methodolgy's strengths. The strength of isolated unit testing is
verifying that given certain inputs the expected output is always returned. The
strength of integration testing is coordinating dependencies, making sure they
utilize and interact with the API of other objects correctly. If you design your
classes to be a mix of dependencies and logic, it becomes difficult to
effectively test cover them without writing both sets of tests for all classes
in your codebase.

So What?
--------

OK, so you are probably wondering what the point is then. This sounds like a
good plan to just be more disciplined in your test coverage, right? Well, let's
explore a way to better segregate dependency orchestration from actual logic and
behavior. This could allow us to use the right testing methodology depending on
which type of responsibility the object is meant to encapsulate: dependencies or
decisions? Gary also asserts that it will lead us down a path that could yield a
codebase with better modularity, scalability, and potentially even concurrency.
I believe you can also add better maintainability and extensibility to that
list.

Next week I will dive into how Gary suggests we can better seperate these
concerns of dependencies and decisions. Stay tuned (or go watch the video and
spoil the surprise).
