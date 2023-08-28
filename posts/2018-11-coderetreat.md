<!--
.. title: 2018 Coderetreat & Game of Life
.. slug: 2018-coderetreat
.. date: 2018-11-28 20:00:00 UTC
.. tags:
.. category: dev
.. link:
.. description:
.. type: text
-->


Nov 17 was the [Global Day of Coderetreat](https://www.coderetreat.org/). I participated for this kind of event for the first time.


> Coderetreat is a day-long, intensive practice event, focusing on the fundamentals of software development and design, away from the pressures of 'getting things done'.

The essence of the format is solving the same problem multiple times in consecutive sessions, each time with a different person.

I can confirm a [saying](https://twitter.com/83TB/status/1063773762078867456) that it's "the opposite of a hackathon". In both you have a chance to reset your mind by adapting a completely different mode of work for the duration of the event. But they’re also on the opposite ends of a spectrum: While hackathons are all about focusing on delivering value quickly (forgoing methodology), during a Coderetreat the focus is on methodology and there’s no value delivered by definition - the problem is chosen to be mundane, and the outcome ends up ritually deleted after each session. The only trace is what you learned.

<!--more-->

## Coding with a stranger

I found this to be the most entertaining aspect of Coderetreat. Learning to work together with a new person is a Big Thing for me, but also a step out of my comfort zone. This format forces me to take this step multiple times, each time with someone else.

Each session allows to exchange some working habits:

- tool setup (editor, shell, utilities)
- approach towards testing
- rhythm of work (design upfront vs emergent design, …)

Even with short sessions, this ends up effective because you naturally focus on the most important aspect to you.

Interestingly, experience gap in either direction is never an obstacle: either you’ll learn something technical, or you’ll learn to convey ideas better.

## A vectorised life

I was worried that implementing the same thing twice will be boring, so I intuitively looked for ways around that. During the first session we implemented [Game of Life](https://bitstorm.org/gameoflife/) (this event’s theme) with classic [ping-pong approach](http://wiki.c2.com/?PairProgrammingPingPongPattern):


> person A writes a test
> person B writes just enough code to makes it pass
> person A improves a thing about the code
> person B writes a test …

A large amount of the time was spent on implementing and testing a `Board` class which was really just a wrapper over a 2D array of booleans - it surprised me how many design decisions were there on the way.

In the second session, I wanted to mix things up, so I suggested using an existing board implementation (we picked `numpy`) and focusing on operations. While juggling 2D matrices around, we couldn’t help but notice fun things along the way.

We divided the implementation into two steps for easier testing:


1. For each cell, calculate the count of neighboring alive cells
2. For each cell, given its value and count of neighboring cells, determine its status

Step 1 can be naively implemented using a set of nested `for` loops and this is roughly what we initially went for:


```python
def get_neighbour_counts(array):
    neighbour_indices = [(a, b) for a in (-1, 0, 1) for b in (-1, 0, 1) if a or b]
    counts = np.zeros(array.shape, dtype=int)
    range_x = range(array.shape[0])
    range_y = range(array.shape[1])
    for x in range_x:
        for y in range_y:
            neighbours = 0
            for dx, dy in neighbour_indices:
                if x + dx not in range_x or y + dy not in range_y:
                    continue
                neighbours += array[x + dx, y + dy]
            counts[x, y] = neighbours
    return counts
```

Now for the second step, we observed it’s really a simple logic function (per cell) that we could write and test separately:


```python
def get_next_state(state, neighbours):
    return (neighbours == 3) or (state and neighbours == 2)
```

So now we just had to apply that over the whole matrix. But since basic arithmetic in `numpy` is broadcast over the whole matrix, we could use the code directly! For example, here’s what happens if you compare an `array` to a number in `numpy`:


```python
In [3]: a
Out[3]: array([0, 1, 2, 3, 4])
```

```python
In [4]: a > 1
Out[4]: array([False, False,  True,  True,  True])
```

```python
In [5]: a == 4
Out[5]: array([False, False, False, False,  True])
```

Given that, we could take the function that worked on scalars and just pass it a matrix instead, relying on duck typing. (We need to change logical operators to equivalent bitwise operators though - this is what `numpy` uses for element-wise logic. Also mind the different operator precedence!)


```python
def get_next_state(state, neighbours):
    return (neighbours == 3) | (state & (neighbours == 2))
```

Written like this, it can be called either with both scalars and arrays.

Combining the two, the main function looks like:


```python
def life(array):
    return get_next_state(array, get_neighbour_counts(array))
```

Having vectorised the second step, we wanted to revisit the first one too. Turns out the operation we wrote earlier was just a crude implementation of [matrix convolution](http://setosa.io/ev/image-kernels/) which is conveniently available in `scipy`. This means we can replace the nested-loop code with a two-liner:


```python
def get_neighbour_counts(array):
    kernel = np.array([[1, 1, 1], [1, 0, 1], [1, 1, 1]], dtype=int)
    return convolve2d(array, kernel, mode="same", fillvalue=False)
```

(At this point I was extremely happy to have written a decent suite of tests. The moment we completely replaced the implementation and saw green tests was the highlight of the evening!)

Bonus find was that we could build different variants of Life by tweaking the variant of convolution used. Here’s a version that assumes all cells outside of the are alive:


```python
def get_neighbour_counts(array):
    kernel = np.array([[1, 1, 1], [1, 0, 1], [1, 1, 1]], dtype=int)
    return convolve2d(array, kernel, mode="same", fillvalue=True)
```

And here’s one that assumes a wrapped (toroidal) surface:


```python
def get_neighbour_counts(array):
    kernel = np.array([[1, 1, 1], [1, 0, 1], [1, 1, 1]], dtype=int)
    return convolve2d(array, kernel, mode='same', boundary='wrap')
```


![Glider animation with wrap-around](https://d2mxuefqeaa7sj.cloudfront.net/s_98591C77C443198231BBBC6D781431A45DB721180A1DBBF0C00F4C8C36F484B5_1543437855287_glider2.gif)


Here’s the [gist with complete code](https://gist.github.com/Kos/6df986d4369b7f80dd81d47dd281f149).

