---
layout: post
title: "Monte Carlo radiation transfer"
description: An introduction to Monte Carlo radiation transfer algorithms.
#date: 2019-04-07
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
  - Code Development
---

Since I am currently working on a task-based Monte Carlo radiation 
transfer algorithm, I thought it would be nice to dedicate a post to 
this subject. However, until now I have only covered the topic of 
task-based parallelisation, and I have not yet explained what Monte 
Carlo radiation transfer is. Hence the topic of this week's post.

# Radiation transfer

*Radiation transfer* is a broad topic that can be classified as 
theoretical astrophysics but is also used in many other fields, ranging 
from medical science (the interaction between radiation and skin for - 
among others - medical imaging) to computer graphics (rendering 
realistic scenes in a videogame for example). The principle aim of 
radiation transfer is to describe the interaction between light that 
originates at one or multiple, possible extended *sources* with the 
*medium* it travels through. This is done to either figure out what the 
properties are of the light that arrives at a specific location (e.g. 
the camera in a video game or the telescope that observes some 
astrophysical phenomenon), or to find out what the light does with the 
medium (e.g. destroying cancer cells embedded in the skin or heating up 
interstellar gas). Or to do both simultaneously (e.g. when the heated 
interstellar gas emits additional radiation that also reaches the 
telescope).

Whatever its aim might be, radiation transfer is never easy. The reason 
for this is that radiation, unlike most physical processes we know, can 
travel very far away from the source before it interacts with the 
medium. On top of that, the radiation changes while it travels from the 
source to a specific location in the medium: a fraction of it is 
absorbed on the way, another fraction is scattered away in another 
direction, while yet another fraction is scattered into this specific 
direction and mixes up with the original radiation. What eventually ends 
up at the location we are interested in hence does not only depend on 
the local properties of the medium, the properties of the source and the 
distance between the source and that location, but also on the 
properties of the medium in between the source and the location, and 
even the properties of the medium that is not directly in between. In 
scientific terms, we call radiation transfer a highly *non-local* 
process.

The situation is not as bad in a very *dense* medium, when radiation is 
absorbed and scattered on very small scales compared to the size scales 
that govern the dynamics of the system we want to study, e.g. in the 
atmosphere of our Sun. In this so called *optically thick* case, the 
radiation transfer can usually be approximated using some kind of heat 
transfer equation without actually worrying about radiation. However, 
the transition from this optically thick to the *optically thin* regime 
(where radiation travels far before interacting) can be very sudden: 
radiation that reaches the top of the Solar atmosphere can suddenly 
escape and travel all the way to Earth before its next interaction. For 
general problems, it is therefore important to be able to handle both 
the optically thick and thin regimes simultaneously.

To tackle the general problem of radiation transfer, scientists (and 
computer scientists) have devised many strategies over the years. These 
range from trying to capture the radiation physics in sets of 
complicated equations, to probing the interaction between the source(s) 
and the medium by casting a large number of rays on a GPU (as done in 
computer games). Monte Carlo radiation transfer can be seen as a very 
powerful hybrid method that uses elements of both these approaches, and 
that works best in 3D cases.

# Monte Carlo radiation transfer

While using physically motivated sets of equations to solve the 
radiation transfer problem is arguably the most scientific way to track 
this problem, it is generally impossible to do this in 3D without making 
serious limiting assumptions. This is simply because the necessary 
computations are so involved that it becomes impossible to solve them in 
any reasonable time. When we accept that a general solution is not 
feasible, then the next best thing we can hope for is a *sampling* of 
the general solution, i.e. a solution that is not as exact as the 
general solution, but that nonetheless provides a good representation of 
the real solution. This is what ray tracing hopes to achieve: by 
sampling the radiation as a finite number of rays that travel in various 
directions, we hope to capture the dominant processes in the radiation 
transfer without going to the trouble of sampling all possible 
directions (which is not possible).

The problem with a ray tracing approach is the choice of sampling 
directions. While a uniform sampling of possible directions seems like a 
good choice, there is no guarantee that this will actually work: if the 
source is located close to a dense gas cloud in a further almost empty 
space, then we clearly want more rays to be cast in the direction of the 
dense cloud than in the other (uninteresting) directions. If we know 
that this is the particular setup, we can of course take this 
information into account when choosing sampling directions, but 
generally this information might not be available. And what about the 
situation where there is a large number of dense clouds, scattered 
randomly around the star? A regular sampling of the rays might 
systematically miss some of these, and yield a completely skewed image 
of the system...

Monte Carlo does not really solve these issues, but instead overcomes 
them by taking away the choice of sampling. Instead of casting rays in 
regular, chosen directions, a Monte Carlo radiation transfer technique 
casts rays in a uniform *random* direction. Random in this case means 
that we choose the directions for the rays using some process that has 
nothing to do with the physics of the radiation transfer, e.g. a random 
number generator on our computer. Uniform means that while individual 
directions are completely random (and *uncorrelated* to each other and 
to the physics of the problem), the statistical distribution of all 
random rays will still cover all possible directions uniformally, i.e. 
every direction is equally likely to be chosen.

For small numbers of rays, this random generation of ray directions is 
obviously a worse approximation than using a regular set of rays. The 
power of the technique is in the numbers: as the number of rays 
increases, the distribution of the random rays will become more uniform. 
And while the traditional ray tracing method always uses the same 
directions, the Monte Carlo rays are always different, so that they will 
pick up all features, however small, provided that enough rays are cast.

The use of random numbers is not only limited to the directions of rays, 
but can also be used to sample other aspects of the radiation transfer 
problem. A star does not just emit one kind of radiation, but emits a 
whole spectrum of radiation at different energies. Radiation with 
different energies interacts entirely differently with the interstellar 
medium, so that a general treatment of the interstellar radiation 
transfer problem would require not only a sampling in directions, but 
also a sampling in energies. Which again implies a choice of energies 
that can skew the results. Monte Carlo radiation transfer again replaces 
these choices of how to sample the spectrum of the star by a random 
sampling of the spectrum, which does a better job if enough rays are 
cast.
