---
layout: post
title: "Monte Carlo radiation transfer"
description: An introduction to Monte Carlo radiation transfer algorithms.
date: 2019-04-13
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
realistic scenes in a video game for example). The principle aim of 
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
random rays will still cover all possible directions uniformly, i.e. 
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

A similar story can be told about the direction of radiation. While a 
star generally emits the same amount of radiation in all directions 
(*isotropic* emission), there are plenty of sources that have very 
specific directional emission (e.g. a spotlight). This directional 
dependence can be modelled using a distribution function which then 
needs to be sampled properly. Again, Monte Carlo techniques can do this 
very easily.

So in general, Monte Carlo radiation transfer can be seen as a brute 
force technique to account for a whole wealth of additional physical 
processes by just translating them into appropriate probability 
distributions for the randomly emitted, scattered and absorbed 
radiation. For simple problems, its results will always be less accurate 
than those of more accurate methods that actually properly solve the 
equations that are involved. But for more complicated systems, Monte 
Carlo will still be as accurate (or approximate) as for these simple 
systems, while more accurate methods will suffer from the problems set 
out above or will become simply intractable. And to improve the 
accuracy of the Monte Carlo sampling, we can simply increase the number 
of random rays that is used.

# How does it work?

The precise workings of a specific Monte Carlo radiation transfer 
technique depend on what kind of radiation transfer we are considering, 
i.e. whether we are interested in the properties of the radiation, the 
properties of the medium being irradiated, or both. Nevertheless, some 
general aspects can be discussed.

Firstly, there is the way Monte Carlo techniques treat the radiation and 
the medium respectively. For the former, the *rays* introduced above are 
replaced by a more general *photon packet*, which for the purposes of 
the radiation transfer is treated as a single physical photon with well 
determined properties, but in fact represents a statistical sample of 
physical photons that is much smaller than the actual number of photons 
involved. The latter is represented on some kind of *grid* structure 
that consists of cells with a well defined geometrical volume and edges. 
Within these cells, the optical properties of the medium are assumed to 
be constant, similar to a single pixel in an image.

The Monte Carlo technique then consists of a number of distinct steps:
 1. Photon packet generation: this is the process during which a random 
photon packet is generated at the position of a source (or at a position 
that is randomly sampled from a source distribution). The photon packet 
receives a random direction and properties that are sampled from 
corresponding probability distributions.
 2. Photon packet propagation: the photon packet is passed on to the 
cell in the grid that contains it, or the first cell in the grid that it 
encounters on its path. By tracing the path through the geometrical 
volume of the cell, we can compute the distance the photon packet 
travels through that cell. By multiplying that distance with the optical 
properties of the photon packet and the medium inside the cell (remember 
that we assume constant properties inside the cell) we can decide 
whether the photon packet is absorbed within the cell or travels to the 
next cell. We can also use this information to update the optical 
properties of both the photon packet and the medium in the cell.
 3. Photon packet scattering: the photon packet is scattered within the 
cell. Based on the optical properties of the cell in which the 
scattering takes place, the photon packet will receive a new random 
direction. Depending on the interaction physics, absorbed photon packets 
can also be re-emitted with a new direction and new optical properties; 
although physically different, the Monte Carlo algorithm treats this 
process in the same way as a scattering.
 4. Photon packet recording: the photon packet (or a weighted copy of 
it) is recorded onto a model image. This step is only required if the 
radiation transfer is used to produce model images that can be compared 
with real observations.

While all of this might sound very complicated, it is relatively 
straightforward to implement these various steps and they can be 
executed very efficiently by modern computers. The full Monte Carlo 
algorithm is then obtained by repeating these steps for a (very) large 
number of photon packets to obtain a good statistical sampling of the 
radiation and its interaction with the medium.

In cases where the optical properties of the medium are the main target 
of the radiation transfer, the photon packet propagation is only one 
step in the algorithm that is followed by a step in which the optical 
properties of the medium are recomputed based on the radiation field. In 
many cases, the dynamic equilibrium between the medium and the radiation 
field can only be found through an iterative procedure in which these 
two steps are applied repeatedly until a sufficiently converged state is 
obtained.

# What about efficiency?

Some people call Monte Carlo methods *ridiculously parallel*, as it is 
relatively straightforward to obtain very significant parallel speedups 
for any Monte Carlo algorithm by processing individual photon packets in 
parallel. The main reason for this is that - by construction - each 
photon packet is independent of each other photon packet, so that they 
can be processed simultaneously without any obvious conflicts.

This is however not entirely accurate. While individual photon packets 
are uncorrelated and conflict-free, the same is not true for the medium 
with which the radiation interacts. Different photon packets can enter 
the same cell simultaneously and this can still lead to potential 
conflicts during the update of the optical properties of the cell. 
Resolving these conflicts can hamper the parallel efficiency, especially 
in small grids with a high number of parallel threads.

For small grids, distributed memory parallelisation is incredibly 
straightforward and the speedup can be near ideal. The only 
communication that is required is a sync of all cell properties after 
every photon packet propagation step, which can be done with very little 
MPI calls and introduces almost no overhead. This scenario however 
assumes that each MPI process stores a full copy of the grid, which is 
only possible for large memory nodes and for relatively small grids.

As soon as grids get really large, Monte Carlo techniques usually hit a 
serious bottleneck in terms of scalability. When the grid needs to be 
split over multiple MPI processes, additional communication is required 
to send photon packets from one part of the grid to another. Since these 
communications follow the random paths of the photon packets, they are 
unpredictable and therefore extremely hard to manage. There are a few 
algorithms in the relevant scientific literature that overcome this 
issue, but none of them is really straightforward to implement.

So overall, Monte Carlo techniques are very efficient for small grids. 
They get increasingly hard to parallelise for larger grids and there is 
some size scale limit above which they become impractical and very hard 
to use. Even if a large grid fits in memory, its size combined with the 
random access patterns that are the natural result of the Monte Carlo 
nature of the technique lead to very inefficient memory usage, which 
hampers the overall efficiency. This is why I will discuss an 
alternative approach to Monte Carlo techniques in an upcoming post.
