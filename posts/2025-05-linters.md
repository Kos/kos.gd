<!--
.. title: The most common Python trap & how to find it
.. slug: most-common-python-trap
.. date: 2025-05-18
.. tags: python,ruff
.. category: dev
.. link:
.. description:
.. type: text
-->

We had a near miss with a mutable default argument in one of my Python projects.

Mutable defaults are [an old and well-documented problem in Python](https://docs.python-guide.org/writing/gotchas/), arguably a language design mistake. Given its notoriety, I was running under the assumption that it's the kind of obvious Python bug that any linter would immediately pick up and flag. I were to explain to a 6-year-old what a "python linter" is, I'd probably say someting like "a tool that finds typical bugs, like mutable default function arguments".

Turns out I was being optimistic!

We caught the bug before it went live, but I resolved to find which static analysis tools actually check for this problem.

<!--more-->

## Test code

Here's the code I used for my tests:

```python
def buggy_fn(arg: list[int] = []) -> list[int]:
    arg.append(42)
    return arg


class BuggyClass:
    items: list[int] = []

    def append(self, x: int) -> None:
        self.items.append(x)

```

There's two similar but different problems here:

- `buggy_fn` incorrectly uses the same list instance for each execution of the function,
- `BuggyClass` incorrectly shares the same instance of `items` across all class instances. 


Let's see what various linters have to say about this code.

## Tests

First, `flake8 7.2.0` which is what my project uses:

```console
$ uvx flake8 sample.py
$
```

Nothing by default, but there's a plug-in:

```console
$ uvx --with flake8-bugbear flake8 sample.py
sample.py:1:31: B006 Do not use mutable data structures for argument defaults.  They are created during function definition time. All calls to the function reuse this one instance of that data structure, persisting changes between them.
```

Better! It flagged `buggy_fn`, but it still didn't flag `BuggyClass`.

Next, let's try `pyright 1.1.400`. Pyright is a type checker but also does some misc diagnostics.

```console
$ uvx pyright sample.py
0 errors, 0 warnings, 0 informations 
```

What if we tried more power?

```console
$ echo '{"typeCheckingMode": "strict", "strict": ["*"]}' > pyrightconfig.json
$ uvx pyright sample.py
0 errors, 0 warnings, 0 informations 
```

Not much. How about `mypy 1.15.0` then? Mypy is more strictly focused on types, but let's try:

```console
$ uvx mypy --strict sample.py
Success: no issues found in 1 source file
```

How about the classic `pylint 3.3.7`?

```console
$ uvx pylint sample.py
************* Module sample
sample.py:1:0: C0114: Missing module docstring (missing-module-docstring)
sample.py:1:0: C0116: Missing function or method docstring (missing-function-docstring)
sample.py:1:0: W0102: Dangerous default value [] as argument (dangerous-default-value)
sample.py:6:0: C0115: Missing class docstring (missing-class-docstring)
sample.py:9:4: C0116: Missing function or method docstring (missing-function-docstring)
sample.py:6:0: R0903: Too few public methods (1/2) (too-few-public-methods)

------------------------------------------------------------------
Your code has been rated at 1.43/10 (previous run: 0.00/10, +1.43)
```

I never used Pylint on a production project but it actually found one of two issues!

Finally, let's look at `ruff 0.11.10`, our promising Rust-powered contestant.

```console
$ uvx ruff check sample.py
All checks passed!
```

Nothing on default settings, but let's enable some optional rules:

```console
$ uvx ruff check --extend-select B,RUF sample.py
sample.py:1:31: B006 Do not use mutable data structures for argument defaults
  |
1 | def buggy_fn(arg: list[int] = []) -> list[int]:
  |                               ^^ B006
2 |     arg.append(42)
3 |     return arg
  |
  = help: Replace with `None`; initialize within function

sample.py:7:24: RUF012 Mutable class attributes should be annotated with `typing.ClassVar`
  |
6 | class BuggyClass:
7 |     items: list[int] = []
  |                        ^^ RUF012
8 |
9 |     def append(self, x: int) -> None:
  |

Found 2 errors.
No fixes available (1 hidden fix can be enabled with the `--unsafe-fixes` option).
```

Okay, `ruff` was the only linter here that flagged both the function and the class! But we had to enable some optional checks: what are they?

[The list of all Ruff rules](https://docs.astral.sh/ruff/rules/) is very comprehensive (over 800 rules when I'm reading it). Instead of leveraging plug-ins, all rules are implemented within Ruff itself and assigned new codes. Specifically:

- Prefix `B` is for rules imported from flake8-bugbear (that we used earlier)
- Prefix `RUF` is for Ruff-specific rules that weren't imported from any existing linter.

Neither is enabled by default.

I didn't know these prefixes before; I cheated and ran `ruff check --select=ALL` before to see what sticks. The output with `--select=ALL` is more noisy because it also has some errors from `pydocstyle` (prefix `D`).

## Conclusions

I feel I now have a good justification to switch the project over to Ruff, but also I learned that I shouldn't necessarily trust Ruff's defaults and it's worth the shot to go over the rule list and find some additional ones that make sense to enable. The default set is better than nothing, but it's more conservative than my expectations for production code.
