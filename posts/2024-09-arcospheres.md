<!--
.. title: Solving Arcospheres
.. slug: solving-arcospheres
.. date: 2024-09-03 12:00:00 UTC
.. tags:
.. category: dev
.. link:
.. description:
.. type: text
.. status: draft
-->

[Factorio][factorio] is renowned as a game that involves a lot of creative problem solving. One thing I did, however, not expect, is that I'm going to encounter a problem that will inspire me to stop playing the game, dust off my copy of "Introduction to Algorithms" and spend three weeks fiddling with graph algorithms in Python.

The problem appears in the [Space Exploration mod][se]. If you're planning to play this mod, beware, there are spoilers ahead. I captured my little adventure in this post, so that I can share the experience with anyone who enjoys a good programming puzzle, not necessarily a Factorio player.

<!--more-->

## The problem

The “Arcospheres” are a limited resource that you need to use in some late-game production chains. An arcosphere can be in one of eight states:

![eight colored balls labeled with greek letters](/images/arco/spheres.svg)

There is ~~eight~~ ten transformations available, which allow you to change of your spheres, but they always work on groups of two (or four) so you have to plan your transformations.

![complete list of transformations](/images/arco/recipes.svg)

Finally, there’s a few “production recipes” that take a few arcospheres (along with other ingredients) and produce something useful. The production recipes return the arcospheres in different state, than they entered, but there’s also a random factor involved.

I’m focusing on a particular production recipe that makes a Naquium Tesseract:

![tesseract recipe](/images/arco/tess-recipe.svg)

The questions I seeked to answer are:

- given a production recipe, what is the **minimum number of spheres** needed to keep production running indefinitely?
- what is the specific **sequence of steps** needed to keep the production going?

Most players gather a lot of arcospheres and build a system that keeps production going using simple heuristic rules, which is probably the way intended by the mod's author. I decided to embark on a mini-journey to find an exact solution analytically.

## Graph search

There’s more than one way to go around this, but I decided to model the state as a graph problem. If we assume that we’re working with, say, 5 spheres, then the possible set of states (graph nodes) is the set of 5-length combinations with repetition of the possible 8 sphere types.

![example states](/images/arco/combinations.svg)

(Another way to put it is that the set of possible states is the set of all possible length-N multisets of the 8 sphere types.)

For a given set of K spheres from N types, there are $N+K-1 \choose K$ possible states, which gives:

- 792 states for 5 spheres,
- 1716 states for 6 spheres.

My initial approach was like so:

- find an example “outcome state“ that we can get as result of the production recipe,
- find a sequence of transformations that leads from that state back to a different “ingredient state” which can be used to produce the same “outcome state”

This is a bit similar to finding a cycle in a graph. I implemented Dijkstra graph search and got an example result quickly for K=5:

![result](/images/arco/steps.svg)

I was very excited at this point; I went back to the game and produced a few tesseracts this way. Now I just had to “somehow account for” non-determinism of the production recipe: what if instead of “θεφ” we got “φγω” instead?

That was a bit optimistic…

A few long weeks ensured, during which I went through a few approaches and found out that the problem is actually hard to represent as a graph problem. I tried looking at the reversed graph which should be deterministic, but that didn’t help - I found cycles but I didn’t find them usable. I also toyed with a graph-like concept that has both “edges” and “pseudo-edges” (to represent the non-deterministic transforms) and tried to invent a traversal algorithm for that, which was pretty promising but got complicated quickly. Eventually I settled for a much simpler option.

## State classification

My last approach used graph search but only as a part of the whole solution. First I identified some subsets of the set of states:

- *“ingredient states”* which include the whole set of ingredients for the production recipe,
- *“product states”* which can appear as a result of the production recipe.

![ingredient and product states](/images/classify.svg)

Now for each product state, I identified the set of all possible ingredient states which are reachable using *some series of transformations*. I used Dijkstra for this step. A regular BFS graph search would be sufficient, but Dijkstra was more convenient (and not just because I had it implemented from previous steps).

Dijkstra’s algorithm, when ran from a source node, finds the shortest path from the source to EACH other node. The output tells you both which nodes are reachable, and each node’s “predecessor” in the shortest path (which lets you reconstruct this path by traversing it all the way to the source). We’ll use it later!

My output for this step is a table that tells me different ways how a product state can "loop back" to an ingredient state, so that production can continue.

![product states with their ingredient states](/images/classify2.svg)

In the next step, I started marking some ingredient states as “bad”. Note how every ingredient state randomly leads to one of two product states. A state is considered “bad” if at least one of its two product states is a “blind alley” that doesn’t lead to any non-bad ingredient state. (That’s a lot of negations!)

Note that marking one ingredient state as “bad” can potentially make some product states fall under “blind alley”, which can in turn be sufficient to make other ingredient states “bad”. I call this algorithm “tree shaking” because every iteration cuts off some possible ingredient states.

After a few iterations there are two possible outcomes:

- A. Every ingredient state has been marked as “bad”: This means that for the given parameters, we can’t guarantee running production indefinitely,
- B. Tree shaking doesn’t mark any additional states as “bad”: This means that the remaining ingredient states are “good” and starting production from them is guaranteed to lead to a cycle.

For tesseract production, I found that for K=5 there’s no solution, but for K=6 out of 120 ingredient states only 18 are bad! So if we find a way to avoid producing anything from these states, we can keep the production going forever!

(This sounds like a quick conclusion, but it took me embarassingly long to get here! I had a bug in my problem formulation: I only took into account 8 out of 10 possible sphere transformations, so the graph I have been working on was much more sparse. For example, for K=12, I ended up with all 6435 ingredient states marked as bad, and I started to doubt if the problem is even solvable!)

## The result

Let’s revisit what I had at this point:

- I had 104 states identified as “good ingredient states”. These are the ones where I’m supposed to produce a tesseract.
- Each of these states could lead to one of two product states. I have already verified that each product state obtainable this way could be *somehow* translated back to another “good ingredient state”. (Often there would be multiple ingredient states to choose from!)
- Also remember how I used Dijkstra to identify the possible ingredient states for each Product state? Another output of Dijkstra’s algorithm is the map of predecessors, which told me which exact transformations should be applied to a product state to map it back to the ingredient state of my choice.

In order to make a useful solution out of this, I needed to formulate it in a usable format. For me this was some kind of lookup table that tells me: “given a particular state, what transformation should I apply?“ Idea is that the result of that operation would be another state from the lookup table, which lets me apply it indefinitely without ever getting stuck.

The way to do this, incidentally, also looks like a graph search:

- pick any ingredient state as starting point
- while there’s any queued ingredient states…
    - (skip if seen already)
    - add instruction to use the production recipe from that state
    - for each of 2 corresponding product states…
        - pick an arbitrary reachable good ingredient state
        - from Djikstra results, add instructions how to navigate from that state back to another ingredient state
        - enqueue that ingredient state

The result in my case looks like so (when represented as a table), or like so when rendered as a graph. I noticed that I get a valid result no matter how I pick an ingredient state from available choices of each product state, but if I pick the lexicographically smaller ingredient state, then the resulting handbook tends to be smaller (less states / rows). I’m sure there’s a way to be smarter about this and e.g. find a handbook that optimizes for producing as few steps as possible on average, but I didn’t go there.

With the handbook in hand, just one question remains: How do I build this in Factorio? After all, this is a game of automation, so I’m supposed to have a machine that follows the handbook for me, right? 

This part was simpler in comparison, but just as interesting. I’ll leave it for a follow-up post.

[factorio]: https://factorio.com/
[se]: https://mods.factorio.com/mod/space-exploration