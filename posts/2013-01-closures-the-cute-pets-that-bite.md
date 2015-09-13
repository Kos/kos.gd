<!--
.. title: Closures: The cute pets that bite
.. slug: closures-the-cute-pets-that-bite
.. date: 2013-01-18 20:08:21 UTC
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

A closure is basically a function that was defined in some local scope and has access to the local variables from that scope. Additionally when a closure (the function object) outlives the scope where the closed variables were defined, their lifetime gets extended (or not, depending on the language).

I find closures wildly useful, especially in prototyping. They also are welcome in production code, if you don't get too carried away. However they are not exactly as intuitive as one might expect and if you decide to use them there's a handful of details you must remember not to introduce bugs.

This article has two parts:

- Differences between closures in some languages
- Pitfalls related to closures that you should remember and avoid.

Also for a quick recap on closures see [this gist](https://gist.github.com/4568081).

<!--more-->

# Differences in languages

The most common case is above, however some languages give you some additional features... Or take away some.

## By value, by reference

While closures typically capture variables, some languages also allow you to capture the variable's value (at the moment when the function object gets defined). Most notable examples of languages that allow this are C++ and PHP.

C++ example:

	#include <functional>
	#include <iostream>

	std::function<int(int)> times(int n) {
		auto value = [n](int m) { // Captures by value
			return n*m;
		};
		n = 999; // This won't change a thing,
				 // the previous value was copied into the closure
		return value;             
	}

	int main() {
		auto twice = times(2);
		std::cout << twice(15);
	}

PHP, same example:

	function times($n) {
		$closure = function($m) use ($n) { // Captures by value
			return $n*$m;
		};
		$n = 999; // This won't change a thing either
		return $closure;             
	}

	$twice = times(2);
	echo $twice(15);

I'll refer to capturing variables as "by-reference closures", and to capturing only current values as "by-value closures".

Capturing by-value has the obvious consequence of preventing further modifications of the variable from propagating into the closure objects. In C++, it also allows the closure object to be used when the captured variables go out-of-scope (which cause trouble when capturing by reference - we'll get back to that).

There's a small difference in how C++ and PHP deal with modifications:

- C++ doesn't allow the closure to modify the variable captured by value. An attempt yields an compilation error.
- PHP does allow that, but the modification isn't "saved" inside the closure. It will have the original value if you call the function again. (Think of modifying a regular function parameter inside function body.) I recommend not using this feature - instead use an additional local variable. Assignments to the closure variable might confuse someone into thinking that the modification will propagate outside.

In both C++ and PHP, you can consider variables captured by value as the closure's "constants", defined per function object.

## Writable or not

### Python 2: Writable... one way

Python 2 is a rare example of a language that has by-reference closures, but doesn't allow to modify the variable from within the closure. This is a consequence of the simple rule "if variable X is assigned to inside a function, then it's local to that function unless declared otherwise (like with `global` statement)". The difference from by-value capture is that changes may still propagate from the variable's original scope into the closure, which is easily confirmed:

	def foo():
		a = 'not modified'
		def closure():
			return a
		a = 'modified'
		return closure()

	assert foo() == 'modified'

Python 3 luckily mitigates this restriction by introducing the `nonlocal` statement while keeping the remaining rules unchanged. This allows for:

	def bar():
		a = 'not modified'
		def closure():
			nonlocal a
			a = 'modified'
		closure()
		return a

	assert bar() == 'modified'

The relatively common hack to emulate "full" closures in Python 2 is to use an immediate mutable object, like a list:

	def bar():
		a = ['not modified']
		def closure():
			a[0] = 'modified'
		closure()
		return a

	assert bar() == 'modified'

Here the variable `a` is never assigned to inside the closure, so it's not considered a local variable. It's found in an enclosing scope, however, so it's included into the closure.

### Java 7: The paranoid

*Wait, Java has closures? Java 7 doesn't have function objects, not to mention nested functions!* - Well... kinda. It has anonymous classes that have methods, and they are able to form a limited closure.

http://ideone.com/xcMeg6
	class Main {
		
		public static Runnable makeClosure() {
			final int a = 10;
			return new Runnable() {
				@Override
				public void run() {
					System.out.println(a);
				}
			};
		}
		
		public static void main(String[] args) {
			Runnable obj = makeClosure();
			obj.run(); // prints 10
		}
	}

There's the restriction however: You can only close over local variables declarered as `final`. **This resolves the problem of change propagation into and out of the closure by preventing changes altogether.** This fact also cuts the argument whether the capture is by-value or by-reference: implementation-wise it could be both and you wouldn't notice. Also note that the values get captured into the anonymous object, not exactly into functions.

Anonymous class objects also implicitly capture the parent instance when applicable, not unlike nested class objects. It is used to access the parent instance's members.

This concludes the description of how closures look like in languages. The fun part begins: Traps!

# Trap 1: Variable scoping

Closures usually capture variables by reference. This is sometimes forgotten, which leads to unexpected results. Consider this innocent C# code:

	using System;
	using System.Linq;
	 
	class Example1 {
		
		static String[] ingredients = new[] {"Spam", "eggs", "magic"};
		static String[] fridge = new[] {"Five cans of Spam", "a crate of eggs", "milk juice"};
		
		static int Main() {
			
			foreach (var ingredient in ingredients) {
				if (fridge.Any(x => x.IndexOf(ingredient) > -1)) {
					System.Console.WriteLine("{0} - check", ingredient);
				} else {
					System.Console.WriteLine("Go buy {0}!", ingredient);
				}
			}
			return 0;
		}
	}

Looks nice. See this closure in delegate passed to `Any`? How about this example. Another simple, read-only closure:

	using System;
	 
	class Example2 {
		
		static event Action Event;
		
		static int Main() {
			
			for (int i=0; i<3; ++i) {
				Event += () => System.Console.WriteLine("Event {0}", i);
			}
			Event();
			return 0;
		}
	}

The result might be surprising:

> Event 3
> Event 3
> Event 3

The explanation is trivial, however, if you remember that closures capture by reference. There's only one variable `i` declared and the same variable gets into all 3 closures. The workaround is:

		for (int i=0; i<3; ++i) {
				var j = i;
				Event += () => System.Console.WriteLine("Event {0}", j);
			}

This fixes the problem:

> Event 0
> Event 1
> Event 2

Note that what we're *actually* trying to do here is capture `i` by value instead of by reference. This temporary-variable solution has an equivalent effect.

Foreach loops suffer from the same issue in C# 4.0, but allegedly the loop semantics have been altered in C# 5.0 to mitigate that, see [Marc Gravell's excellent post][so-cs] on StackOverflow. This is a bold change; let's hope it doesn't cause extra confusion by making `foreach` scope the loop variable differently from `for`.

If you're using [Resharper](http://www.jetbrains.com/resharper/) (which you totally should if you're a C# programmer), you'll encounter the "Access to modified closure" warning when treading on these grounds.

Bonus question: Why didn't the first example suffer from problems of this kind, even assuming C# <= 4.0? Answer: While all delegates closed over the exact same variable, they were called immediately after declaration and weren't stored afterwards. Unlike event handlers from Example 2, they didn't live long enough to see their captured variable change.

[so-cs]: http://stackoverflow.com/a/304387/399317

# Trap 1a: Variable scoping continued

(This one is my favourite.)

Let's rewrite that in Javascript. Same problem:

	var handlers = [];
	function setup() {
		for (var i=0; i<3; ++i) {
			handlers.push(function() {
				print(i);
			});
		}
	}
	setup();
	for (var x=0; x<3; ++x) handlers[x]();

Same result:

> 3 
> 3
> 3

Same fix:

	var handlers = [];
	function setup() {
		for (var i=0; i<3; ++i) {
			var j = i;
			handlers.push(function() {
				print(j);
			});
		}
	}
	setup();
	for (var x=0; x<3; ++x) handlers[x]();

Result:

> 2
> 2 
> 2

...Wait, what?! Why not 0, 1, 2? Each closure gets its own variable, right?

Well, no. Not at all.

The culprit is, again, variable scoping rules. In C# local variables have **block scope** (like Java, C and C++), but **block scope doesn't exist in Javascript!** Local variables have **function scope**, so that fix didn't really fix much - all closures still capture the same variable (only that this time it's `j`, not `i`, so we end up with three `2`s).

To work this problem around, you can rewrite the code like this:
 
	function make_printer(n) {
		return function() {print(n);};
	}

	var handlers = []; 
	function setup() {
		for(var i=0; i<3; ++i) {
			handlers.push(make_printer(i));
		}
	}
	setup();
	for (var x=0; x<3; ++x) handlers[x]();

Result:

> 0
> 1
> 2

The function `makeEventHandler` can be a global function or a nested function - the point is, it introduces another function scope into which the variable `i` is copied during the function call. Additionally, this version improves readability by making the whole closure thing very explicit.

*I believe this is also what [JSLint](http://www.jslint.com/) gently pushes you to do by issuing the "Don't make functions within a loop" warning.*

The same situation exists in Python** and I recommend exactly the same solution. There's one more workaround that you might stumble upon, though:

	handlers = []
	def setup():
		for i in range(3):
			def handler(i=i):        # See?
				print i
			handlers.append(handler)

	setup()
	for x in handlers:
		x()

The result is correct: `0 1 2`. Without that `i=i`, though, you'd end up with result `2 2 2` because the same variable `i` would get into all 3 closures.

Fun fact: There's no closure here! Instead, each loop iteration the current value of the variable is saved inside the new function object as a default parameter value.

This is an interesting hack but I feel it's less readable than the former solution.

# Trap 2: Variable lifetime

C++, as a non-garbage-collected language, has a specific issue that might come unexpected to a programmer used to closures from other languages.
Consider this code:

	// WRONG
	std::function<int()> makeClosure() {
		int counter=0;
		return [&counter]() { 
			return counter++;
		}; 
	}
 
This won't work here like it would in Javascript or C#. The lambda function stores a reference to the local variable `counter` which goes out of scope. If you'd ever call the result, you'd end up with undefined results.

C++ is very strict about variable scope and lifetime. There's no GC, there's the strict notion of no ownership and no magic transfer of such ownership can happen. Consider the more general case: What if `makeClosure` would spawn several closures, each of them referring to `counter`? The compiler would need a non-trivial scheme to make these things work since there's no garbage collector to clean `counter` up after all closures are destroyed.

Lambdas in C++ are safe when either...

- The function object doesn't outlive any objects it captures by reference.
- The lambda function doesn't need its own mutable state (like sharing its closure with another lambdas or modifying variables in its parent scope) - you can capture by value in this case.

Otherwise, don't go with closures and make a class instead.

