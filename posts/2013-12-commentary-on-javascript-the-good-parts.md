<!--
.. title: Commentary on Javascript: The Good Parts
.. slug: commentary-on-javascript-the-good-parts
.. date: 2013-12-22 18:18:12 UTC
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

"The Good Parts" has been a good read. The language has been described briefly and consistently from ground up, including a thorough explanation of objects, functions, closures and arrays. A great addition to the mix is a description of the numerous pitfalls and design problems. It seems opinionated at some points (more on this later), but very valuable as a whole. (That's correct - "The Good Parts" actually has two appendices dedicated to the not-really-good ones.)

My commentary for selected parts of the book follows.

<!--more-->

## OOP

The book critiques and recommends against the pseudo-classical style (constructor invocation pattern `new Foo()` and extending `Foo.prototype`), but isn't very convincing on why it's bad ("looks alien", no encapsulation, things silently go boom if you forget "new"). I feel there's more to say here. One thing is that this approach becomes unwieldy when you try inheritance - the common hack is to create a dummy `Base` *instance* and use it as a prototype for Derived instances, which kind of works sometimes but is conceptually flawed (the base constructor is invoked only once, while you'd normally want it invoked for each new Derived instance, perhaps with different parameters).

Prototypal inheritance (the "pure" flavour with `Object.create`) is described technically, but description of practical usage seems vague.

The most interesting part describes a functional approach: define a function that creates an object (or takes an existing one) and augments it with some code. The examples rely exclusively on closures, so there's no `this` and no building prototype chains. Normally you'd have methods re-used among many objects (via the prototype chain) and rely on dynamic `this` binding for the method to work on the proper object; the described pattern creates the methods from scratch whenever you instantiate each new object, with the equivalent of `this` hard-wired through a closure. This seems wasteful at first glance.

However, the concept of objects being completely standalone (no classes, no prototypes, no "kinds" - just a builder function) is a very interesting one, native to the dynamic nature of Javascript. One of the examples suggested that adding behaviour to an object can be deferred from its creation, so that you could create an object first and then augment it with some more pieces of code - even depending on runtime conditions.

The chapter left me confused - should I now consider sharing methods via prototypes to be a form of micro-optimisation?

## Encapsulation

The aforementioned functional pattern includes an elegant solution for keeping an object's state private. It relies on the builder function having a local variable with another object, the "private storage". This object would then be closured by every method that needs to access it. This is only possible because the methods are created separately for every instance. If you wanted to pull a method definition upwards to a shared prototype, it would no longer have a way of accessing this storage.

I also miss an explanation why it's useful to go such a long way in hiding your privates. As a dedicated Pythonista, I've long considered "private" to be a form of documentation, rather than access control. It's useful to define a strict public interface for your objects, but why should your logger be prevented from accessing their inner state if you need some diagnostics? (The only reasonable use case I know for strictly enforced access control is a safe-execution sandbox for untrusted code.)

## Builtins

The examples extend builtin types all over the place with every kind of helper functions. There's been a lot of controversy about that practice; some consider it as bad as global variables (for analogous reasons: polluting thy neighbour's namespace and risking name clashes). At the very least, I consider it an omission to suggest that practice without a word of warning.

## Unicode

The book briefly raises a claim that there's a problem with Javascript characters (indexable String elements) being 16 bit and representing code units of UTF-16, so that you sometimes need two Javascript characters to represent a whole Unicode code point. There's no explanation what's wrong with that. [I've mentioned before][hello] that code units aren't generally more or less meaningful than code points in string processing, and an user-perceived character may need several code points to describe. (If still in doubt, refer to [Myths @ utf8everywhere][myths] - most points from there also hold for UTF-16.)

Javascript stays consistent with languages like Java, C# and the part of C world that relies on 16-bit wide chars. I'm glad it didn't bring any nasty surprises here ([I'm looking at you, pre 3.3 Python][py2]).

[hello]: /2013/02/say-hello-to-unicode
[myths]: http://www.utf8everywhere.org/#myths
[py2]: /2013/02/say-hello-to-unicode#Python_2

## Floating point numbers

Another, even bolder claim from the "awful parts" section is the usage of IEEE 754 floating point numbers. Knowing that binary floating point arithmetic is one of the basicmost pieces of knowledge for a programmer, shall we really look for another default optimised towards the human (decimal), rather than the machine (binary)? This is a valid claim, but I don't consider it Javascript-specific; I feel it belongs in a blog post rather than a book of this kind. That said, I do agree that decimal fraction support should be available out-of-the-box in a mature language. Same for arbitrary precision integers.

## Summary

Douglas Crockford's articles have helped me a lot back when I was doing my first steps in serious Javascript. It's been very valuable to solidify my JS-fu with "The Good Parts", even though I had a "but..." to add here and there. The book is from 2008, which is before ``use strict'` and other valuable ES5 additions, so there are still some more good parts to describe, but still it's definitely worth the read.
