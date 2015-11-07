<!--
.. title: Slicing traps in Python 2
.. slug: slicing-traps-in-python-2
.. date: 2013-01-06 13:40:15 UTC
.. tags: python
.. category: dev
.. link:
.. description:
.. type: text
-->

Python 2 has some interesting pitfalls related to slicing. They have roots in the old Python versions before slice objects were introduced. The syntax has been improved since then, but some little gotchas are there and ready to bite you in your knee at the most unexpected moment. [Tomer Filiba describes one such scenario.][tfiliba]

See also [this gist][gist] for some illustation what's happening.

<!--more-->

[gist]: https://gist.github.com/4467206
[tfiliba]: http://tomerfiliba.com/blog/Getslice/

Before explaining, let me first introduce a distinction:

- simple slicing has the form of `obj[x:y]`; x and y are integers or are omitted
- extended slicing is anything else, like:
	- `obj[x:y:z]` or even `obj[x:y:]` (slice has a step)
	- `obj[x:y, 1, 2]` or `obj[a:b, c:d]` (param is a tuple with slices, not a slice)
	- `obj[x:None]` (slice has a non-integer index)

The traps are as follows:

1.
	Simple slicings are interpreted differently with new-style and old-style classes. Differences appear with omitted indices and negative indices. Here are examples:

	- `a[:]`   new-style is `Slice(None, None, None)`, old-style is `Slice(0, sys.maxint, None)`,
	- `a[:-1]` new-style is `Slice(0, -1, None)`, old-style is `Slice(0, len(a)-1, None)`
	   and can throw if `a` doesn't support `len()`.

	New-style slice interpretation always replaces missing args with None, while old-style fills them with 0 or sys.maxint.
	New-style keeps negatives as-is, old-style asks for len() and performs substraction. (You can still end up with a negative.)

	**Luckily, there seems to be no difference with how the `indices()` functions works on these slices**, so as long as you use it (which you probably should be doing anyway), you're green.

2.
	There's an obsolete method `__getslice__` (together with `__setslice__` and `__delslice__`) that was introduced before slice objects. If defined, it will be preferred over `__getitem__` for simple slicing (with both new-style and old-style classes!). It always takes 2 arguments: start index and end index. It interprets omitted and negated indices using old-style interpretation.

	Examples (assuming `type(a).__getslice__` exists):
	- `a[2:3]` calls `a.__getslice__(2,3)`
	- `a[2:]` calls `a.__getslice__(2, sys.maxint)`
	- `a[:-1]` calls `a.__getslice__(0, len(a)-1)`

	Some built-in types have __getslice__ defined too. This means if you ever inherit `list` and override `__getitem__`, you should override `__getslice__` as well for consistency.


