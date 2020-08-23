---
published: true
title: Gambit considered helpful
layout: post
---

I've just packed up Delectus(tm) 1.0 Alpha 2 and shipped it off to my
testers. Delectus is an application for keeping track of collections
of things. The inspiration for it was an ancient and hoary program my
mother used to keep track of her movies. She has around a thousand
movies, the majority of them still on videocassettes. Each cassette
has a catalog number on it, and she uses a database program to
associate those numbers with information like the title, star, genre,
and so forth, so that she can quickly find a particular movie in her
stacks of shelves. Recently the software, which is over twenty years
old, finally became obsolete. As we were looking for a replacement, I
decided I'd just write one, and that became the genesis of Delectus.

The basic design of Delectus is very simple. A Delectus document
stores a list of rows and a list of columns. Each column has a text
label. Each row contains a list of text fields. The number of fields
is the same in every row, and it's also the same as the number of
columns. So you can create a document with the columns "Title",
"Genre", and "Star", and every row will then have three fields. You
can make a new row and put the title of a movie into the "Title"
field, the star into the "Star" field, and the genre into the "Genre"
field.

Delectus doesn't decide for you how many columns to create or what the
names will be. Instead, a new Delectus document has no rows and no
columns. You can add new rows and new columns at any time, and you can
delete them at any time, too. This flexibility makes it easy to set up
your databases any way that is convenient for you. You can sort a
document's entries, forward or backward, by clicking the heading of a
column. If every entry in a column is a number, Delectus detects that
automatically, and sorts by number instead of lexicographically.

Delectus uses a trash can, like OS X's trash can. When you delete a
row or column, it's not really gone, but tossed into the trash. Click
a trash button, and Delectus shows the trash to you, integrated in the
document, but displayed in a different color, so you can easily tell
which items are deleted. Empty the trash, of course, and the deleted
items are gone forever.

Delectus is mainly written in Scheme. The Scheme compiler I'm using is
Gambit-C. A few weeks ago, I wrote a post about the happy discovery
that Gambit-C made it easy to work with Mac OS X's Cocoa frameworks. I
said Gambit was a great platform for Cocoa development. That opinion
drew some fire, but I haven't changed it. To the contrary, I've grown
happier with Gambit as I've developed Delectus.

Some people grumbled that I'm not allowed to call it a great platform
if I still have to deal with C code. Now, that's silly. You know, I
actually had a Lisp system once in which I never had to deal with C
code. It was called a Symbolics Lisp Machine, Every other Lisp I've
used--and I've used a lot of them in twenty one years of
programming--has had to deal with C in one way or another, at one time
or another. Heck, even when I was writing Lisp code for an operating
system written in Lisp (an unshipped version of Apple's Newton), we
had to deal with C code. If your eyes are damaged by the sight of
semicolons, a hard road awaits you. C code is in the underpinnings of
every system you are ever likely to use. At least Gambit makes it easy
to wrap our beloved parentheses around it, so that the hothouse
flowers in our midst are protected from the hot exhaust bits spewing
forth from the C code beneath our feet.

The other common complaint seems to be that I made a grammatical error
in using the adjective "great" to modify a set of development tools
that does not include an IDE. To paraphrase Yoda, "IDEs not make one
great."  Sure, there have been great IDEs. Smalltalk-80 was a great
IDE. Xerox Interlisp was a great IDE. Most IDEs are not great. Most
IDEs are *going* to be great, someday. That is, they're going to be
great, right up to the point where the weight of the implementation
collapses the envelope of optimism providing their buoyancy, and they
crumple into a heap of random features, slowly and inexorably sinking
under their combined mass.

IDEs are large, complicated systems that require serious design and
engineering. They are major cost centers to anyone who is not in the
business of selling IDEs, and the market for them is small. It should
not be surprising that they're mostly not all that good.

What's more, the benefit of even a great IDE is, let's face it,
marginal, as long as the basic tools are comprehensive and good. So
yeah, I'd like to have a great IDE. I'm not holding my breath. Maybe
I'll like the Factor IDE. It looks pretty good. But I'm working in
Scheme, right now.

So, yeah, folks who never want to see a line of C code, you're not
going to like the tools I'm using. Ditto for anyone who must have his
projects managed by a digital valet in an impeccably-tailored suit of
color-coordinated windows. There's nothing for you guys to see here;
move along.

For me, though, Gambit has definitely been great. In the development
of Delectus, I've ditched XCode (but not Interface Builder). In Alpha
2, 83% of the application code by linecount is Scheme code. The
remaining 17% is Objective-C. Basically, there's just enough
Objective-C to load the nibfiles that define the app's windows and
menus, and hook their event-handlers to the Scheme code that does all
the real work.

After some experimentation, I partitioned the application into the
following parts:

  1. The back-end: this is pure Scheme code that handles all the data
     structures, as well as sorting, filtering, and updating. It also
     handles serializing and deserializing data as CSV and a native
     binary format, and writing to and reading from disk files.  

  2. The UI: this is Objective-C code that loads the nibfiles, sets up
     calls into the back-end, responds to notifications that originate
     in the back-end, and provides the minor customizations I needed
     to make Apple's UI elements look just the way I wanted. This part
     is less than 700 lines of code, total.  

  3. The bridge. This is a small amount of Scheme code, and an even
     smaller amount of Objective-C code, in a couple of files. It
     provides simple utilities like a means of converting Scheme
     strings to NSString objects when the UI needs to fetch a string
     from the back-end, without being prodigal about allocation of
     strings. It provides a mechanism for notifying the Objective-C
     code from Scheme code, so that the pure-Scheme back-end can post
     error messages, update requests, and other notifications, without
     any code that is actually aware that an Objective-C component
     exists.

The last point is maybe interesting. There is a very small amount of
mixed Scheme/C code that enables the Objective-C UI to interact
productively with the Scheme back-end. I could easily have put that
code into the main body of the back-end proper, and at first, that's
what I did. After all, there's only a little of it; what's the harm?

Before long though, I refactored the code to completely separate all
that code into a bridge layer. The reason, basically, is that I like
to have a fairly comprehensive set of unit tests around my major
application functionality. With such a set of tests in place, it's
very easy to detect when a change in the code makes an application
feature fail, and it's easy to find what broke it. By factoring all of
the mixed code out into its own small layer, I was able to make the
back-end pure Scheme code, with no reference anywhere to any foreign
functions. That means that I can load the entire back-end into an
interactive repl and exercise it. I can run all the unit tests at
once, or run particular ones interactively. If one breaks, I can use
the interactive debugger to crawl around inside the back-end's runtime
and figure out what's wrong.

Furthermore, by factoring things this way, I create a circumscribed
and coherent API for calling into the back-end and getting data from
it. That makes it very easy to hook up the Cocoa UI, simply by having
event-handlers call the appropriate API functions. UI calls into the
back-end are almost always one or two lines of code. That means that,
even when a bug is in the UI rather than the back-end, the way the app
is factored makes that easy to detect. And since there is so little
front-end code, it's easy to find and fix.

Obviously, you could structure any Cocoa app similarly, whether the
back-end is Scheme or C or ML or whatever. What makes Gambit great for
me, a Lisp hacker by preference, is that over eighty percent of the
code I work with is pure Scheme code that I can work with
interactively, in the way I prefer.

You could argue that Delectus is really a C program, on the grounds
that Gambit-C compiles Scheme to C, before subsequently compiling it
to native code. That suits me fine, as long as I can continue to write
my C code in Scheme.

I'll say it again: Gambit-C has been a great platform for developing a
Cocoa app. I'll enthusiastically use it again in the future to do the
same kind of work. I would recommend it to anyone who wants to use
Lisp to write a Cocoa app, provided that they're not allergic to C
code, and provided that they're okay with using compilers and editors
without an IDE to command the servants for them.

Delectus should be out soon. The gating factor right now is the
opinion of the testers. When they're happy with the feature set and
the stability of the product, I'll release it.

After the release of Delectus, I'll begin work on a web application
aimed at MMORPG players. Most likely, the server side of that
application will be written in Clojure.

A little while ago, I promised interested people that they would soon
be able to get their hands on my Categories object system. The object
system had to take a back seat to revenue-generating work, but when
Delectus is out, I'll be working on a release of Categories. I intend
to use it in my next product anyway, and so I may as well also package
it for release to the tiny community of people who are interested in
such things.

The current working prototype of Categories runs on Gambit, and it's
likely that's the version I'll clean up and release first. Then I'll
port it back to Clojure (which is where I developed the first versions
of it), and make that available as well. It may be a little while
before I do either; we must now wait on the verdict of my
testers. When they say Delectus is done, then I can move forward.

I have several other plans in the works, including the resurrection of
some old friends. After six years, I still occasionally get fan mail
about Alpaca, the text editor. Those few fans may be glad to know that
a new version of Alpaca is on the roadmap. Another old project that
I'm still sometimes asked about is Bosco, a simple set of tools for
building a Cocoa application in Clozure Common Lisp. Well, Bosco was
absorbed into Clozure Common Lisp a couple of years ago, but there
stlll seems to be a place under the sun for a small package of files
that shows how to build a minimal Cocoa application with the smallest
possible amount of Lisp code. I have another project called Apis that
can serve that purpose, among others. Apis, and other things derived
from it, are also on the roadmap.

There are other things, but they're even further down the road. For
now, I'm still squarely in the midst of Delectus, and loving
Gambit. Turns out it's still a great platform for developing Cocoa
apps.
