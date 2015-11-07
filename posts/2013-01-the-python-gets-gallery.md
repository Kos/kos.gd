<!--
.. title: The Python Gets Gallery
.. slug: the-python-gets-gallery
.. date: 2013-01-08 20:34:01 UTC
.. tags: python
.. category: dev
.. link:
.. description:
.. type: text
-->

Behaviour of Python objects can be customised in tons of ways using special functions. This article describes some of them, all confusingly starting with `__get` and all having drastically different purposes.

<!--more-->

# `__getitem__`: indexing and slicing

This function is associated with the indexing operator: foo[123].

Counterparts: `__setitem__` and `__delitem__`, respectively for assigning to and `del`eting indices or slices.

Note that (unlike function calls, like C++ but unlike C#) indexing is always done with one argument. If the expression inside `[]` contains commas, they denote a tuple, not separate formal arguments.

In indexing, you can use some special syntax:

- `x:y:z` is expanded into `slice` objects,
- `...` is expanded into the `Ellipsis` object.

`slice` objects have 3 properties (`start`, `stop` and `step`). Handling them consistently with Python builtins is at least non-trivial, so they also [provide a method `indices`][indices] which you're encouraged to use. It takes one parameter (the sequence length) and returns a 3-tuple which you can directly pass to `range` or `xrange` to obtain the actual indices (in order) that this slice should refer to.

`Ellipsis` is only there to support the `...` syntax. The only thing you are to do with it is check for its presence.

You can leverage these to introduce complex indexing schemes for your objects. [NumPy's multidimensional array][npindexing] is a notable example for that.

[npindexing]: http://docs.scipy.org/doc/numpy/reference/arrays.indexing.html
[indices]: http://docs.python.org/release/2.3.5/whatsnew/section-slices.html

Some examples:

	>>> class A(object):
	...   def __getitem__(self, x):
	...     print x, type(x)
	...
	>>> a = A()
	>>> a[1]
	1 <type 'int'>
	>>> a[1,]
	(1,) <type 'tuple'>
	>>> a[2,3]
	(2, 3) <type 'tuple'>
	>>> a[2:3]
	slice(2, 3, None) <type 'slice'>
	>>> a[:]
	slice(None, None, None) <type 'slice'>
	>>> a[...]
	Ellipsis <type 'ellipsis'>
	>>> a[1:2:3, :5, 'anything', ...]
	(slice(1, 2, 3), slice(None, 5, None), 'anything', Ellipsis) <type 'tuple'>

Slicing in Python 2 has some gotchas. I've dedicated a small article to them: [Slicing traps in Python 2][traps]

[traps]: /2013/01/slicing-traps-in-python-2

# `__getattr__`: missing attributes

This function takes part in the attribute lookup, like `obj.something`.

Here's how basic member lookup works:

- First, the object's dict is searched for a member of that name.
- If no success, its type is searched.
- Base classes are searched. (Note: if multiple inheritance is involved, the order of this search varies between old-style and new-style classes, [here's a good doc on that][mro].)
- If still no luck, `__getattr__` is called.

`__getattr__` is last here, which means it will only be called if the member isn't found via normal means.

[mro]: http://www.python.org/download/releases/2.3/mro/

The same lookup chain is performed by the `getattr` built-in function. If you want to call `__getattr__` directly, you can simply do `obj.__getattr__(name)`.

	>>> class A:
	...   def __getattr__(self, name):
	...     print 'getattr called'
	...
	>>> a = A()
	>>> a.foo
	getattr called
	>>> a.foo = 123
	>>> a.foo
	123
	>>> getattr(a,'foo')
	123
	>>> a.__getattr__('foo')
	getattr called

There's an important difference how this works for old-style and new-style classes, though:

- for old-style classes, built-in operators and methods use `__getattr__` to look up special functions
- for new-style classes, the object's type (together with its bases) is directly checked for this function. This means that special member functions should only be defined in the class, not in the object itself.

For details, the [the docs][nssl] or my other post: [Special member lookup in Python][smlip].

[nssl]: http://docs.python.org/2/reference/datamodel.html#new-style-special-lookup)
[smlip]: /2012/12/special-member-lookup-in-python

## Notes on counterparts

The counterparts to `__getattr__` are `__setattr__` and `__delattr__`.

There's an assymetry here: the counterparts aren't a part of any lookup chain. Assignments and deletions always modify the object's own properties and don't look into base classes.

Note that there's no `__hasattr__`; there's just the built-in function [`hasattr`](http://docs.python.org/2/library/functions.html#hasattr) that simply...

> is implemented by calling getattr(object, name) and seeing whether it raises an exception or not.

# `__getattribute__`: total attribute lookup

*new-style classes only*

This method is stronger than `__getattr__`: it allows to completely re-define how object attributes are retrieved. This method is implemented in `object` and by default performs the look-up chain (mentioned earlier), and additionally handles descriptors (described later).

Overriding it allows to make total control over how attributes are looked up (except special member functions). Hence, it's only useful for metaprogramming.

It's easy to make the mistake of infinite recursion when implementing `__getattribute__`. In order to actually read any field directly, you can use `super(ClassName, self).__getattribute__(name)`, or perhaps `object.__getattribute__(obj, name)`. Accessing `obj.__dict__` won't work since it will also go through `__getattribute__`.

Counterparts: None, because none are necessary. There's no "default lookup chain" to work around, like it was the case with `__getattr__`. You should be happy with `__setattr__` and `__delattr__` described above.

Trivia: While this method only applies to new-style classes, the docs mention [a way around that](http://docs.python.org/2/reference/datamodel.html#object.__getattr__) for old-style classes too using `__getattr__`:

> Note that at least for instance variables, you can fake total control by not inserting any values in the instance attribute dictionary (but instead inserting them in another object).

# `__get__`: descriptors

*(new-style classes only)*

Descriptors are a mechanism that allows to customise lookup of *single properties* of classes and instances. If a descriptor is found in an object's class (or its bases) via member lookup, then instead of being returned directly, its `__get__` method is called.

A function is the most obvious example of a descriptor. When the lookup `object.func` finds a function in the object's class, then the function's `__get__` is called and the result (a function-like "bound method" object) is returned.

	>>> def func(*args):
	...     print 'called with', args
	...
	>>> class A(object):
	...     func = func
	...
	>>> a = A()
	>>> func
	<function func at 0x0236C770>
	>>> a = A()
	>>> a.func
	<bound method A.func of <__main__.A object at 0x028C4050>>
	>>> # which is the same as...
	>>> func.__get__(a, A)
	<bound method A.func of <__main__.A object at 0x028C4050>>

Descriptors are also triggered on class lookup:

	>>> A.func
	<unbound method A.func>
	>>> # which is same as...
	>>> func.__get__(None, A)
	<unbound method A.func>

The distinction we're observing here is due to different implementations of `object.__getattribute__` and `type.__getattribute__`.

*Note: Python 3 has a slight difference here: instead of the cryptic "unbound methods" a plain function object is returned. The difference is in `function.__get__`, not the descriptor mechanism itself.*

Also note that `__get__` isn't called if the member is found in the object's own `__dict__`:

	>>> a.prop = func
	>>> a.prop
	<function func at 0x0236C770>

Descriptors allow to attach your own semantics to property lookup, on a per-property basis. Common use cases available in Python library are `property`, `staticmethod` and `classmethod`.

See [this excellent guide][rh] to read up on details and see how these standard descriptors can be re-implemented.

- [How-To Guide for Descriptors by Raymond Hettinger][rh]

[rh]: http://users.rcn.com/python/download/Descriptor.htm

Fun fact: The machinery behind descriptors is in the implementation of `__getattribute__` - if you override this method in your class, you're also able to tinker with how they work. (Let me know if you went creative with this!)

