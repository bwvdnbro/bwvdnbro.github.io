---
layout: post
title: "Some general programming experiences"
description: A general rant about things I experienced while programming.
date: 2019-12-23
author: Bert Vandenbroucke
tags:
  - Code development
---

After 51 weeks of writing posts, I'm pretty much out of good topics to 
write about, so I thought I would finish my year of posts with some 
general remarks about things I haven't really talked about (too) much 
yet. There is no real general topic here, and some of it might be quite 
random.

# Avoid unnecessary function inversions

Recently, I came across a numerical maths problem that gave me a lot of 
issues, until I suddenly realised things were a lot easier than I 
thought. The realisation was basically that computers *always* deal with 
discrete representations of mathematical functions (say, $$f(x)$$), and 
that it does not matter if you take a discrete range of $$x$$ values and 
then compute $$f(x)$$ from that, or if you sample discrete values of 
$$y=f(x)$$ and then compute $$x=f^{-1}(y)$$.

Why does this matter? In the problem I encountered, I had a distribution 
function for a quantity called $$L$$, a *shape factor* for a geometrical 
object called a *spheroid* (none of this is really important). The 
distribution function, $$G(L)$$, was given by

$$
G(L) = 12 L (1-L)^2
$$

and defined for a clearly set out range of $$L$$ values. The $$L$$ 
factor itself is related to the *axis ratio*, $$d$$, of the spheroid by

$$
L = \frac{1-e^2}{e^2} \left[
    \frac{1}{2e} \ln\left( \frac{1+e}{1-e}\right) -1
  \right],
$$

$$
e^2 = 1-d^2
$$

for $$1>d$$, and a similar but annoyingly different expression for 
$$d>1$$.

What I needed to get was the distribution function for the axis ratio 
$$d$$, given the distribution function for the shape factor $$L$$ and 
the various expressions that relate $$d$$ and $$L$$. My first naive 
approach was to set up a discrete range in $$L$$ values and then 
numerically invert the $$L(d)$$ relations to get a corresponding value 
for $$d$$. Unfortunately, this inversion turned out to be pretty 
difficult. Furthermore, it turned out to be difficult to figure out when 
to use which expression for $$d^{-1}(L)$$ given a value of $$L$$, since 
the distinction $$d>1$$ vs $$1>d$$ is not as clear in $$L$$ space.

After struggling with this for longer than I would want to admit, I 
realised it is actually a lot easier to set up a range of axis ratios 
$$d$$, use those to compute a range of $$L$$ values (no numerical 
inversion required) and then compute the distribution function for 
those. Added advantage: to convert the distribution function from $$L$$ 
space to $$d$$ space, it also needs to be multiplied with the *Jacobian* 
of that coordinate transformation, $$\frac{dL}{dd}$$, which is again 
much easier to compute as a function of $$d$$.

So in general: if you need to compute a mathematical function 
numerically, it is well worth considering the difference between a 
forward computation of $$f(x)$$, or an inverse computation 
$$x=f^{-1}(y)$$, depending on which one of those is easiest. For most 
practical purposes, both will lead to the same result, e.g. for 
visualisation or as part of a larger calculation. The only reason to 
actually stick to a forward calculation is if your input values are 
really $$x$$ and you need to compute $$y=f(x)$$ or if you need to have 
control over the sampling in $$x$$.

# Visualisation is key

This might just be because of the way I think and work, but I find it 
incredibly helpful to *see* things. *Things* in this context is pretty 
much anything: mathematical functions $$f(x)$$ like the ones above, grid 
structures like the ones used in hydrodynamical integration codes or 
Monte Carlo radiation transfer codes, geometrical shapes like spheroids, 
and even flow charts and task plots that show the structure and progress 
of a parallel algorithm.

The reason for this is simple: all of these things are to a certain 
extent very abstract, and visualising them helps you understand them. 
Mathematical expressions like the $$L(d)$$ relation above are not 
incredibly hard, but it is still worth having a look at their graph to 
see what shape they have and how they behave in the domain where they 
are defined. If you do that, you see that $$L(d)$$ has some weird 
asymptotic behaviour for $$d\rightarrow{}1$$, which explains why it so 
hard to invert this function. The same goes for functions that you want 
to integrate numerically: if the function is badly behaved, visualising 
it can show you why you have trouble computing its integral for some 
values. And by comparing the surface area under the curve you see 
visually with the result you get, you can check whether your integration 
routine returns a sensible result.

For algorithms (especially parallel ones) flow charts offer you an 
easier way to check the large structure of your program, and help spot 
missing dependencies or obvious bottlenecks. Task plots are very good at 
exposing load imbalances and will give you a very good idea of where 
most of your program time is spent. This is crucial information to 
decide where optimisation would be most beneficial.

The weird effect of this importance of visualisation is that I spend 
quite a lot of time on writing visualisation code: Python scripts that 
make diagrams or function graphs, but also specific code to output the 
necessary information from a code to generate flow charts and task 
plots. It also means I spent a lot of time already figuring out how 
various kinds of visualisation tools work, as obvious from the large 
fraction of past posts dealing with those topics. This is a lot of 
overhead, but as in a lot of cases, I think it is totally worth it.

# First the test

As important as good visualisations are good tests. I have stated before 
that a code that has not been tested should be assumed to not work. This 
also means that if you do not have a test for a piece of code that you 
are planning to write, there is no point in writing that code. So before 
you start writing any piece of code, you should ask yourself two 
questions:
 1. Does a test for what I am about to write exist?
 2. If not, can I come up with a good test myself?

If the answer to both of these questions is negative, then you should 
probably not write the piece of code, as you will not be able to trust 
any of the results that come out of it.

Once you have a good test, it is also a good idea to already think about 
visualisation. What plot would convince you that your new code works? 
How do you make that plot? What data do you need to output to make the 
plot? This is not only practical towards the test itself, but can help 
you think about a good structure of your code. And if you do this part 
right, you will end up with a bunch of plotting scripts that can 
probably be helpful in the future too.

# A healthy dose of curiosity

When I was still young and a PhD student, I spent an awful lot of time 
nosing around on the internet, just to figure out how things work. And I 
have come to realise in recent years that this has had some long term 
benefits.

To put this into context: when I was a PhD student, we had a nice 
tradition called *pizza Friday*: one of the Italian postdocs in the 
group would go to the local Italian approved pizzeria and would get 
pizza for lunch. After lunch, I would be pretty full, so Friday 
afternoon tended to be a very lazy time of the week. Then I would just 
start thinking about computers and software and things I did not know, 
like "can I run a CUDA computation on the GPU in my desktop?" I would 
then spent the next hour just googling CUDA related stuff and try to run 
some small tutorials to convince myself that I could indeed do this.

By the end of that Friday, I would have a basic understanding of the 
topic (CUDA in this case), and I would have a small test program that I 
would then start restructuring into a somewhat satisfactory 
object-oriented structure. I would never actually use this for anything 
useful, but it would give me a sense of accomplishment and would help me 
understand something about that topic.

I didn't do this every Friday, but I definitely did it a few times. And 
sometimes some of these topics would turn out to be useful in the long 
term: I definitely did use things like memory-mapping or Python API 
exposure in real applications, but usually years after I first learned 
about them. On top of that, these little expeditions into the unknown 
helped me to develop my own coding style, as they gave me an opportunity 
to write little programs from scratch very often. And they gave me a lot 
of background knowledge for future projects. Maybe one day I will come 
across a problem where I can actually use GPUs...

So my advice would be to sometimes give in to this kind of curiosity. If 
you have a "free" afternoon - you finished something and you don't 
really feel like starting something new - then just think about small 
and slightly irrelevant things and try to figure out how they work. The 
worst thing that can happen is that you spend an afternoon figuring out 
that particular thing is quite complicated. But you can also end up 
discovering something that is actually useful, or just do something 
small and constructive that makes you feel good about yourself.

# Write about it

This last item is pretty much the whole philosophy of academia. If you 
spent a lot of time developing something new and possibly complicated, 
it can be very useful to write or talk about it with other people. 
Presenting scientific work during a meeting or conference helps you put 
it into context: it gives you a better idea of where your work fits into 
the bigger picture. Writing or talking about complicated code concepts 
helps you structure them and disentangle the essential and non-essential 
aspects. No better way to really understand a topic than by lecturing 
about that topic. If a horde of knowledge-thirsty students will pick 
your brain after your lecture (that's what you imagine - they never 
actually do), you better understand the thing yourself.

So if you feel you are getting stuck designing a particularly 
challenging bit of code, it can really help to put it into words 
somehow. You can find a colleague and try to explain your problems to 
her, or you can open up an empty text document and just start typing. 
More often than not will you find yourself solving half of your issues 
in the process, by spotting little inconsistencies or things you 
overlooked. And even if all the things you write do make sense, it is 
still useful to have them written down or structured in your head in 
such a way that you can more easily present them later on.

This strategy is also useful when thinking about visualisation: try to 
imagine that the plot you are about to make has to convince a crowd of 
people that the thing you just did actually makes a difference. How do 
you convey this message most clearly?
