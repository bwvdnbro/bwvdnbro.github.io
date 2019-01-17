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
curve just traverses the space row by row, the Hilbert curve has a twist 
in it. As a result, the distance between successive points in the 
Hilbert curve is always the same, while for the Morton curve it is not. 
This distinction remains for higher level curves.

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
level 5 space-filling curves:

![Level 5 space-filling curves](/assets/images/morton_hilbert_5.png)

You can see that the curves indeed start to fill the entire square. A 
curve with a high enough level should be able to fill any finite space.

# Why are space-filling curves useful?

One of the main reasons I ever started looking into space-filling curves 
is because of their use in *spatial domain-decomposition*. Imagine that 
you have a large number of points that are somewhat arbitrarily ordered 
in our square space, like this:

![A square with randomly positioned points](/assets/images/uniform_points.png)

Spatial domain-decomposition deals with the following question: what is 
the best way to divide this set of points into an arbitrary number of 
$$G$$ groups, so that points in the same group are guaranteed to be close 
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
star and every other star in the cluster. If you have $$N$$ stars, then 
the computation for every star would involve $$(N-1)$$ other stars, and 
you would have $$N(N-1)$$ computations in total. For the specific example 
shown above, this is not too bad. But things get a bit tricky if $$N$$ 
gets very big...

They get tricky for two reasons: (1) the number of computations grows 
really fast as $$N$$ gets bigger, (2) if $$N$$ gets really big, the stars 
might no longer fit on a single computer. This might sound very 
unlikely, but actually happens a lot these days, as large N-body 
simulations can go up to $$N=10^{13}=10,000,000,000,000$$!

To deal with the growing number of computations, astrophysicists have 
developed clever algorithms. They realised for example that the force 
for a given star is usually dominated by close neighbours of that star, 
and that the forces due to stars that are further away can be 
approximated reasonably accurately by grouping them together, 
drastically decreasing the number of computations per star. To deal with 
$$N$$ too big to fit on a single computer, they started using parallel 
computers that consist of many computers linked together. Each computer 
stores only a fraction of the $$N$$ stars, and sends information about 
these to the other computers when necessary.

Now we get back to the original point, because to divide the $$N$$ stars 
among the different computers, we need to perform a spatial 
domain-decomposition: we want to make sure that all computers have 
roughly the same number of stars (the same number of computations), but 
we also want to make sure that stars that are close together are 
generally found on the same computer, because that reduces the amount of 
communication necessary during the computation.

And now for the answer: what we could try to do is fit a space-filling 
curve through the $$N$$ stars (let's revert to calling them *points*). 
This will effectively map the 2D distribution of points onto a 1D line, 
so that we can assign a unique number to each point that we can compare 
with other points to sort them. Additionally, we know that points that 
are close on the space-filling curve are generally also close in space 
(especially for the Hilbert curve). If we know cut up the space-filling 
curve into $$G$$ (almost) equal pieces, we accomplish the desired 
spatial decomposition:

![Spatial domain-decomposition along a Hilbert 
curve](/assets/images/hilbert_domains.png)

More about this later...

# How do we compute space-filling curves?

As I mentioned before, space-filling curves are inherently linked to a 
level of subdivision of the space in which they are defined. This level 
defines the number of points in the space, and also tells you how far 
you have to recurse when constructing the curve.

To compute a space-filling curve, we hence need to discretize our space 
to some level. The easiest way to do this is by mapping the coordinates 
of all points in the space to integer values with a fixed number of 
bits, say $$N_b$$. If we are in a square space with side lenght $$L$$, 
then the integer coordinates for a point with coordinates $$(x,y)$$ are

$$(x_i, y_i) = \left( \frac{x}{L} 2^{N_b}, \frac{y}{L} 2^{N_b} 
\right).$$

Note that if $$0 \leq{} x \lt{} L$$, then this guarantees $$0 \leq{} x_i 
\lt{} 2^{N_b}$$, so the integer coordinates will exactly fit in an 
integer value with $$N_b$$ bit precision.

Both the Morton and the Hilbert space-filling curve can then be computed 
by using these integer coordinates and mapping them to an integer *key* 
with $$2N_b$$ bits. The key encodes the order of the points along the 
space-filling curve, and the space-filling curve can be generated by 
sorting the points on the key and the drawing the connections between 
the point in that order. This is exactly how the images above were made.

The difference between the Morton and Hilbert curves is the way their 
key is computed. For the Morton curve, the key, $$k$$, is computed by 
*interleaving* the bits of $$x_i$$ and $$y_i$$. This means that we run 
through the bitwise representation of the two integer coordinates and 
one by one strip their bits and put them in the key. So if

$$x_i = \color{blue}{10111}$$

and

$$y_i = \color{red}{00100}$$

then

$$k = \color{red}{0} \color{blue}{1} \color{red}{0} \color{blue}{0} 
\color{red}{1} \color{blue}{1} \color{red}{0} \color{blue}{1} 
\color{red}{0} \color{blue}{1}.$$

We can understand this if we think about a single level of the curve, or 
hence a single bit of $$x_i$$ and $$y_i$$ and the 2 corresponding bits 
of $$k$$. For the Morton curve, each level is equivalent, so this would 
be equivalent to a simple level 1 curve. For a level 1 curve, there 
would be 4 points in the space:

$$(x_0, y_0) = (0, 0),$$

$$(x_1, y_1) = (0, 1),$$

$$(x_2, y_2) = (1, 0),$$

$$(x_3, y_3) = (1, 1).$$

Now it is very easy to see that the (single) bit of $$x_i$$ tells us in 
which half of the square we are in the horizontal direction, while the 
same is true for $$y_i$$ in the vertical direction. The characteristic 
inverse Z-shape of the Morton curve means that we want the $$x_i$$ to 
make up the lower part of the 2-bit key, while $$y_i$$ makes up the high 
part, so that we first traverse all the bottom horizontal points and 
then all top ones.

Now suppose we were to move on to a level 2 curve. As we saw before, 
this curve follows the overall path of the level 1 curve, but has 
smaller copies of the curve for each of the points in the original 
curve. To make sure the key still follows the overall path of the level 
1 curve, the level 1 key has to make up the highest part of the level 2 
key. However, in (bitwise) integer coordinates, the original level 1 
points are now

$$(x_0, y_0) = (00, 00),$$

$$(x_1, y_1) = (00, 10),$$

$$(x_2, y_2) = (10, 00),$$

$$(x_3, y_3) = (10, 10),$$

since we added a level in our integer coordinate conversion formula and 
hence an additional factor 2 (which means we shift all bits 1 position 
to the left and add an extra 0-bit at the end). So the highest bit of 
the level 2 key is determined by the highest bits in the integer 
coordinates.

The new integer coordinates for the original level 1 curve points are 
each only 1 in 4 points for their specific smaller square, as the new 
0-bit can be replaced by a 1-bit (and there are 4 available options). In 
fact, the situation is exactly the same as for the level 1 curve, so we 
can get the new low part of the level 2 key by doing the same we did for 
the level 1 key. And the same is true for higher level keys.

Because computing a Morton key only requires bitwise operations, it can 
be done very efficiently on a computer. Below is a snippet of Python 
code that computes the Morton key for two integer coordinates `x` and 
`y`:

```
def get_morton_key(level, x, y):
  key = 0
  for i in range(level):
    key |= (y & 1) << (2 * i + 1)
    key |= (x & 1) << (2 * i)
    x >>= 1
    y >>= 1
  return key
```

# What about the Hilbert curve?

Computing the Morton key simply requires you to interleave the bits of 
the integer coordinates of the points in your space. For the Hilbert 
key, things are a little bit more complicated. The reason for this are 
the twists in the Hilbert curve which require a rotation when going from 
one level to the next. Which means that the level 2 part of the Hilbert 
key can only be computed if you know what the level 1 part of the key 
is... Fortunately, we only require one level of extra knowledge compared 
to the Morton curve; the rotation required for a given level only 
depends on the level above.

There are a few ways we could go about this...
