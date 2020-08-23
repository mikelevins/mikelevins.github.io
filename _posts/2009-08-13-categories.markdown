---
published: true
title: Categories
layout: post
---

There are a few things I've written that I see cited or quoted
repeatedly. The oldest is a quip about omnipotent beings that I wrote
eighteen years ago in a USENET discussion:

"Perhaps this morning there were only three Euclidean solids, but god
changed its mind retroactively at lunchtime, remaking the whole
history of the universe.  That's the way it is with omnipotent
beings."

More recent is an explanation I once gave of the concept of "natural
law" in politics. There are a couple of basic theories of law; one of
them is called "natural law". Because that phrase is also used to mean
"law of nature", people sometimes confuse the two concepts, often, I
think, because they're not aware that the legal one exists, or are not
aware of what it is. This was my attempt to succinctly summarize it in
one discussion:

"People will naturally and predictably find some methods for resolving
[...] conflicts more congenial than others. There are some classes of
conflict for which people will naturally and predictably find certain
kinds of resolutions more congenial than others. The procedures people
find more congenial will also produce the resolutions people find more
congenial. And the procedures and resolutions that people find more
congenial will tend to resemble each other across times and cultures."

Two others are more technical articles that I wrote early in the 1990s
about programming. They were published in a magazine aimed at Mac
developers, called Frameworks (now archived at
http://groups.google.com/group/alt.atheism/msg/300d7aef502eb5ba).

"Objects Without Classes" was republished by the ACM's Computer
magazine in Volume 27 , Issue 3 (March 1994) . It's a basic
description of prototype-based object systems, such as those found in
languages like Self and Javascript. Languages with prototype-based
object systems probably seemed novel to more people then than they do
now. Lots of people use Javascript now, but at that time you might
never have heard of prototype-based object systems unless you were at
Sun working on Self, or in the AI business, working with frame
languages, or at Apple working on Newton or SK8.

The topic of this post is the ideas presented in "Protocols,"
originally published in the March, 1994 issue of Frameworks, and
archived at
http://www.mactech.com/articles/frameworks/8_2/Protocol_Evins.html. It's
an article about how representation, behavior, and taxonomy are
distinct concepts that can be handled separately, despite the fact
that object-oriented languages tend to confuse them.

Those points need clarifying, I know.

By "representation," I mean how we concretely lay out data. As an
example, you can represent an array as a contiguous sequence of memory
locations, or as a table of indexes paired with pointers, or as a tree
of cells in which bit patterns are mapped to subtrees, or, of course,
in many other ways. There are circumstances in which each of the
different approaches might be advantageous. These are different
*representations* of a common abstract concept that I have here called
an "array".

By "behavior," I mean the set of operations that is defined on any
given set of values. Taking the abovementioned abstract concept of an
"array" as an example, you can use any of the mentioned
representations to support a common behavior. You can
straightforwardly implement functions that fetch an element of an
array by index, that iterate over the members of an array, that write
a new value to the array at some specified index. The implementation
and performance details will differ, of course, because the
representation differs, but the behavior—that is, the API and what it
accomplishes—is the same.

By "taxonomy," I mean the relationship between one defined set of
values and another. Types are categories of values. They are (possibly
unbounded) collections of data objects. Types naturally have subtypes;
a collection of values has a natural relation to a smaller collection
of some of the same values: the smaller set is a subtype of the larger
one. It's natural to think of obtaining the smaller set by starting
with the larger one and adding restrictions that filter out some
values. You can think of taxonomies of types, such as the class
hierarchies in languages like Smalltalk and Java, as sets of values
that begin with a very inclusive type, like Java's Object, and
develops more and more refined types by adding more and more exclusive
restrictions. Object is all the values (I know; it's not really all of
them.) Number excludes all those objects that aren't representations
of magnitudes. Integer excludes all that are not representations of
whole numbers. And so on.

These three concepts, representation, behavior, and taxonomy, are
quite distinct, but most object-oriented languages conflate at least
two of them. Some combine all three. Smalltalk classes, for example,
are representation, behavior, and taxonomy all rolled into one. The
representation is in the instance variables; the behavior is in the
methods; the taxonomy is in the subclass/superclass relations. THe
trouble with that approach is that when you ask for one of the three,
you get the other two whether you want them or not. For example, if
you create a new subclass, you inherit representation, behavior, and
taxonomy, even if all you wanted was behavior. If you create some
subclass because you want a certain taxonomic relation, you also get a
representation and a behavior that may or may not be what you need.

The claim of the "Prototypes" article was that you can treat these
three concepts separately, and by doing so develop a discipline that
makes object-oriented design and implementation feasible in any
language. Furthermore, by doing so, you can develop APIs that are
language independent. To take the "array" example, what makes it an
"array" is that you can get and set elements by index, and iterate
over them in index order. That's behavior. If representation,
behavior, and taxonomy are properly separated, you can represent
"arrays" any way you like, and you can arrange for them to be subtypes
of any type you like, and they'll still be arrays, because they
support "array" behavior. Representation and taxonomy can be treated
independently as well, but not if your tools insist on conflating
them.

I still think that's true, and useful, but I was always interested in
codifying the ideas of that article more concretely. I was interested
in working with an object system designed around clearly
distinguishing representation, behavior, and taxonomy.

One language that pretty much nails it is Haskell. In Haskell,
representation is the province of datatypes. Behavior is handled by
functions. Taxonomy belongs to typeclasses.

For those unfamiliar with Haskell, the terminology is probably
confusing. Without some experience with Haskell, it's probably not at
all clear how and why "datatypes" and "typeclasses" are
different. Briefly, a Haskell datatype is a named description of an
arrangement of data. A typeclass is a named description of a set of
operations. You can make a datatype into a member of a typeclass by
defining functions that implement the operations specified by the
typeclass.

This is pretty close to what I was talking about in "Protocols;" it's
probably closer than anything else I've seen to language support for
those concepts. The next closest thing, I think, is the style of
object system exemplified by CLOS (the Common Lisp Object System). In
CLOS, classes describe representation and taxonomy. Generic functions
and methods describe behavior. It's easy and convenient to describe
behavior in CLOS independently of representation or taxonomy. CLOS
does still conflate representation with taxonomy, though it provides
facilities for modifying itself that enable you to work around that
conflation if you really want to.

In February of 2009 I had been using the new Lisp dialect, Clojure,
for about five months on some projects, and was generally pretty happy
with it. One area I wasn't happy with, though, was Clojure's approach
to types and polymorphic functions. Clojure doesn't really have a type
system as such. It has a small number of well-chosen and well-designed
types, and it can transparently use Java's types (Clojure's main
implementation is built on the JVM). For most uses, these two
facilities are more than sufficient, and they work very well. They
fell a little short of what I wanted for some of my work, though. One
of my projects requires ways to specify a large number of structured
data elements with taxonomic relations. The set of elements is open,
and expected to grow over time, so I need a convenient way to add
descriptions of new ones, and ways to ensure that relevant APIs are
defined to work in the proper way over all the values currently
described, and over all those that are yet to be described.

Clojure gives me a lot of what I want to support this scenario, but
not all of it. Clojure's maps provide a great way to describe
structured data, but not a great way to place restrictions on it. I
can say that Foo has fields A, B, and C, but there's no convenient way
to say that A is an automobile and B is a hypotenuse; nor is there a
convenient way to say that a Foo has A,B, and C, and nothing else.

I was even more dissatisfied with the facilities that Clojure provided
for defining taxonomy. Clojure's documentation, and its justifiably
enthusiastic users, make much of the fact that Clojure's derive
function and its polymorphic MultiFns make it possible to define
arbitrary taxonomies. That's true for small values of "arbitrary". You
can easily construct any taxonomy, as long as it's a taxonomy that
Clojure was designed to easily construct. Mine wasn't.

The details of the problems don't matter. My main point is that I ran
into a couple of insoluble problems with Clojure's MultiFns, derive,
and hierarchies, and, after some discussion, satisfied myself that the
Clojure community regarded those obstacles as features rather than
bugs. So I did what any sensible Lisp hacker does in a situation like
that: I wrote my own object system.

It began as an existence proof of an alternative way of handling
polymorphic dispatch. There was a little bit of interest in it from a
couple of people, but for the most part the Clojure community reacted
with a shrug. On the whole, they're happy with Clojure's
approach. More power to them.

As I tweaked a few things to respond to casually-mentioned
hypothetical objections, I started to like what I had. I was testing
it by reimplementing important parts of some production code I was
working on. I realized pretty early that the subsystem I was builiding
bore more than a passing resemblance to the ideas in "Protocols". I
refactored it a few times. I tried a couple of different
implementation strategies. It got faster, simpler, and more appealing
(to me, that is). Somewhere along the way, I started calling it
Categories.

I have a couple of implementations of it in Clojure now, with somewhat
different APIs. More recently, I ported it to Scheme, so that I can
try it out in a different application context. The API mutated a
little more. I'm still refactoring things to try to make the surface
API simpler and easier to understand.

Categories represents the concepts from "Protocols" pretty
straightforwardly. The basic concepts are:

Types: descriptions of how data are laid out.

Functions: operations that accept zero or more values as parameters,
and that compute and return zero or more values as results.

Domains: descriptions of relations among types.

Functions are polymorphic, and are defined in terms of domains. In
other words, a functions looks at its arguments at runtime and decides
which actual code to run based on what it sees. This is like any other
object-oriented language, as far as it goes. The distinguishing
characteristic of the system is how a function choses a method.

When you construct a function, one parameter to its constructor is a
domain. A domain contains a catalog of types, and a set of rules
(represented as functions) that describe how the types are
related. Domains can tell you things like whether a type is a member
of the domain, whether one type is a subtype of another, and whether a
method can be applied to a particular set of argument values.

Importantly, you can have as many domains as you want, and each one
can work differently. How you represent the relations among types is
entirely up to you. This means that you can pretty easily implement
any kind of dispatching you want. Smaltalk-style, Java-style,
CLOS-style, predicate dispatch—it's all good. The Categories subsystem
defines a default domain that implements a dispatch scheme very
similar to those of CLOS and Dylan (because I like those schemes and
am comfortable with them), but the default is just one domain. If you
want something different, it's easy enough to build it. Just to make
sure, I wrote implementations of Clojure's dispatch and of a
predicate-dispatch scheme. They were easy to do.

When I was first discussing these ideas, someone mentioned some
concern that it wouldn't be possible to implement such a system
efficiently, or to optimize it. In practice, it hasn't been a
problem. There is a tradeoff between exposing the API you need to
implement a domain efficiently, and making the domain API
understandable, and that's been a major issue I'm dealing with in
rewrites of Categories. It seems clear at this point, though, that
efficient implementations are doable. The trick is setting things up
so that you can write domains that are efficient without exposing a
confusing array of knobs and switches. I'm still working on striking
the right balance.

You can't get a current working version of Categories right now; I
still have its guts out on the table so I can tinker with them. Some
of its early predecessors are readily available if you really want
them, but if you read this and are really interested in Categories,
I'd counsel patience. I'm folding it into some product code right now,
and making corrections and improvements as that process reveals the
need. It's my intention to nail it down and ship a product that uses
it, then port that version back over to Clojure to support some other
work I'm doing in that language. Once I reach that point, if you ask
me for it, I'll give it to you.

Why would you want it? I dunno; you might not. I sure did, though,
enough to build it. Soon I'll see if it was a waste of time, or a
great new tool for my toolbox.
