---
layout: post
title: "Space-filling curves"
date: 2019-01-14
description: An overview of what space-filling curves are, and why they are (not) useful
image: /assets/images/space_filling_curve.png
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Geometry
---

One of the perks or issues of being a researcher (pick your favourite) 
is that what you do does not necessarily always result in something 
useful. You could of course argue semantics and ask yourself *what is 
useful?* (definitely if you are an astronomer and think about your 
field), but I think it is easy to answer that question from a personal 
point of view: something useful is something that gives you the feeling 
that it was worth it.

The topic of this post is one of these things that I spent a lot of time 
on at some point (during my PhD, when I could afford such 
extravagances), but that in hindsight was not useful. Interesting, but 
definitely not useful. This post is an attempt to maybe make it useful 
after all, by at least sharing what I know and what I figured out. And 
by then explaining why you should definitely (not) spend time on this 
same topic.

The topic of this post are *space-filling curves*. These are geometrical 
constructs that almost automatically show up whenever you are breaking 
down space into regular blocks, and are very easy to work with or think 
about when you are dealing with bit series. And they are in a way 
incredibly beautiful and elegant. But definitely not as useful as I 
thought they were. Here is a short introduction.

# What are space-filling curves?

Space-filling curves are, as the name says, curves that fill space. This 
means that if you define a space (for example a square), then a 
space-filling curve in that space is a single curve that moves through 
every *point* of that space, without ever visiting the same point twice. 
Of course, this only works if the square (or more generally, the space) 
is a finite space, i.e. has a finite number of points in it. Or at least 
has a *countable* number of points in it. Let's not be too mathematical 
about this; I will only ever consider finite spaces.

The square space could for example only consist of 4 points: the 
midpoints of the 4 smaller squares you get when you subdivide the square 
into 4 equal parts. In this case, a space filling curve could look like 
this:

![A first order Morton space-filling curve](/assets/images/morton_1.png)

Or like this:

![A first order Hilbert space-filling curve](/assets/images/hilbert_1.png)

The two curves above are both *level 1* space-filling curves, since they 
fill the space that you get by subdividing the square once into 4 equal 
smaller squares. The first one is what we call a *Morton* or *Z-order* 
space-filling curve, the second is what we call a *Hilbert* curve. You 
can already see the difference between both curves: while the Morton 
curve just traverses the space in the same way as you would read a text 
(i.e. row-by-row), the Hilbert curve has a twist in it. As a result, the 
distance between successive points in the Hilbert curve is always the 
same, while for the Morton curve it is not. This will always be the 
case.

If we subdivide the square again (we subdivide each smaller square into 
4 equal even smaller squares), we end up with a level 2 subdivision, in 
which we can define level 2 space-filling Morton and Hilbert curves:

![Level 2 space-filling curves](/assets/images/morton_hilbert_2.png)

For the Morton curve, it is immediately clear what we do: we just copy 
the original level 1 Morton curve into each level 1 square, and then 
order these along the original level 1 curve to make up the total curve. 
When we reach the end of the second level 1 square, we need to jump to 
the third level 1 square.

For the Hilbert curve, things are a bit more complicated. The level 1 
squares also contain a copy of the level 1 curve, but unlike for the 
Morton curve, this copy is rotated, so that the endpoint of each level 1 
square matches the startpoint of the next square along the level 1 
ordering. By doing this, we guarantee that the level 2 curve still 
satisfies the property that the distance between succesive points is 
always the same. Notice that the rotations for the different squares are 
different and depend on the position of that square along the level 1 
curve.

Once we know how to go from a level 1 to a level 2 space-filling curve, 
we know everything we need to move to even higher levels. Here are some
level 10 space-filling curves:

![Level 10 space-filling curves](/assets/images/morton_hilbert_10.png)

You can see that the curves indeed start to fill the entire square. A 
curve with a high enough level should be able to fill any finite space.

# Why are space-filling curves useful (part 1)?

One of the main reasons I ever started looking into space-filling curves 
is because of there use in *spatial domain-decomposition*. Imagine that 
you have a large number of points that are somewhat arbitrarily ordered 
in our square space, like this:

![A square with randomly positioned points](/assets/images/uniform_points.png)

Spatial domain decomposition deals with the following question: what is 
the best way to divide this set of points into an arbitrary number of 
$G$ groups, so that points in the same group are guaranteed to be close 
to each other? You might notice that this question is not formulated 
very rigorously. I will come back to that later (spoiler: it is one of 
the reasons space-filling curves are not that useful).

Before answering this question, let me first explain why a scientist 
(and more importantly: an astronomer!) would even ask such a question. 
The reason for this are large (N-body) simulations. Just imagine that 
the points above are stars in a cluster, and that you want to compute 
the gravitational forces that act on these stars. In principle, to 
compute these forces, you would need to loop over the stars, one by one, 
and for each of them compute the gravitational interaction between that 
star and every other star in the cluster. If you have $N$ stars, then 
the computation for every star would involve $(N-1)$ other stars, and 
you would have $N(N-1)$ computations in total. For the specific example 
shown above, this is not too bad. But things get a bit tricky if $N$ 
gets very big...

They get tricky for two reasons: (1) the number of computations grows 
really fast as $N$ gets bigger, (2) if $N$ gets really big, the stars 
might no longer fit on a single computer. This might sound very 
unlikely, but actually happens a lot these days, as large N-body 
simulations can go up to $N=10^{13}=10,000,000,000,000$!

To deal with the growing number of computations, astrophysicists have 
developed clever algorithms. They realised for example that the force 
for a given star is usually dominated by close neighbours of that star, 
and that the forces due to stars that are further away can be 
approximated reasonably accurately by grouping them together, 
drastically decreasing the number of computations per star. To deal with 
$N$ too big to fit on a single computer, they started using parallel 
computers that consist of many computers linked together. Each computer 
stores only a fraction of the $N$ stars, and sends information about 
these to the other computers when necessary.

Now we get back to the original point, because to divide the $N$ stars 
among the different computers, we need to perform a spatial domain 
decomposition: we want to make sure that all computers have roughly the 
same number of stars (the same number of computations), but we also want 
to make sure that stars that are close together are generally found on 
the same computer, because that reduces the amount of communication 
necessary during the computation.

And now for the answer: what we could try to do is fit a space-filling 
curve through the $N$ stars (let's revert to calling them *points*). 
This will effectively map the 2D distribution of points onto a 1D line, 
so that we can assign a unique number to each point that we can compare 
with other points. Additionally, we know that points that are close on 
the space-filling curve are generally also close in space (especially 
for the Hilbert curve). If we know cut up the space-filling curve into 
$G$ (almost) equal pieces, we accomplish the desired spatial 
decomposition:

![Spatial domain-decomposition along a Hilbert 
curve](/assets/images/hilbert_domains.png)

More about this later...

# How do we compute space-filling curves?

As I mentioned before, space-filling curves are inherently linked to a 
level of subdivision of the space in which they are defined. This level 
defines the number of points in the space, and also tells you how far 
you have to recurse when constructing the curve.

To compute a space-filling curve, we hence need to discretize our space 
to some level. One way to do this is by 
