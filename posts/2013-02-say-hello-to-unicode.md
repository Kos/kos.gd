<!--
.. title: Say hello to Unicode
.. slug: say-hello-to-unicode
.. date: 2013-02-09 21:52:19 UTC
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

# Basic explanations

Unicode is a <em>standard </em>that defines several things. Here is a handful of some most important facts about it:

- There are thousands of "code points" that represent characters in various alphabets, numbers, punctuation, various symbols, etc. Each of these has a name and a number. Here are some examples:
	- LATIN CAPITAL LETTER A
	- LATIN SMALL LETTER C WITH ACUTE
	- DIGIT SEVEN
	- MINUS SIGN
	- SUPERSCRIPT TWO
	- RIGHTWARDS ARROW
	- BLACK CHESS ROOK
- Different code points may share their appearance, or "glyph" (like OHM SIGN and GREEK CAPITAL LETTER OMEGA).
- Code points are grouped into blocks like "Basic Latin" or "Mathematial Operators". They also are divided into categories like "Letter, Uppercase".
- There are some "encodings" that provide a mapping between code points and their bitwise representation. The encodings have names like UTF-8, UTF-7 or UTF-32. Each of these has some interesting properties.
<!--more-->
- Code point numbers go from 0 up to 0x10FFFF. Most of them is still unassigned. This range is divided into 17 "planes", each 0x10000 characters long; the first is called the "basic Multilingual Plane" (BMP).
- Some of areas (0xU+E000..0xF8FF and whole two last planes) are the "Private Use Areas", this is where you can find codes for Tengwar or Klingon alphabets (unofficially defined).

A code point can be represented in many ways, for example:

- Its name: `INTEGRAL`
- Its number: 8747 or 0x222B or U+222B
- A sequence of bits in a particular encoding:
	- 111000101000100010101011 as UTF-8
	- 00101011001000100000000000000000 as UTF-32 little-endian
- Other conventions:
	- HTML entity: `&#8747;` or `&int;`
	- String literals in a programming language
		- e.g. `u'\u222b'` or `u'\N{INTEGRAL}'` in Python

There are many code points with special meaning, for example:

- Hard space or soft hyphen that indicates where a word can be hyphenated
- Replacement characters
- Combining characters, such as accents

A visible character can be "built" from a letter code point and some combining code points. It is collectively called a "grapheme cluster". Note that there may be several ways to write the same grapheme cluster: For example "ą" can be written as a single `LATIN SMALL LETTER A WITH OGONEK`, but just as well written as `LATIN SMALL LETTER A` + `COMBINING OGONEK`. Creative use of combining characters allows to form funny constructs like <span style="font-family: sans;">ṭ͓h̜̦͝i̸҉͇̯̘̜͕̰̼s̸̡͚͖͉̜͎̠͉͠ͅ</span>.

# Encodings

An encoding is a scheme that allows you to encode a particular code point into a sequence of bits. Unicode encodings all have the notion of a "code unit" - some fixed-width integer.

Note: "Unicode" isn't an encoding itself. It seems common to say "this file is in Unicode", esp. when referring to UTF-16, but that's misleading. Don't ever do that, pretty please. Hear me, Microsoft? Got that, LibreOffice team? Stop doing that and I'll buy you some żurek if you ever come and visit.

## UTF-8

UTF-8 uses one to four code units to encode a code point. It has some awesome properties that make it a very popular encoding (and indeed _the_ most popular on the web):

- Characters from ASCII look the same in UTF-8 (including NUL). An ASCII document can be correctly interpreted as UTF-8.
- The code units are bytes, so byte order isn't important. This makes UTF-8 neat when little-endian and big-endian machines talk to each other.
- Encoded character is never a subsequence of another character. This allows `printf` to correctly look for `%` while scanning bytes of an UTF-8 string. This also holds for NUL, which makes UTF-8 compatible with NUL-terminated convention.

Wider encodings (UTF-16, UTF-32) solve the endianess problem by allowing the document to be started with a "byte order mark" (U+FEFF) that allows to determine which variant is used (as the reverse, 0xFFFE, isn't a valid code point). The encoding names "UTF-16LE" or "UTF-16BE" refer to that. (Outside of this special job, U+FEFF code point normally serves as "ZERO WIDTH NO-BREAK SPACE".)

UTF-8 doesn't need the BOM, but it's still allowed. It's neither necessary nor recommended, but some Windows programs like Notepad use it by default. 

## UTF-16

UTF-16 uses one or two 16-bit code units per code point. This is enough bits to represent any code point in the Basic Multilingual Plane directly - that is, making code points equal to code units.

The higher code points use a bit more trickery, namely "surrogate pairs". Surrogates are special code points reserved in the BMP. High surrogates span U+D800..U+DBFF and low surrogates are contained in U+DC00..U+DFFF. They have no other meaning. Every code point outside the BMP is uniquely represented by a pair of those. Do the math: there are 0x400 low surrogates and 0x400 high - this gives 0x100000 possible combinations, just as many as there are code points outside of the BMP. Nice!

UTF-16 is somehow widely used in software and programming languages for historical reasons, but since Unicode evolved to its current form it's hard to find good use cases for it. It's a "halfway" format.

## UTF-32

UTF-32 encodes code points as 32-bit code units. This allows to encode any code point at all directly. with their value equal to the code point, and it might be considered the simplest encoding. Since the codespace ends at 0x110000-1, highest 11 bits are always zero.

While all encodings are fixed-width with respect to their code units, UTF-32 also has the property of being fixed-width with respect to code points. This isn't as useful as it sounds, though, as code points alone aren't much more meaningful to string processing than code units. ([UTF-8 Everywhere][u8e] makes a bold claim that *"the number of code points is irrelevant to almost any software engineering question"* and I'm so signing under that.)

# Unicode strings in programming languages

Most languages provide you some kind of a "string" type that is supposed to represent an arbitrary string of Unicode characters - an it generally can, as a matter of fact. There are some caveats here, though.

A string is conceptually a sequence of characters. This allows for asking:

- "how many characters does this string have?" 
- "show me the character X of that string".

To answer these questions consistently, whe have to define what is "a character"? Turns out that's not obvious. In a high-level language one might assume one "character" to be a code point, but this isn't generally the case (and there's nothing wrong in that).

Let's see:

## C, C++

There two most common cases here:

- Byte strings: Arrays of `char`s, like `char[]` or `std::basic_string<char>` (aka `std::string`)
- Multi-byte strings: Arrays of `wchar_t`s (usually 16-bit), including `std::basic_string<wchar_t>` = `std::wstring`.

Neither of these are Unicode-aware. The most logical conventions to use are:

- `UTF-8`-encoded strings in `char[]`s
- `UTF-16`-encoded strings in 16-bit `wchar_t`s

These two have the property of having 1 character equal to 1 code unit. Both encodings are variable width though, so one "character" alone isn't of any meaning in text processing.

## PHP

See C. PHP strings are bytestrings too, which is surprising for a high-level language.

This is often quoted as a disadvantage of PHP, but is it, really? There's no concensus on that. I uphold there's nothing wrong with bytestrings per se, as long as you are aware of their content's encoding and handle them consistently. The latter is not always the case with PHP libraries though, which is probably the real problem here.

## C#, Java

These languages have a `char` type that is said to represent "any Unicode character". This had indeed been the case a long time ago, before Unicode 2.0 extended the codespace beyond the BMP. Nowadays they can contain any UTF-16 code unit, which means either a "whole" code point from BMP or a surrogate. Code points from beyond the MBP need two `char`s.

This also means that:

- UTF-16 is the local concensus for string processing,
- The language won't prevent you from accidentally splitting a code point in half when extracting a substring, just like with raw bytestrings. Be careful!

## Python 2

Python has the the most curious situation around. I'll start with major version 2 (precisely, 2.2 ~ 2.7).

There are two types:

- `str` is a byte string. Smooth, cool and predictable.
- `unicode` is more interesting, however. Let's have a look.

Conceptually, `unicode` is a sequence of code points. There are several syntaxes to 

	u'Ω'
	u'\u03a9'
	u'\U000003a9'
	u'\N{GREEK CAPITAL LETTER OMEGA}'

There's also the function `unichr` that gives you any Unicode code point:

	>>> unichr(0x25)
	u'%'

Looks nice, but here comes the funny part: Depending on how the Python is configured before build, it stores the strings internally either as UTF-16 or as UTF-32. And the representation used actually changes how Python behaves.

Let's try to obtain a character that won't fit in a single UTF-16 code unit:

	>>> a = u'\N{MAHJONG TILE GREEN DRAGON}'
	>>> a
	u'\U0001f005'
	>>> len(a)

Depending on your build ("narrow" or "wide"), you'll obtain either 1 or 2! That depends whether your Python build has been configured with `--enable-unicode=ucs4`. Usually, Linux builds are wide and Windows builds are narrow.

We can further confirm that it's UTF-16 we're seeing:

	>>> a[0], a[1]
	(u'\ud83c', u'\udc05')
	>>> [hex(ord(c)) for c in a.encode('utf-16be')]
	['0xd8', '0x3c', '0xdc', '0x5']

While it's surprising, don't worry about that. Indexing and `len()` may work inconsistently depending on the build, but the `unicode` type is Unicode-aware and behaves correctly when encoding, decoding, looking for substrings, etc. Also note how `repr` correctly interprets the surrogate pair as a single code point.

If you're curious, read up the rationale in [PEP 261](http://www.python.org/dev/peps/pep-0261/) that introduced this curious solution back in 2001.

## Python 3

The major language update cleaned up the string types. The `str` was replaced by the new `bytes` type and `unicode` was renamed to the simple `str` - this is probably the first thing people learn when switching from 2 to 3.

The internal representation issue, however, hasn't been cleaned up until Python 3.3 and [PEP 393](http://www.python.org/dev/peps/pep-0393/). Only then was the distinction between narrow and wide builds dropped. In Python 3.0 through 3.2, the situation for `str` is the same as for `unicode` in Python 2.

# Conclusions

Pay attention to how your language represents strings and characters. Understand the differences between byte strings and "code unit" strings with an encoding assumed.

A "character" can be understood a vague term - it can be a code unit, code point, a grapheme cluster... Pay attention and choose the correct abstraction level for your problem.

UTF-8 is great, use it. (Concerned about storage size with non-Western text? Forget UTF-16, go UTF-8 + gzip. Seriously.)

# Recommended reading

- [UTF8 Everywhere][u8e] gives plenty of reasons to make UTF-8 your text encoding of choice.
- [The Absolute Minimum...][tam] by Joel Spolsky has an interesting overview on the subject. That said, I'm not really comfortable with his bold claim that "PHP using bytestrings makes it "darn near impossible to develop good international web applications" - but that's a story for another day.

[u8e]: http://www.utf8everywhere.org/
[tam]: http://www.joelonsoftware.com/articles/Unicode.html
