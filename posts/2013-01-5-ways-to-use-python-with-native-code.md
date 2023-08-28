<!--
.. title: 5 ways to use Python with native code
.. slug: 5-ways-to-use-python-with-native-code
.. date: 2013-01-29 21:07:38 UTC
.. tags: python, polyglot
.. category: dev
.. link:
.. description:
.. type: text
-->

So you want to make a project with the advantages of native code. You like the performance of fine-tuned C, you need some heavy pointer juggling or bit fiddling or SSE, or perhaps you just dream of the vast lands of C libraries available around there. But you're in love with Python and really want to be able to glue your app together with elegant high-level code. Hard decision? Definitely.

But why can't we have both? Using two languages together introduces complexity, but might have good payoffs for your project. Give it a shot.

<!--more-->

# 1. Load a dynamically linked library

There's a standard Python library called `ctypes` that is able to load a library (`dll` or `so` depending on your system), call functions from it and do the necessary conversions between C types and Python types. Easier than it sounds like!

    >>> import ctypes
    >>> n = ctypes.cdll.msvcrt.printf('hello world! %d\n',123)
    hello world! 123
    >>> n
    17

That said, I must mention that the library needs some boilerplate code when dealing with more complex types, like structures. You may have some luck with wrapper generators like [ctypesgen][ctypesgen] that can parse C headers and generate Python code with the needed definitions.

[ctypesgen]: http://code.google.com/p/ctypesgen/

This library doesn't excel in performance and doesn't grok C++ (I don't think that's doable, there are too many differences between compilers here). However it has a great advantage: You don't need the library's source to use it via ctypes (except the signatures of functions you're calling). Just load up and go!

Go ahead and read [**my other article: "Python likes DLLs"**][glfw] to see how far can you get with just a few lines of `ctypes`-driven Python code.

[glfw]: /2012/10/python-likes-dlls/

# 2. Write a Python module in C

The basicmost way to create "real" Python extension modules is to code them against the official Python API directly. The API is rather massive and needs some insight in how Python works internally, but it also gives you full control and allows to squeeze the best performance.

Interested? Consult the official docs:

- [Python 2](http://docs.python.org/2/c-api/)
- [Python 3](http://docs.python.org/3/c-api/)

# 3. Wrap up your code with [Boost::Python][boost]

Boost::Python is a template library that provides a declarative syntax to wrap up your code. The compiler expands the templates and ends up with some code that cleverly exposes your module to Python via the C API.

Exposing a class simply looks like:

    class_<World>("World")
        .def("greet", &World::greet)
        .def("set", &World::set)

The library is clever enough to handle type conversions, exception translation and more. [See the docs][boost] for more examples.

*(If you have LuaBind background, you'll feel at home here.)*

[boost]: http://www.boost.org/doc/libs/release/libs/python/

# 4. Generate a wrapper module with [SWIG][swig]

SWIG, or "Simplified wrapper and interface generator" is a standalone application that generates wrapping code. The typical use is to include it in your build chain, much like flex/bison.

The idea is:

- Write an "interface definition" file that lists the parts of your C or C++ code you want wrapped.
- Show it to SWIG, obtain a C file with wrapper code.
- Compile it and link together with your project and Python libraries. Boom, you have a Python module.

Too much work? Just give SWIG a header and it will read the definitions from it. The authors claim that SWIG understands "almost all of ANSI C++". It's also customisable, of course.

And here's the big news: Besides Python, SWIG is also comfortable with generating extensions for C#, D, Go, Java, Lua, Perl... [You name it.][swig_compare] How awesome is that?!

[swig]: http://www.swig.org/
[swig_compare]: http://www.swig.org/compare.html

# 5. Rewrite your module in [Cython][cython]

So you have already written some Python code and you're desperate to make it faster? Read on. Cython is a language very similar to Python that allows you to add some annotations, in particular static type definitions.

The authors give two cases where Cython works well:

- extending the CPython interpreter with fast binary modules
- interfacing Python code with external C libraries

Cython code gets compiled to equivalent C that can be compiled into Python extension module. Depending on what you're doing, it may result in 2x or 100x speedup.

Here's a snippet:

	def integrate_f(double a, double b, int N):
	    cdef int i
	    cdef double s, dx
	    s = 0
	    dx = (b-a)/N
	    for i in range(N):
	        s += f(a+i*dx)
	    return s * dx

Cython is able to generate Python functions, C functions or C functions with Python wrappers - all with similar syntax.

[cython]: http://docs.cython.org/src/quickstart/overview.html

# Still not satisfied?

Fan of managed code? Check [IronPython][iron] for .NET and [Jython][jython] for the JVM.

[iron]: https://ironpython.net/
[jython]: https://www.jython.org/

Or maybe [embedding the Python interpreter inside a C or C++ application][embedding] is what you are after? This can work in tandem with the solutions mentioned before.

[embedding]: http://docs.python.org/2/extending/embedding.html
