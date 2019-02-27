---
layout: post
title: "Graph decomposition"
description: >-
  A brief introduction to graph decomposition and its applications in domain
  decomposition for distributed memory computations.
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
  - Code Development
  - Tools
---

I previously mentioned [space-filling curves and their application in 
scientific computing]({% post_url 2019-01-18-space-filling-curves %}), 
and concluded that post with the remark that space-filling curves are 
actually not the best solution if you want to distribute some large 
computational task over the separate nodes in a distributed memory 
parallel environment. In this post, I will show a more appropriate way 
to do this. And at the same time introduce another important 
mathematical concept: graph decomposition.

# A short recap

*Domain decomposition* is the technical term for splitting up a large 
problem into smaller parts so that multiple processing units can work on 
it simultaneously. Suppose for example that you have a ridiculously 
large image ($$1,000,000\times{}1,000,000$$ pixels) and you want to 
apply some sort of filter to each of its $$1,000,000,000,000$$ pixels. 
If your filtering operation takes a millisecond per pixel, it would take 
a single processor unit almost $$32$$ years to finish the filter 
operation. Using all cores of a computer (let's say there are $$16$$) 
reduces this to $$2$$ years. Using a supercomputer with $$1,000$$ of 
these computers (*nodes*) strung together reduces it to $$17.5$$ hours. 
Which is clearly a more acceptable time to wait.

In the case of the simple image filter (a clearly artificial case 
anyway) splitting up the image in $$1,000$$ pieces is the only thing 
required to work on it in parallel, and since operations on individual 
pixels are uncorrelated, the way these pieces match together is not 
really important. This changes when e.g. the filter operation does not 
only depend on the value of each pixel, but also on the value of its 
neighbouring pixels. After splitting up the image, it could happen that 
a node that is working on a specific pixel needs a neighbouring pixel 
that is stored on another node. In this case, the pixel information 
needs to be *communicated* from one node to another. This can only 
happen if we know where that pixel is located. Since this clearly also 
introduces an *overhead* (we need to do a communication that was not 
present in the original single threaded version of the filter), we 
clearly want to minimize the number of times this happens. In this case, 
the domain decomposition does not only need to split up the image, it 
also needs to (a) keep track of where the different pieces are relative 
w.r.t. each other, and (b) try to minimize the communication load caused 
by the decomposition.

Space-filling curves are a good way to decompose an image (or any other 
spatial domain) in an ordered way. They do however not address the 
second requirement for a good domain decomposition directly: they do not 
automatically minimize the number of communications. Despite this, they 
are still very often used for this purpose. The reason for this is that 
a space-filling curve (especially one with a constant separation length 
between successive points on the curve like the Hilbert curve) *tends 
to* (notice the choice of words) minimize the surface to volume ratio of 
the domains. Since the computational cost of a domain *usually* scales 
with the volume of the domain and its communication cost with its 
surface area, this leads to a decomposition with a small communication 
load.

This is however only true if the volume and surface area can be 
truthfully linked to respectively the computational cost and the 
communication cost, and if the factors linking both are fairly similar. 
Especially this last requirement is generally not satisfied: there is no 
reason why a domain that has a low volume to computation factor would 
also have a small surface area to communication factor or vice versa.

In other words, using a space-filling curve for your domain 
decomposition might provide very satisfactory results, but since it does 
not address the actual requirements for a true domain decomposition, it 
can also go terribly wrong. So it might be better to use a domain 
decomposition strategy that actually constructs a real domain 
decomposition.

# Graphs

Before I can show you how to do a real domain decomposition, I have to 
introduce some mathematical concepts. This will allow us to translate 
the domain decomposition problem into a mathematical question to which 
we can apply mathematical theorems in order to solve it.

The image pixels in our example above each individually make up a 
computational unit, to which we can assign a number: the time it will 
take to apply the filter operation to that specific pixel. In our 
example, these numbers will be the same for each pixel, but this does 
not necessarily need to be the case. Each pixel is linked to a number of 
neighbouring pixels (generally 4 or 8 - depending on whether or not we 
include the corners - but less for pixels near the border). If one of 
these links were to be broken (i.e. by putting the two pixels on 
separate node), this will lead to a communication. The corresponding 
communication overhead can also be measured (e.g. in time) and can be 
assigned as a number to the *link* that links both pixels.

We can now construct a diagram for the image in which each pixel 
constitutes a *vertex*, and its corresponding computational cost is its 
*vertex weight*. Neighbouring vertices are linked together with *edges* 
and the corresponding communication costs constitute an *edge weight*. 
The diagram itself is what we call a *graph*. It is a rigorously defined 
mathematical concept, with a lot of interesting properties (I'm sure) 
that I will mostly ignore.

For our image, the corresponding graph is huge (it would actually take 
more memory to store it than it would take to store the actual image) 
and very structured. But graphs can be very complex and unstructured as 
well. Note also that in practice, we would never construct a graph for 
the whole image; the domain decomposition method below works just as 
well if we were to split up the image in a large enough number of small 
blocks (say $$1,000\times{}1,000$$ blocks of the same pixel size) and 
construct the graph for these blocks.
