---
layout: post
title: "Testing numerical models"
description: >-
  Some thoughts on how to convince oneself that numerical models work as they
  should.
date: 2019-05-18
author: Bert Vandenbroucke
tags: 
  - Scientific computing
---

I have spent the whole week working on 
[swift](https://github.com/SWIFTSIM/swiftsim), and this week's post is a 
result of my experiences this week. You might notice that these 
experiences weren't all too positive. But I think they might raise some 
useful questions about the models that are currently used in 
astrophysics.

Numerical hydrodynamics is a field within theoretical physics that is 
concerned with modelling the dynamics of fluids using numerical 
simulations. *Fluids* in this sense can be taken as a wide range of 
physical fluids, ranging from actual fluids like water over gases to 
things that are not necessarily real fluids (like planets). Different 
fluids generally require a very different treatment, but in all cases 
the dynamical evolution of the fluid is captured by a set of physical 
equations that are then translated into a computer algorithm.

Computer algorithms necessarily only consist of basic mathematical 
operations (addition/subtraction, multiplication and division, plus some 
basic logical operators and the ability to make yes/no decisions) and 
hence do not solve the actual physical equations, assuming that these 
equations themselves are accurate enough to describe the real fluid and 
not some idealised version of it. As a result, numerical simulations of 
hydrodynamics will always only yield an approximate solution for any 
given scenario; the accuracy of the solution will depend on a lot of 
factors. One of these factors is the numerical *resolution*, i.e. how 
much computer resources we allocate to model the fluid. In general, more 
resources (a higher resolution) will lead to a more accurate solution, 
this is called *convergence*. Assuming that the computer algorithm 
accurately captures the underlying physics, it should converge to the 
right result, and this will yield some confidence in the solution we 
find.

Convergence is hence a very powerful way of checking if we can trust the 
results of a numerical simulation. If a simulation result changes 
significantly when we increase the resolution of a simulation, the 
result was not converged and we need to keep increasing the resolution 
until the result does not change noticeably any more. The required 
resolution will also depend on what part of the result we are interested 
in: some large-scale statistical properties of the simulation might 
converge at reasonably low resolution, while specific features within 
the fluid might require a much higher resolution. Any numerical 
simulation that is conducted to prove a scientific point should be able 
to show convergence for the quantities that are used to draw scientific 
conclusions. If not, there is no reason to believe that the simulations 
capture the actual physics and the results are physically valid.

But even when convergence is shown, there is no guarantee that the 
simulation actually captures real physics. Some methods might converge 
to the wrong result, because they do not accurately capture the 
underlying physics. To test for this, we can do two things:
 1. We can compare numerical results obtained with our method against 
known theoretical answers. Some physical scenarios can be modelled 
directly in terms of mathematical equations and provide us with an 
*analytic solution* for a specific scenario. If our method works, it 
should be able to converge to this analytic solution.
 2. We can compare different methods against each other. This is called 
*benchmarking* and is especially useful for scenarios where an analytic 
solution is not available. Benchmarking however does not give the same 
level of confidence as having a true test case; it only exposes 
differences between different methods and does not show which method is 
correct. If possible, benchmarking should try to explain the differences 
between different methods, and should aid the development of better 
tests with actual analytic solutions that can provide more confidence in 
a method.

Within numerical astrophysics, a whole battery of standard test cases 
and benchmarking tests has been developed, and these have exposed 
problems with basically all methods that are used in the field (I am 
specifically targeting the field of large-scale cosmological 
simulations and simulations of galaxy formation, but I imagine a similar 
story for other fields within astrophysics). Some methods have problems 
within regimes with large velocities that stem from the way these 
methods work. Other methods can handle large velocities better, but can 
be shown to converge to the wrong result for specific scenarios. No 
method is hence absolutely accurate, and which method you want to use 
depends a lot on the scenario you want to study.

And that's where I am a bit concerned. It has become common practice to 
present numerical algorithms in dedicated *code papers* and then show 
how these methods perform on the various test cases that have been used 
historically. As it should, all methods that are used nicely reproduce 
most test cases at sufficiently high resolution. However, many methods 
require special switches and equations to deal with extreme scenarios 
that are present in some of the tests: these switches activate special 
equations to deal with fluids that are extremely cold and moving at high 
velocities, or artificially introduce physical behaviour that is not 
captured accurately by the method. All of these usually have some free 
parameters that need to be tuned for optimal behaviour, and it is often 
not clear what values are used for a specific test case. And it is also 
not clear if the same values are used for all test cases, or if the 
parameters are tuned to reproduce each test case as well as possible.

If we want to make sure we can trust our numerical results, we should 
fix the free parameters for switches and artificial equations, and as 
such make them an integral part of *the method*. The same method with 
other values for the parameters is not really the same method, and we 
should not treat it that way. There is merit in tuning the parameters 
for a specific test and showing that some variant of the method can 
reproduce that test very accurately, but it is even more important to 
show that the method that is actually used for scientific problems can 
handle the tests, or at least does not too bad. We hence need to come up 
with a good sample of test cases and somehow optimise the parameters for 
the general method for the whole set of tests. The method with those 
optimal parameters should then be promoted to the status of *the 
method*, and we should show how it behaves on all test cases, 
potentially in comparison with the variant of the method that was tuned 
for each specific test.

We should also do the same with regard to resolution. Large-scale 
simulations are very expensive to run, and will never be converged 
within an absolute sense; an increase in resolution will always resolve 
more small-scale structures that were not even present in a lower 
resolution run. That is fine, since we are usually only interested in 
statistical properties of the whole simulation that may well converge 
earlier. But we should at least make sure that these properties can be 
trusted. So instead of testing the methods against test cases and 
showing that we can reach a converged result, we should show that we can 
get an accurate result at the resolution that we effectively reach in 
the large-scale simulation. Only then can we be sure that our method is 
actually doing physics, and not something that could be physics, but is 
actually only a blurry version of it.

So in conclusion, I think that we could somewhat improve the way we 
define methods, and more clearly distinguish between general methods 
that have a lot of tunable parameters, and actual methods that use one 
specific set of these parameters. I feel like many methods that are very 
widely used in astrophysics only pass the relevant test cases as a 
general method. I have very little proof that the actual methods that 
are used pass the same tests, let alone at the resolution at which they 
are used in practice. And I really miss a rigorous treatment of 
parameter tuning in any code paper I have seen: people usually are very 
hand-wavy about the values they use and base these values on personal 
experience rather than actual quantitative metrics.
