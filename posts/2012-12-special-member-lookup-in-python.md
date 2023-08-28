<!--
.. title: Special member lookup in Python
.. slug: special-member-lookup-in-python
.. date: 2012-12-22 22:07:23 UTC
.. tags: python
.. category: dev
.. link:
.. description:
.. type: text
-->

This article describes special method lookup and shows how it differs between old- and new-style classes in Python 2.

The [docs](http://docs.python.org/2/reference/datamodel.html#special-method-lookup-for-old-style-classes) say that...

> For old-style classes, special methods are always looked up in exactly the same way as any other method or attribute.
> (...)
> For new-style classes, implicit invocations of special methods are only guaranteed to work correctly if defined on an object’s type

I'm going to elaborate on that a bit.

<!--more-->

# Intro #

Let's sketch some classes to start with:

```
class OldStyle:
    def __getattr__(self, name):
        print 'old getattr', name
    foo = 10
```

```
class NewStyle(object):
    def __getattr__(self, name):
        print 'new getattr', name
    foo = 10
```

```
a = OldStyle()
b = NewStyle()
```

Obviously we have:

```
>>> type(a)
<type 'instance'>
>>> type(b)
<class '__main__.NewStyle'>
```

The basic lookup works identically: first the instance's own dict is searched for the attribute, then its class and its bases, and finally `__getattr__` gets called.

```
>>> a.foo
10
>>> b.foo
10
>>> a.bar
old getattr bar
>>> b.bar
new getattr bar
```

# Special methods: old-style #

Things get a more interesting when we see how these objects work together with built-in functions and operators. Let's try the old-style instance first:

```
>>> a[5]
old getattr __getitem__
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'NoneType' object is not callable
>>> len(a)
old getattr __len__
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'NoneType' object is not callable
>>> a+a
old getattr __coerce__
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'NoneType' object is not callable
```

All these operations tried to look up their corresponding special member in the object, and since our `__getattr__` doesn't implicitly returns `None` when a function is expected, we end up with the "NoneType object is not callable" exceptions.

However we have observed that `__getattr__` gets called in these situations. It's more than that, actually: The lookup goes the usual way, just like with `a.attribute`: the object's dict is checked first, then it's class and bases, then `__getattr__` if present. We can leverage that and do some "monkey-patching":

```
>>> def printAndReturn(label, value=None):
...     print label
...     return value
...
>>> a.__getitem__ = lambda *args: printAndReturn('getitem called')
>>> a.__len__ = lambda *args: printAndReturn('len called', 123)
>>> a.__coerce__ = lambda *args: printAndReturn('coerce called')
>>> a.__add__ = lambda *args: printAndReturn('add called')
>>> a[5]
getitem called
>>> len(a)
len called
123
>>> a+a
coerce called
add called
```

Now `__getattr__` doesn't get to be called because the functions are quickly found in the instance's own dict.

# Special methods: new-style #

Let's look at the new-style class' object now:

```
>>> b[0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'NewStyle' object does not support indexing
>>> len(b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'NewStyle' has no len()
>>> b+b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'NewStyle' and 'NewStyle'
```

The errors are different this time. Most importantly, **`__getattr__` was never called.** The idea is that lookup for special methods doesn't follow the typical route mentioned before. Instead, only the object's type (along with its bases) is searched for these functions. If not found, there's no fallback to `__getattr__` either.

This also means that monkey-patching the object no longer makes sense, because the instance's own dict isn't searched.

```
>>> b.__len__ = lambda *args: printAndReturn('len called', 5432)
>>> len(b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'NewStyle' has no len()
```

Manual lookup works as usual, of course (but isn't useful since you can't expect anyone to call your special functions like that).

```
>>> b.__len__    # Monkey patched, found in instance dict
<function <lambda> at 0x02962B30>
>>> b.__add__    # Not monkey patched, falls back to getattr
new getattr __add__
>>> b.__len__()
len called
5432
```

For the curious: Does the presence of __getattribute__ change anything in how special member functions are looked up in new-style classes? Answer: Nope, it doesn't.

```
>>> class NewWithGetattribute(object):
...   def __getattribute__(self, name):
...     print 'getattribute called'
...
>>> c = NewWithGetattribute()
>>> c.__len__
getattribute called
>>> len(c)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'NewWithGetattribute' has no len()
```

# Reference #

For more detailed information, as well as some rationale behind this change, I recommend the official docs.

- [Special method lookup for old-style classes)(http://docs.python.org/2/reference/datamodel.html#special-method-lookup-for-old-style-classes)
- [Special method lookup for new-style classes](http://docs.python.org/2/reference/datamodel.html#special-method-lookup-for-new-style-classes)
