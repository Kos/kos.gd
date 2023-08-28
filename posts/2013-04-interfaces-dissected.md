<!--
.. title: Interfaces dissected
.. slug: interfaces-dissected
.. date: 2013-04-28 19:54:46 UTC
.. tags:
.. category: dev
.. link:
.. description:
.. type: text
-->

Recall methods. A method is some piece of code that can be "called on objects". There are two things to methods: They're meaningful with an object to call on, and their interface looks like "they can be called with some arguments to return a value".

Properties are similar to methods, in that a property is meaningful when paired with an object. They have a different interface than methods, though: you don't "call" them; instead you "get" and "set" their "current value".

We normally consider properties and methods to be some abstract "traits" of objects. But why not treat them as first-class objects too? Imagine properties and methods as **building blocks for object interfaces**. A method would be an object that describes callable parts of other objects, while a property describes readable and assignable parts.

<!--more-->

There's an abstract interface to properties and methods, which could look like:

    interface Method<ArgTypes..., ReturnType> {
    	ReturnType call(Object, ArgTypes...);
    }
    interface Property<ValueType> {
    	ValueType get(Object) [optional];
    	ValueType set(Object, ValueType) [optional];
    }

(Or you could define Method to be a special case of a Property, that is read-only and returns a callable.)

One instance of a Method is a complete definition together with its code; similarily for Properties. Such instances are typically placed together, in classes or prototypes. One instance of a property or a method is relevant for many objects - all objects of the given class. All properties and methods in a particular class build the interface for the object of that class.

## Accessing the interface

The normal syntax for accessing properties is:

    obj = new Class();
    obj.foo;      # read
    obj.foo = 10; # write

One could invent an alternative syntax which highlights that the property is an object too:

    obj = new Class();
    foo_property = Class.foo;
    foo_property.get(obj);     # read
    foo_property.set(obj, 10); # write

*(Fun fact: That's pretty much how properties [work internally][descr] in Python.)*

From this point of view, it's easy to notice that a property can behave like an object too, and its interface could easily contain more facilities than just "get" and "set", even though there might be no equivalent "simple syntax" to use them directly.

[descr]: http://docs.python.org/2/reference/datamodel.html#implementing-descriptors

## Let's make more of them

In order to have objects with richer interfaces is to invent more building blocks for interfaces than just the simple Property and Method.

- You could extend the Property interface with more than just getters and setters. How about Properties that also can tell if they're "enabled" or "disabled" at a given moment? Or inherently support change subscription and notification?

- You could add different meaningful building blocks for interfaces. For example .NET interfaces can also have events, their interface being "add callback" and "remove callback".

While there's much elegance in building complex interfaces using simple building blocks, the approach of redefining the brickis tempting too. Richer interface building blocks should lead to richer interfaces, right?

A notable example of this direction is the set of .NET features that power the WPF, such as Dependency Properties and Routed Events. I've also had some success with this approach during [Glory][glory]'s development, when I was designing what later became [pyvvm][pyvvm], a microframework for MVVM programming in Python.

[glory]: https://kos.gd/2013/03/introducing-glory/
[pyvvm]: https://github.com/Kos/pyvvm
