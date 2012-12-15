---
layout: post
title: "The Value is the Boundary"
date: 2012-12-15 15:31
comments: true
categories:
 - Code Organization
 - Functional Programming
 - Mocking
 - Ruby
 - Testing
---

*Author's Note: I do not take credit for the phrase used as the title of this
blog post. It comes from Gary's Ruby Conf 12 video recording entitled
<u>Boundaries</u> mentioned below and in the previous blog post.*

In the last blog post [Testing
Trade-offs](/blog/2012/12/08/testing-trade-offs/), I talked about the
difficulties of verifying the decisions and dependencies of our classes with the
current mainstream testing methodologies. Based on a recorded conference talk by
[Gary Bernhardt](https://twitter.com/garybernhardt), the focus was on
effectively testing the logic a class contains and the dependent collobrators it
takes in for coordinating with other classes and objects to perform its
responsibilities. Mixing the two concerns in the same object definition requires
utilizing both isolated unit testing and integration/integrated testing in order
to adequately test cover the class. However, code designed this way seems to
play to the weaknesses of each testing strategy just as much as it plays to
their strengths (please see the previous blog post if you would like more
details on that discussion).

Today let's go one step further and talk about ways that we could more cleanly
separate the concerns of decisions and dependencies, with the hope that we can
create objects that better lend themselves to one type of testing over the
other. Gary proposes that a such a codebase could have better modularity,
scalability, and even concurrency. I assert that your code will also be more
maintainable and extensible as well. Most of today's content will take a lesson
from the functional programming paradigm, including practices they have espoused
for decades.

Frictionless Isolated Unit Testing
----------------------------------

If you wanted to test the mathematical addition operator (i.e., the plus sign
`+`), what frameworks, tools, and/or coding tricks do you have to employ to
sufficiently isolate it from all other concerns and objects? Absolutely nothing!
There are no dependencies to mock or stub; it isolates for free. Why is that?
Gary cautions against assuming it is because the addition operator is simple and
lacks complexity. He digs deeper in order to identify two properities the
implementation of plus sign exhibits that allows it to be naturally isolated.

The first property is that the operator takes values as arguments and returns
new values as output without any mutation. The second property is that the
operator requires no dependencies in order to perform its computation and logic.
Thus there is nothing to mock or stub when testing it, which was the major
weakness of isolated unit testing. Also because of the lack of dependencies, no
integrated tests are required in order to better test how the operator will
behave in a production environment where there are no mocks and stubs. To test
the addition operator we merely need to write simple pass-values-in,
assert-value-out tests with no extra setup required.

As Gary applies these concepts to existing code, we notice a few changes. Pieces
of domain logic and pieces of code that coordinate dependencies are separated
from each other, broken out into new objects created for a single purpose and
responsibility. The nature of the communication between objects also changed,
with values (inputs and outputs) becoming the boundary between objects instead
of the emphasis being on several synchronous method calls. Value objects focused
on data (and not behavior) become the new contract between collaborating
classes.

To Be Continued...
------------------

You may notice that many of these concepts have a functional programming
influence. The properties of immutability and focus on data at the boundaries
allow us to write code that isolates very easily and lends itself very well to
isolated unit testing that is simple and not brittle. It is very good at
verifying the domain logic and decision paths of our objects. Of course, we
can't write the entirety of our codebase in this manner with no dependencies
ever. Next time I will discuss a code architecture that Gary proposes which can
utilize this style of code married with some more imperative glue code that
coordinates the dependencies in the system. We will also discuss testing those
portions of code as well.
