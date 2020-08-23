---
published: true
title: A peek at Categories
layout: post
tags: programming
---

I've begun preparations to package Categories for release. The first
thing I'm doing is going over the surface syntax. In previous
iterations I've built Categories from the bottom up, which confirmed
that the concepts and mechanisms worked, but left a few warts in the
surface syntax. This time, I figured I'd start at the surface syntax
and work the other way, in hopes that the result will be more
congenial.

This rather long post describes the current state of the Categories
surface syntax. Those who are interested in such things are invited to
tell me what's wrong with it.

Categories has three logical parts: datatypes (representation),
functions (behavior), and domains (taxonomy). Following Larry Tesler's
maxim that 'simple things should be simple; complex things should be
possible,' I'll start by describing just the simplest and most common
use-cases, and then make another pass through all three parts of the
system to add in the more complicated options.

## Datatypes

Representation is the province of Datatypes. Types are first-class
objects in Categories. They can be created by evaluating expressions,
passed to and returned from functions, and bound to variables.

Categories defines a set of primitive type objects that represent
built-in types of the underlying platform. For example, the Scheme
version defines primitive types like `<pair>` and `<symbol>`, and it
arranges for its reflective operations to return appropriate
values. For example:

```scheme
? (type 'foo)

<symbol>

? (type? 'foo <symbol>)

#t
```

## Type synonyms

The simplest possible way to define a type is to say that it's the
same as some existing type. Of course, even simpler than that is to
just go ahead and use the existing type. But sometimes you want a
different name for the type, to better communicate its intended
use. Sometimes you want to use an existing type for now, but may want
to represent the data differently in the future. To make these uses
easier, Categories provides type synonyms:

```scheme
(define <name> 
  (type-synonym <string>))
```

By defining `<name>` as a synonym for `<string>` you can use `<string>`
objects to represent names, while calling them `<name>`s in your code to
clearly communicate their purpose. Later, if you find that you need to
represent names in some other way--say as some structured object that
takes into account various different kinds of names that people
use--you can replace the definition of `<name>`. You'll then have to
change the implementations of functions that operate on `<name>`s, of
course, but if you structure your APIs appropriately, that's all
you'll have to change.

### Structures

More commonly, we want to create structured types by grouping several
simpler types into named fields. In Categories, that kind of type is
called a structure:

```scheme
(define <cartesian-point> 
  (structure () x y))
```

Like other types, structures are first class objects. The above code
binds a new structure to the variable `<cartesian-point>`. The new
structure has two keys: x and y. Instances of `<cartesian-point>` will
contain two values, one stored on the key x, and the other stored on
the key y.

You can create an instance of a structure by applying the function
make to it:

```scheme
(make <cartesian-point>)
```

You can get the value associated with a key by applying the function
get-key:

```scheme
? (get-key (make <cartesian-point>) 'x)

Error: Unbound key x 
```

Unfortunately, we didn't specify any value for x when we created the
instance. You can do that by passing in an initializer for the key:

```scheme
? (get-key (make <cartesian-point> x: 101) 'x) 

101 
```

You can also specify a default value for a key in the definition of
the type:

```scheme
(define <cartesian-point> 
  (structure () 
    (x default: 7)
    y))

? (get-key (make <cartesian-point>) 'x) 
  
7 
```

Before I go any farther, I had better explain why there's an empty
list after the symbol 'structure' in:

```scheme
(define <cartesian-point> 
  (structure () 
    x
    y)) 
```

When you create a new structure, you can specify zero or more existing
structures to include in its definition. So if I wanted to define a
new type of 3D point that is just like a cartesian point, but with an
extra coordinate, I could define it like so:

```scheme
(define <3d-point> 
  (structure (<cartesian-point>) 
   z)) 
```

'Okay,' you're thinking, 'that's nothing new. The new type inherits
from the old one.' Not so fast: there's no such thing as inheritance
in types. Including a type just means you don't have to retype all its
field definitions when you want to reuse them. Categories does support
inheritance, but that belongs to domains, not to datatypes. Remember:
datatypes are representation. Inheritance belongs to taxonomy, and
that's the province of domains, not of datatypes.

### Structure amenities

The structure constructor provides a few more conveniences. You can
say that you want Categories to automatically construct getter
functions for the new type's keys:

```scheme
(define <cartesian-point> 
  (structure () 
    (x get: get-x) 
    (y get: get-y))) 
```

Now you can fetch the x value from a cartesian point like this:

```scheme
? (get-x (make <cartesian-point> x: 101))

101 
```

If you leave off the get: argument for a key, then you have to use
get-key and the key's name to fetch the value.

You can also specify a setter:

```scheme
(define <cartesian-point> 
  (structure () 
    (x get: get-x set: set-x!)  
    (y get: get-y set: set-y!)))  
```

Categories then constructs a function for setting the value associated
with the key:

```scheme
? (define p (make <cartesian-point> x: 101))

#<<cartesian-point> x: 101 y: #<unbound>>

? (get-x p) 

101

? (set-x! p 1001) 

? (get-x p) 

1001 
```

If you omit the set: argument then Categories makes the key
read-only. Categories is generally biased in favor of immutable data.

### Functions

Now that we know how to define datatypes, what do we do with them? We
apply **functions** to them.

Functions are objects that can be applied to zero or more parameters
and return zero or more results. Functions as defined in Categories
are polymoprphic. In other words, the same function can have several
different definitions. The one that applies to a particular set of
inputs depends on the inputs.

This practice of making the particular implementation of a function
depend on its inputs is called **polymorphism**, and is one of the
foundations of object-oriented programming. Choosing a specific
implementation to run is called **dispatching**. Categories provides
polymorphic functions with programmable dispatching.

We'll look at how you can create your own dispatching schemes
below. First, though, let's look at what it's like to define and use
functions in the simplest way.

### The Default domain

The simplest way to define and use functions in Categories is with the
**default domain**. Categories assumes you are using the default
domain unless you tell it otherwise. The features of the default
domain generally resemble those of CLOS.

Like types, functions are first-class objects. You can create one and
bind it to a variable, like this:

```scheme
(define serialize 
  (function ((s <store>))
    (vector 'store 
            (version s) 
            (map serialize (columns s)) 
            (column-order s) 
            (sort-column s) 
            (sort-reversed? s) 
            (map serialize (rows s)) 
            (notes s)))) 
```

The function object that is bound to the variable named 'serialize' is
created with a single implementation, corresponding to the type
`<store>`. If we want to add another implementation, we can use
add-method!:

```scheme
(add-method! serialize ((d <document>)) ...)

```

Why do the parameter lists have double parentheses? Because each
parameter may be type-qualified:

```scheme
(add-method! serialize ((d <document>)(outs <stream>)) 
  ...)  
```

You can define a method to execute when no match is found for the
inputs by omitting the type qualifiers:

```scheme
(add-method! serialize (something somewhere) ...)

```

It can be an inconvenience to keep track of where the definition of
the function first appears, and to make sure that all the add-method!
calls appear after it. If you instead use the define-function macro,
that bookkeeping is handled for you automatically:

```scheme
(define-function serialize ((s <store>)) ...)

(define-function serialize ((d <document>)) ...)
    
```

In that case, it doesn't matter which comes first.

### Domains

So how exactly does dispatching work in Categories? Well, that depends
on what domain you use. By default, you use the default domain. The
default domain, which is named -c3-, uses a dispatching algorithm very
similar to that of CLOS. It's not identical to CLOS; it uses a scheme
first described in

 [http://192.220.96.201/dylan/linearization-oopsla96.html](http://192.220.96.201/dylan/linearization-oopsla96.html)

That scheme, which is generally called 'C3,' yields dispatching
behavior that is very similar to that of CLOS, but which in a few
complicated corner cases produces less surprising results. It's the
default in Categories because I like it, and because it has served the
general programming community well. It was designed for the Dylan
programming language, but has been adopted by implementors outside the
Dylan community, including the Python implementors and the designers
of the Class::C3 object system for Perl.

Although C3 is the default dispatching scheme, it's not the only
dispatching scheme supported by Categories.  Two other dispatching
schemes are provided with Categories, and by defining domains, you can
make your own.

Before we get to that, though, let's just look at how to use something
other than the default domain.

When you write:

```scheme
(define-function serialize ((s <store>)) ...)  
```

that expression is really shorthand. It leaves out a reference to the
domain you're using. If we wrote it out in full, it would look like
this:

```scheme
(define-function -c3- serialize ((s <store>)) ...)  
```

You can see that here we've said explicitly that we're defining a
function that operates in the -c3- domain. We could instead use one of
the other domains supplied with Categories, the Flat domain:

```scheme
(define-function -flat- serialize ((s <store>)) ...)  
```

The difference between -c3- and -flat- is that -c3- defines supertype
relations among types, and -flat- doesn't. Suppose you make an
instance of <repository> and call serialize on it.

```scheme
(define $r (make <repository>)) 

(serialize $r) 
```

What happens? That depends on whether serialize is defined on the -c3-
or on the -flat- domain. It's the domain that defines how dispatching
is done. If it's defined on -flat- then the answer is simple:
serialize finds no match for $r. Unless a default method is defined as
in the example above, an error is signaled.

The -c3- domain defines a more complicated--and more
familiar--dispatching scheme. It will try to match $r against
serialize methods that are defined on `<repository>`.  When it fails to
find one, it then tries to find serialize methods defined on
supertypes of `<repository>`.

Didn't I say above that datatypes don't define supertype or
inheritance relations? Yes, because *domains* do. The -c3- domain
defines supertype relations with multiple inheritance. It tries to
find an inherited method that matches, using the C3 algorithm to
determine which matching method is most specific. Once again, if no
such method is found then either the default method is called or, if
there is no default method, an error is signaled.

### More options

That's a first pass across the simplest and most commonly-used
features of Categories. Let's go over the parts of the system one more
time, and look at a few more options that it provides.

### Categories

First of all, I left out an important group of types, because it's
hard to explain what they're for until you've seen how Categories
works with domains. A constructor named 'category' creates a kind of
type that is different from the others. For example:

```scheme
(define <vehicle> (category <car> <truck>)) 
```

The expression above binds a new type to `<vehicle>`. The new type,
which is called a category, represents any of the types passed as
inputs to the 'category' constructor. Once `<vehicle>` is defined, any
method matching `<vehicle>` will also apply to `<car>` or
`<truck>`. You can therefore define functions that accept `<vehicle>`
values, and those functions will work on `<car>` or `<truck>` values,
*even if the functions are defined on the flat domain*. In other
words, even without inheritance, it is possible to define functions
that operate the same way on a variety of different types.

### Supertypes and subtypes

Okay, but suppose you actually want types that have supertypes and
subtypes? Well, then, you can use -c3-,or any domain that supports
supertype relations, and you can add your own types to it. For
example,

```scheme
(add-type! -c3- <truck> 
  supertypes: (list <transportation> <four-wheeled-objects>)) 
```

...adds to -c3- a type named `<truck>`, with supertypes
`<transportation>` and `<four-wheeled-objects>`.

You can, of course, also add types to the -flat- domain:

```scheme
(add-type! -flat- <truck>) 
```

You can't specify any supertypes for it, though, because -flat-
doesn't support supertype relations. If you try to, then add-type!
will signal an error.

### Name collisions

What happens if I define a type that includes two other types that
have different definitions for the same key?

For example, suppose I have these two definitions:

```scheme
(define <monument> 
  (structure () 
    (age type: <historical-epoch>))

(define <property> 
  (structure () 
    (age type: <count-of-years>)) 
```

Now suppose I want to define `<historic-property>` like
this:

```scheme
(define <historic-property> 
  (structure (<monument> <property>) 
    (square-footage asking-price)) 
```

Well, I can't. The definition of age in `<monument>` conflicts with
the definition of age in `<property>`. What I can do, though, is
rename one or both of the included keys:

```scheme
(define <historic-property> 
  (structure ((<monument> (age as: period))
              (<property> (age as: how-old)))
    (square-footage asking-price))) 
```

### The Predicate domain

Besides the default C3 domain and the flat domain, Categories includes
a third domain that works very differently from either. The -pred-
domain matches input function arguments with a user-defined predicate
function. For example:

```scheme
(define-function -pred- ((x y z) all-odd?)  ...)

```

The example above shows a function that will match any three inputs,
as long as calling the function all-odd? on the sequence of three
inputs returns true. You can define functions that match on any
predicate you like. You just have to ensure that when the function
passes the inputs to your predicate, the predicate correctly returns
either true or false.

The -pred- domain also uses the C3 algorithm to support supertype
relations among predicates. Using the -pred- domain you can construct
whatever relations among predicate functions you like, and Categories
will dispatch appropriately. For example, you define all-integers? to
be a supertype of all-odd?, and then anything that matches all-odd?
will also match all-integers?.

I should probably mention here that -pred- may not be all that useful
to anyone. It exists mainly because I wanted to make sure that it was
workable to build a dispatching scheme in Categories that was quite a
bit different from the usual thing.

One mystery remains with the -pred- domain, though: the syntax for
defining a function is different from the syntax used with the -c3-
and -flat- domains. How does that work?

### Defining domains

The unique define-function syntax of the -pred- domain is provided by
the domain itself. It turns out that the operators 'function' and
define-function' don't define the syntax of parameter lists; the
domain does. The -c3- and -flat- domains both define a syntax that is
familiar to users of CLOS and similar object systems. You write
define-function forms like this:

```scheme
(define-function -c3- add ((x <integer>)(y <integer>)) ...)  
```

The forms `(x <integer>)` and `(y <integer>)` mean that the formal
parameters x and y match inputs only if those inputs can be matched
with the `<integer>` type, according to the domain.

A domain like -pred- works quite differently, and requires a different
syntax for parameter lists. Instead of piecewise matching input values
against types, -pred- matches the parameter list as a whole against
predicates applied to it. That makes it possible to dispatch on
conditions that -c3- and -flat- can't check. For example, -pred- can
match a sequence of five integers that are monotonically increasing,
or a sequence of characters that forms a palindrome.

But if some domains require unique parameter-list syntax, that means
Categories must have a way to define that syntax; and it does.

When you define a domain, you are defining three things:

1. a format for function parameter lists  

2. a function that determines whether a particular method matches a
   given sequence of input values  

3. a function that can tell you which of two methods better matches a
   given set of inputs  

Here's what a domain definition looks like:

```scheme
(define -my-domain- 
  (domain catalog: some-catalog-data-structure 
          parser: (lambda (domain formals) ...)
          matcher: (lambda (domain signature values) ...)  
          comparator: (lambda (domain signature1 signature2 values) ...)))  
```

Right away you'll notice that, although I said you have to define
three things, there are four inputs to the domain
constructor. Categories provides each domain with a data structure
called a **catalog**. The catalog can be anything you like. It can be
a false value, or an integer, or a table, or a more complex data
structure. It's up to you. Its purpose is to provide storage for any
auxiliary data that your domain might need to perform its tasks
correctly and efficiently. You can use it to store method caches, or
arrangements of supertypes designed to make lookup fast, or whatever
you wish. You can also ignore it completely, if you wish.

The other inputs to the domain constructor are essential, though. Each
of them is a function that defines one aspect of how the domain
works. Because your domain may have crucial data stored in its
catalog, all three of the functions accept the domain itself as the
first argument. Its up to you what you do with that argument; it's
available in case you need it.

Besides the domain parameter, the functions that define a domain deal
with three other items: formal parameters, method signatures, and
input values.

A formal parameter is an object that represents the formal arguments
in a function definition. Categories doesn't define the format of
argument lists; that's up to the domain. When it processes a function
definition, it extracts the formal parameter from the function or
define-function expression, and passes it to the domain's parser. The
parser must accept the formal parameter and return a method
signature. It's the parser that determines what the argument list of a
function definition looks like.

The parser doesn't have to be complicated. Consider a -c3- function
definition again:

```scheme
(define-function -c3- add ((x <integer>)(y <integer>)) ...)  
```

All the parser needs to do is collect the type qualifiers from the
formal parameter list, which is `((x <integer>)(y <integer>))`, and
make a method signature that matches two inputs that are both of type
`<integer>`.

That brings us to method signatures, and to the matcher. Once again,
the method signature can be any data structure you like, as long as it
stores the information that the matcher needs. Categories passes to
the matcher a method signature and a collection of input values. The
matcher must return true if the signature matches the inputs, and
false otherwise. For -c3-, a method signature can simply be a sequence
of types. For example, the method signature for the add function
defined above is just `(<integer> <integer>)`. The -c3- domain will
say that any sequence of two values matches that signature, as long as
both are of type `<integer>`, or of some type that, according to -c3-,
is a subtype of `<integer>`.

What if more than one signature matches a set of inputs? How does the
domain know what to do?

That's what the comparator is for. After a function determines all the
signatures that match a set of inputs, it sorts them. The comparator
is the function it uses to determine their order. The comparator must
accept two signatures and a sequence of input values. If the first
signature is a better match than the second for the inputs, then the
comparator returns -1. If the second signature is a better match, it
returns 1. If neither is better, it returns 0. It's best to try to
arrange for the comparator not to return zero, if that makes sense for
the domain you're designing, because returning zero means you can't
tell which method to apply. Categories has to signal a dispatch error
when that happens.

### Customizing the default domain

It may happen that you want to make your own domain, but you don't
really want to totally rewrite how everything works. Maybe, for
example, you want to use something other than C3 to order methods, but
everything else about how Categories works suits you fine. In order to
make it easier to customize just the part you want to customize,
Categories provides functions that implement the behavior of the
default domain. You can make your own domains using them, and you can
replace just one of them, or two of them, or you can use all of the
standard functions, but change their inputs or outputs a little using
your own code.

As an example, you can make a domain that duplicates the behavior of
the default domain like this:

```scheme
(define -my-c3-domain- 
  (domain catalog: (make-c3-catalog)
          parser: (c3-signature-parser) 
          matcher: (c3-signature-matcher)
          comparator: (c3-signature-comparator))) 
```

### What next?

All of the features described in this overview have been implemented,
but the working code doesn't provide this surface syntax. The syntax
of the working versions is not too different in most places, but it's
a little more complicated and not quite so tidy. I'll be fixing that
as part of getting the code ready for release.

The prototype I've most recently been working on is implemented on top
of Gambit Scheme 4.5.2. I've tried to avoid making it particularly
Gambit-specific, but the surface syntax I chose does use DSSSL
keywords, which not all Schemes support. In addition, some of the
syntax needs to be defined as macros. For reasons of convenience,
those will be defined using Gambit's define-macro form, rather than
something more portable. It may turn out that someone wants Categories
on some Scheme that lacks DSSSL keywords, or where a different means
of defining macros is needed. I'll cross that bridge when I come to
it.

I plan to port it back to Clojure, as well. The first few versions of
Categories were written in Clojure, but I wanted to move it elsewhere,
mainly to gain the perspective of looking at it as something
independent of a particular language, rather than as an extension
specifically to Clojure. I'm planning a sizeable project in Clojure
shortly, though, and I know for sure that I'll want what Categories
offers me for it.

It may be that the design of Categories has serious flaws. If so, feel
free to enlighten me about them. If they can be repaired, I'll do my
best to fix them. If not, I'll have to look for alternatives. I could
always port [Sheeple](http://github.com/adlai/sheeple) to Clojure, or
maybe implement [PLOT's](http://users.rcn.com/david-moon/PLOT/) object
system.

