---
published: true
title: On repl-driven programming
layout: post
tags: programming
---

Once upon a time, someone with the handle "entha_saava" posted this question on [Hacker News](https://news.ycombinator.com/item?id=23791152):

> Can someone knowledgeable explain how are lisp REPLs different from
> Python / Ruby REPLs? What is the differentiating point of REPL
> driven development?

The answer is that there is a [particular kind of programming](http://mikelevins.github.io/2020/02/03/programming-as-teaching.html)
in which you build a program by *interacting* with it as it runs, and
there are certain languages and runtimes that are designed from the
ground up to support that kind of programming.

Python and Ruby are not examples of such languages.

Why not? That's the crux of entha_saava's question, right? What are
these **repl-driven** programming systems, and what makes them
different from Python and Ruby and every other language that offers a
repl?

For that matter, what's a repl?

The word **repl** is an acronym that stands for **read-eval-print
loop**. The term comes from the history of Lisp. From the start, sixty
years ago, the standard way of working with a Lisp has been to start a
language processor, type expressions at its prompt, and wait for it to
evaluate the expressions and print their results, before prompting for
another expression. Read, eval, print. Loop.

Nowadays, repls are all the rage. Every language and its brother
offers a repl. There's a website, [repl.it](https://repl.it/), whose
entire purpose is to provide all the repls.

It doesn't actually provide *all* the repls, of course. What's
particularly ironic is that it doesn't provide either of the canonical
repl-driven development environments: Common Lisp and Smalltalk.

That brings us back to entha_saava's question: if Common Lisp and
Smalltalk are repl-driven environments, and Python and Ruby are not,
what's the difference? What do Lisp and Smalltalk have that Python and
Ruby don't?

What they have is a language and runtime system that are designed from
the ground up with the assumption that you're going to develop
programs by starting the language engine and talking to it, teaching
it how to be your program *interactively*, by changing it *while it
runs*.

I can hear the objections formulating already. I've seen them
before. Yes, every language with a repl can do some things in the
repl. Obviously that's true; if it weren't, then the repl would be
entirely useless.

Being able to do *some* things in the repl does not make an engine
into a repl-driven programming environment. What distinguishes
old-fashioned Lisp and Smalltalk environments is that you can do
*everything* in the repl. They place no gratuitous limitations on what
you can do; if the language and runtime can do it, then the repl can
do it.

For example, you can ask the current version of Clozure Common Lisp to
rebuild itself from scratch by evaluating the following expression at
the repl prompt:

    (rebuild-ccl :full t)

CCL responds by completely rebuilding itself from source.

The point is not that you would want to rebuild CCL this way all the
time. The point is that there are no artificial limitations on what
the repl can do. The full range of the development system's
capabilities is accessible from the repl.

That's one of the first things I notice when using newer, lesser
repls: I'm always running into things I can't do from the repl.

It's not just about freedom from restrictions, though. Proper support
for interactive programming means that the language and its runtime
have positive features that support changing your program *while it
runs*.

Try this in your favorite repl:

Define a function, `foo`, that calls some other function, `bar`, that
is not yet defined. Now call `foo`. What happens?

Obviously, the call to `foo` breaks, because `bar` is not defined. But
what happens when it breaks? What happens next?

If your favorite repl is Python's or Ruby's or any of a few dozen
other modern repls, the answer is most likely that it prints an error
message and returns to its prompt. In some cases, perhaps it crashes.

So what's my point, right? What else could it do?

The answer to that question is the "differentiating point" of
repl-driven programming. In an old-fashioned Lisp or Smalltalk
environment, the break in `foo` drops you into a **breakloop**.

A **breakloop** is a full-featured repl, complete with all of the
tools of the main repl, but it exists inside the dynamic environment
of the broken function. From the breakloop you can roam up and down
the suspended call stack, examining all variables that are lexically
visible from each stack frame. In fact, you can inspect all live data
in the running program.

What's more, you can *edit* all live data in the program. If you think
that a break was caused by a wrong value in some particular variable
or field, you can interactively change it and resume the suspended
function. If it now works correctly, then congratulations; you found
the problem!

Moreover, because the entire language and development system are
available, unrestricted, in the repl, you can define the missing
function `bar`, resume `foo`, and get a sensible result.

In fact, there's a style of programming, well known in Lisp and
Smalltalk circles, in which you define a toplevel function with calls
to other functions that don't yet exist, and then define those
functions as you go in the resulting breakloops. It's a fast way to
implement a procedure when you already know how it should work.

If you're a user of old-fashioned Lisp or Smalltalk systems then this
all sounds obvious to you, but that reaction is not common. Surprise
is much more common, or even suspicion: what's the catch?

The catch is that the designers of your language system had to think
that facility through in the planning stages. You don't get a decent
implementation of it by bolting it on after the fact. Breakloops need
*full* access to the *entire* development system, interactively, with
a computation and its call stack suspended in the breakloop's
environment.

Let's take another example of a facility designed to support
interactive programming. Once again, try this in your favorite repl:

Define a datatype. I mean a class, a struct, a record type--whatever
user-defined type your favorite language supports. Make some instances
of it. Write some functions (or methods, or procedures, or whatever)
to operate on them.

Now change the definition of the type. What happens?

Does your language runtime notice that the definition of the type has
changed? Does it realize that the existing instances have a new
definition? When something touches one of them, does it automatically
reinitialize it to conform to the new definition, or, if it doesn't
know how to do that, does it start a breakloop and ask you what to do
about it?

If the answer is "yes," then you're probably using a Lisp or Smalltalk
system. If the answer is "no," then you're missing a crucial element
of repl-driven development.

Remember: the point is to support programming *interactively*. You
don't want to have to kill your program and rebuild it from scratch
just because you changed a definition. That's silly; adding and
changing definitions is most of what you do! If your development
environment is going to support interactive development, then it had
better know how to keep your program running when you change some
definitions.

Old-fashioned Lisp and Smalltalk system know how to do that. There are
also a few other kinds of systems, mostly older ones, that know how to
do it.

These are not eccentric new ideas out of left field. They've been
around for half a century. They contribute materially to productivity
in interactive development.

They're what sets **repl-driven development** apart from mere
development with a repl.

Now, not every programmer prefers that kind of development. Some
programmers prefer to think of development as a process of designing,
planning, making blueprints, and assembling parts on a
workbench. There's nothing wrong with that. Indeed, a
multibillion-dollar international industry has been built upon it.

But if you prefer interactive development, if it's more natural to
you, then it can make you enormously more productive, not to mention
happier in your work.

Interactive development with a proper repl-driven environment is the
exception. Most programming is done in other ways.

As a consequence, there are a lot of programmers out there who've
never even heard of it, who have no idea that it exists. My intuition
is that some fraction of those programmers would prefer well-supported
interactive programming, and would benefit from it, if they just knew
what it was.

Maybe if enough programmers are exposed to that style of programming
then we'll begin to see new tools that embrace it.

