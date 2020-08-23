---
published: true
title: Programming as teaching
layout: post
tags: programming
---


Most programming uses an approach that we might call **programming as
carpentry**. We start with some idea of an artifact we want to
build. We analyze the idea into a set of needed parts. We set to work
on our workbench to craft the needed parts and assemble them into a
finished product, then we examine and measure the result to determine
how close we came to our goal.

Next we repeat the process to correct details that have fallen short
of what we want, either because we made mistakes in construction, or
because our original plan was wrong in some way.

There's a different way to approach programming. We might call it
**programming as teaching**. In this approach, we begin by starting up
a runtime that already knows how to be a working program; it just
doesn't know how to be our particular application. By talking to the
runtime interactively, we incrementally teach it the features we need
it to have. When it knows how to provide all those features, we're
done. We can save the accumulated changes to an artifact, and that
artifact becomes our product.

It's less like nailing together an artifact on a workbench, and more
like teaching a student a new set of skills.

Programming as carpentry is far better known and more widely used than
programming as teaching. I'd venture to say that most programmers
aren't even aware that programming as teaching is an option. That's a
shame, I think, because for some people it's a better option.

I don't claim that the teaching paradigm is absolutely and objectively
better. I claim only that it's better for some people. Take me, for
example: programming as teaching makes me much happier, faster, and
more productive in my work.

I'm not the only one. There are many other programmers who prefer the
teaching paradigm. The reason it's an obscure minority paradigm is
that there are even more programmers—vastly more—who are familiar with
programming as carpentry.

The fact that most programmers don't seem to even know that the
teaching paradigm exists suggests to me that if it were more widely
known, then more people would use it. What are the odds that everyone
who prefers the obscure paradigm is already using it, when most people
don't know it exists?

In some sense, it doesn't matter. The great majority of software is
developed in the carpentry paradigm, and the industry prospers. Is it
really important to evangelize a different paradigm? Probably not, at
least for the sake of the industry as a whole.

On the other hand, I think it is important for the sake of us as
programmers. If I hadn't been exposed to programming as teaching, I
probably never would have known how happy and productive I can be in
my work. A world in which I know about the teaching paradigm is a
better world, at least for me. If there are other programmers who
would prefer the teaching paradigm, and who don't know about it, then
there's a better world awaiting them. All they need is to find out
about it.

So if programming as teaching is such a great idea—even if it's only
great for a minority of programmers—why isn't it better known? I think
it's because programming as teaching requires some tools over and
above what programming as carpentry requires. You have to expend extra
work to make those tools available, and you have to know about them
before you can decide to do that.

You can build a solid carpentry workbench without knowing anything
about the teaching paradigm. The reverse isn't true. A good
programming-as-teaching system is going to need all the tools of the
carpentry workbench. It's going to need parsers and data structures
and code-walkers and code generators. It's going to need file I/O and
performance tools and debuggers, and so forth. It's going to need all
that stuff, but it's going to need some other stuff, too.

Programming as teaching means starting the application running before
it's defined, then defining and redefining features while it runs. It
means inspecting the contents of its memory, stopping control
structures in the middle of executing, and inspecting and changing and
redefining their dynamic context. It means updating the definitions of
functions that are pending on the stack and of datatypes that are used
by existing instances, and relying on the application to keep working
reasonably while you're tinkering around in its guts.

All of those features require substantial runtime support, and if you
want it to work well, the runtime needs to be designed from the start
to support it.

Smalltalk systems have thorough support for that kind of
programming. So do Common Lisp and other old-fashioned Lisp
systems. Outside Smalltalk and Common Lisp and, to some extent FORTH
systems, there aren't many development systems with the full suite of
programming-as-teaching features.

Some of these features exist in some form in modern
programming-as-carpentry systems, but they don't amount to a
programming-as-teaching system. To build a working
programming-as-teaching system, you need to design the whole runtime
from the ground up to support it. Let me give you an example of what I
mean.

The ANSI Common Lisp standard defines a generic function named
UPDATE-INSTANCE-FOR-REDEFINED-CLASS. When the runtime detects a value
whose class has been redefined since it was instantiated, it
automatically calls this function to restructure the value so that it
conforms to the new definition. In effect, it retroactively makes the
value an instance of the new definition instead of the old one.

You can specialize UPDATE-INSTANCE-FOR-REDEFINED-CLASS to correctly
reinitialize instances to conform to their new definitions. If you
don't, then the runtime drops you into an interactive session in which
you can provide the new initialization interactively. You can then
resume execution and the affected value behaves as if it was
originally instantiated from the updated definition.

If you're coming from programming-as-carpentry, you might reasonably
wonder why you would ever want a function like that at all, much less
why you would want it to be part of a language standard. But Common
Lisp was designed by experienced Lisp users and implementors. They
were steeped in the practice of programming as teaching (though they
didn't call it that; they just called it "programming").

To the designers of Common Lisp, the normal way of developing a
program was to start the Lisp and then teach it, definition by
definition, how to be the program they wanted. They wouldn't expect to
have to restart their Lisp just because they redefined something;
that's silly. Redefining things was nearly all they did! No, their
expectation was that you redefine things and keep going. It's the job
of the runtime to adapt in a reasonable way to the changes that you
tell it about. It's also the job of the runtime to ask you for help
when it doesn't know what to do.

UPDATE-INSTANCE-FOR-REDEFINED-CLASS is one element of a whole standard
protocol defined by ANSI Common Lisp for changing and redefining
things while the program you're developing continues to run. The
standard describes facilities for defining classes and functions,
updating bindings, catching errors by dropping into an interactive
session where you can inspect and change everything about the dynamic
environment, then tell the function where the error occurred to resume
execution with the new definitions, and generally for accomplishing
every aspect of building and deploying a program through an
interactive conversation with the running program itself.

Smalltalk systems have the same kind of design.

This kind of development environment is a whole that is greater than
the sum of its parts. To get it, you have to design the language and
runtime from the start to support it. You can't convert a carpentry
system to a teaching system by patching in one feature at a time.

Very few development toolchains support the full suite of
programming-as-teaching features. Most of the ones that do have been
around for a long time. Newer languages and runtimes are mostly
designed without any knowledge or understanding of the whole-system
programming paradigm that is embodied in old Lisp and Smalltalk
systems. Even some newer Lisps have been designed without that
understanding.

I worry sometimes that programming-as-teaching is fading away, and I
feel sad that my favorite paradigm might someday disappear altogether.

I don't think that if it does it'll be the end of software
development, or anything apocalyptic like that. I do think that I'll
miss it when it's gone, and I think it'll be sad if new generations of
programmers never have the opportunity to experience it.

There's something magical about gradually turning a program into the
application you want by talking to it while it runs. For some fraction
of programmers, it's the best way of working. I'd hate to lose it.
