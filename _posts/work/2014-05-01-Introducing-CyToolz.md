---
layout: post
title: Introducing CyToolz
tagline: Functional Python, now in C
category : work
draft: true
tags : [SciPy, scipy, Python, Programming]
---
{% include JB/setup %}

**tl;dr: We reimplement PyToolz, a functional standard library, in Cython.
It's fast.**

**This post highlights work done by [Erik N. Welch](http://github.com/eriknw/).
When I say "we" below, I really mean "Erik"**

[Last year](http://matthewrocklin.com/blog/work/2013/10/17/Introducing-PyToolz/)
I introduced [PyToolz](http://toolz.readthedocs.org/en/latest/), a library that
provides a suite of utility functions for data processing commonly found in
functional languages.

~~~ Python
>>> from toolz import groupby

>>> names = ['Alice', 'Bob', 'Charlie', 'Dan', 'Edith', 'Frank']
>>> groupby(len, names)
{3: ['Bob', 'Dan'],
 5: ['Alice', 'Edith', 'Frank'],
 7: ['Charlie']}
~~~

Over the last year a number of [excellent
contributors](https://github.com/pytoolz/toolz/blob/master/AUTHORS.md) have
benchmarked and tuned these functions to the point where they often beat the
standard library.  When you couple these tuned functions with the power of pure
Python data structures you get a nice analytics platform.  In my experience
`toolz` is often fast enough even for large streaming data projects.


CyToolz
-------

Personally, I'm fine with fast Python speeds.  Erik on the other hand, wanted
unreasonably fast C speeds so he rewrote `toolz` in Cython;  he calls it
[CyToolz](http://github.com/pytoolz/cytoolz/).

~~~~~~~~~~~~Python
>>> import toolz
>>> import cytoolz

>>> timeit toolz.groupby(len, names)            3.19 µs
>>> timeit cytoolz.groupby(len, names)           721 ns
~~~~~~~~~~~~

For data structure bound computations this competes with Java.  It does so
using pure, native Python data structures.  This differs from the NumPy/Pandas
approach which runs Cython code on new C data structures.  CyToolz uses Cython,
but on your basic Python data structures.

Erik just released CyToolz yesterday.  Get it while it's hot

    $ pip install cytoolz


How?
----

Q: Hey Erik, why is CyToolz able to increase performance on Pure Python data
structures?  I thought that Cython was mostly about leveraging C type
information such as you find on numpy arrays.  How are you able to speed up
generic Python code?

A: ...


Example: `merge`
----------------

~~~~~~~~~~~~Python
>>> dicts = {'one': 1, 'two': 2}, {'three': 3}, {'two': 2, 'four': 4}
>>> toolz.merge(dicts)
{'one': 1, 'two': 2, 'three': 3, 'four': 4}

>>> timeit toolz.merge(dicts)                   1.76 µs
>>> timeit cytoolz.merge(dicts)                  264 ns
~~~~~~~~~~~~


Why?
----

I love NumPy and Pandas, so why do I use toolz?  Two reasons

1.  Streaming analytics - Python's iterators and Toolz support for lazy operations allows me to crunch over Pretty-Big-Data without the hassle of setting up a distributed machine.
2.  Trivial parallelism - The functional constructs in PyToolz, coupled with the promise of [total serialization](http://matthewrocklin.com/blog/work/2013/12/05/Parallelism-and-Serialization/), make parallelizing PyToolz applications to multicore or cluster computing trivial.  See the [toolz docs page](http://toolz.readthedocs.org/en/latest/parallelism.html) on the subject.





Testing
-------

CyToolz perfectly satisfies the PyToolz test suite.  This is especially notable
given that PyToolz has 100% coverage.

PyToolz is stable enough that we were able to just copy over the tests en
masse.  We'd like to find a cleaner way to share test suites between codebases.


Pluck
-----

Many Toolz operations provide functional ways of doing plain old Python
operations.  The `pluck` operation gets out items from a collection.

~~~~~~~~~~~~Python
>>> data = [{'id': 1, 'name': 'Cheese'}, {'id': 2, 'name': 'Pies'}]
>>> list(pluck('name', data))
['Cheese', 'Pies']
~~~~~~~~~~~~

In PyToolz we work hard to ensure that we're not much slower than straight
Python.

~~~~~~~~~~~~Python
>>> timeit [item[0] for item in data]
10000 loops, best of 3: 54.2 µs per loop

>>> timeit list(toolz.pluck(0, data))
10000 loops, best of 3: 62.9 µs per loop
~~~~~~~~~~~~

CyToolz just beats straight Python.

~~~~~~~~~~~~Python
>>> timeit list(cytoolz.pluck(0, data))
10000 loops, best of 3: 26.7 µs per loop
~~~~~~~~~~~~





A note on Functional Programming
--------------------------------

PyToolz integrates functional principles into traditional Python programming.
CyToolz supports these same functional principles, in the same workflow, but
now backed by C speeds

I started PyToolz because I liked Clojure's standard library but couldn't stay
on the JVM (I needed to interact with native code).  Originally I thought of
Python and PyToolz as a hack providing functional programming abstractions in
an imperative language.  I've now come to think of Python as a performant and
serious functional language in its own right.  While it lacks macros, monads,
or any sort of type system, it is amazing for 99% of the pedestrian programming
that I do day to day.


Conclusion
----------

The `toolz` functions are simple, fast, and a great way to compose clear and
performant code.  Check out [the docs](http://toolz.readthedocs.org/) and find
a function that you didn't know you needed, or a function that you needed,
wrote, but didn't benchmark quite as heavily as we did.

If you're already a savvy `toolz` user and want Cython speed then you'll be
happy to know that the cytoolz library is a drop in replacement.

    $ pip install cytoolz

~~~~~~~~~~Python
# from toolz import *
from cytoolz import *
~~~~~~~~~~

Most functions improve by 2x-5x with some fantastic exceptions.

Links
-----

Q: Hey Erik, what resources did you find helpful use when writing CyToolz?


Related Blogposts

*   [Introducing PyToolz](http://matthewrocklin.com/blog/work/2013/10/17/Introducing-PyToolz/)
*   [Verbosity](http://matthewrocklin.com/blog/work/2013/11/15/Functional-Wordcount/)
*   [Text Benchmarks](http://matthewrocklin.com/blog/work/2014/01/13/Text-Benchmarks/)
