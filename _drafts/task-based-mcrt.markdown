---
layout: post
title: "Task based MCRT"
description: A brief description of my new task based MCRT method.
#date: 2019-04-18
author: Bert Vandenbroucke
tags: 
  - Code Development
  - Scientific Computing
---

In previous posts, I introduced the topics of [task based 
parallelisation]({% post_url 2019-04-07-task-based-parallelisation %}) 
and [Monte Carlo radiation transfer (MCRT)]({% post_url 
2019-04-13-monte-carlo-radiation-transfer %}). The topic of this week's 
post will be my attempts at combining both. This algorithm forms the 
backbone of version 2.0 of 
[CMacIonize](https://github.com/bwvdnbro/CMacionize) and will hopefully 
be the subject of a code paper later this year.

# MCRT tasks

To create a task based algorithm, we require *tasks*: small units of 
work that are only a small fraction of the total computational work and 
that use only a small amount of the computational resources: a single 
computational core and as little memory as possible. Since the MCRT 
algorithm requires a grid, it makes sense to base the MCRT tasks on a 
subdivision of this grid.

The grid is however only one component of the MCRT algorithm: we also 
have to think about the photon packets that are propagated through this 
grid. Photon packets are generated at a source, then travel through the 
grid for an unknown distance and in a random direction, before 
potentially scattering in a different random direction, leaving the box 
that encompasses the grid, or being absorbed. How do we fit these photon 
packets into the task based framework?

The solution is to again think about the grid. Photon packets only 
interact with the grid during one step of the MCRT algorithm: when they 
are being propagated for an unknown distance in a random direction. 
Generating the photon packets at the source does not require any 
knowledge about the grid. Scattering the photon packets (this can be 
because of direct scattering but also because of reemission at a 
different wave length) requires some knowledge about the grid, but does 
not generally require *access* to the grid, i.e. it does not cause 
potential thread conflicts when done in parallel. So the only step of 
the MCRT algorithm that really requires unique access to a (part) of the 
grid is the propagation step.

When photon packets are propagated through a small part of the grid, 
they will likely leave the box encompassing that part before being 
absorbed, i.e. we cannot reasonably expect the photon propagation to be 
over once it leaves a single grid part. This is however not an issue, as 
all the information required to continue propagating a photon packet can 
be stored in a few variables: the current *position* of the photon 
packet, the current *direction* it is moving in, its current *energy* 
and the current *optical depth* it has accumulated while travelling 
through the grid. If we store these quantities in memory and read them 
back later, we can continue the photon propagation step in another part 
of the grid as if it never stopped. We can hence do the propagation step 
for a single part of the grid and on exit store the photon packet and 
continue its path later.

Storing single photon packets is however not very efficient, as there 
are typically many millions of them during a single step, and many of 
them access the same parts of the grid. It therefore makes sense to 
process photon packets in batches and store all packets in a batch 
within dedicated *buffers*. These buffers can be made to fit the memory 
requirements for a typical task by making them small enough, and can be 
handled in the same way as grid parts to make access to them thread 
safe.

With the introduction of photon packet buffers, the tasks that make up 
the task based MCRT algorithm can be defined as follows:
 - photon packet generation: an empty photon packet buffer is filled 
with new randomly generated photon packets at a source
 - photon packet traversal: the photon packets in a photon packet buffer 
are propagated through a part of the grid. On entry, all these photon 
packets are either within that part of the grid or entering it through 
one of its sides. On exit, the photon packets are either absorbed (and 
potentially scattered/reemitted) within that part of the grid or leave 
the grid part through one of its sides, edges or corners (there are 4 
sides and 4 corners for a 2D square cell and 6 sides, 12 edges and 8 
corners for a 3D cubic cell). There are hence multiple possible output 
buffers for a single input buffer.
 - photon packet scattering: a photon packet that was absorbed is 
scattered or reemitted in a different direction and with potentially 
different properties.

Below I will discuss the necessary components to turn these tasks into a 
working task based MCRT algorithm.

# The distributed grid

The first important component in the task based MCRT algorithm is the 
grid that is split up into many smaller parts. I will refer to these 
parts as *subgrids*. Each subgrid has a number of neighbouring subgrids 
with which it shares a face, edge or corner, and to be able to 
efficiently process photon packets, this neighbour information needs to 
be stored. Each subgrid also requires a *lock*, i.e. some way of 
preventing multiple threads to access the subgrid at once.

Subgrids should as much as possible be self-contained, i.e. they should 
store all the geometrical information required to compute positions and 
volumes within the subgrid. They can however store this information in a 
compressed way, e.g. for a regular grid there is no need to store the 
volumes and coordinates of all cells; you only need to know the 
dimensions of the subgrid box and the number of cells within the 
subgrid.

Due to the strong non-local character of Monte Carlo radiation transfer, 
some parts of the grid (i.e. subgrids) are much more likely to be 
visited by photon packets than others, especially if the number of 
sources is low. This means that the locking mechanism that protects 
individual subgrids can potentially cause serious performance 
bottlenecks. To address this issue, it is possible to make *subgrid 
copies*: a subgrid that is visited a lot more than average is duplicated 
and the neighbouring relations for the neighbouring subgrids are 
adjusted so that half of the neighbours links to the original subgrid, 
and the other half to the copy. From the point of view of the individual 
subgrids, the algorithm can still proceed in the same way. At the end of 
the photon propagation step for all photon packets we can simply 
synchronise the optical properties between the copy and the original. 
This procedure can be repeated to generate more copies; I opted to do 
this using a power of two copy hierarchy, as shown below. In this 
hierarchy, each subgrid gets assigned a *copy level* $$C$$, and the 
number of copies for that subgrid is then given by $$2^C$$. Since all 
subgrids are small in memory, the duplication and synchronisation steps 
are very cheap and cause only a small overhead.

![Power of two copy hierarchy for a number of 
subgrids](/assets/images/copy_hierarchy.png)

To determine optimal values for the number of subgrid copies, there are 
a number of options. One option is to keep track of the number of visits 
to each subgrid during a previous step of the MCRT algorithm and to use 
that value to refine the copy hierarchy dynamically. Although this is 
arguably the most correct method, it is quite tricky to implement and 
has a significant overhead. Furthermore, the load per subgrid tends to 
change between steps, so that this measurement is not necessarily 
representative enough. I therefore opted for more simple methods.

The least simple method consists of running a very low resolution MCRT 
simulation with little photon packets before the start of the actual 
simulation. Each subgrid in the actual simulation makes up a single cell 
in this low resolution simulation, and the optical properties of this 
cell are used to determine a good load for the corresponding subgrid. 
Despite its simplicity, this method performs surprisingly well, and is 
also the best method I have so far to perform the distributed memory 
load balancing for the task based MCRT algorithm. However, some overhead 
associated with the additional bootstrap simulation is clearly present, 
not in the least in the amount of additional code that needs to be 
written and checked. And I didn't test this method in enough different 
scenarios to ensure that it always leads to a good load balance.

This is why I opted for an even simpler method for CMacIonize 2.0: a 
copy hierarchy based on the source positions. The main idea behind this 
is that subgrids with a high load are almost exclusively subgrids that 
either contain a source, or have a neighbour that does. We can flag the 
subgrids that contain a source and assign an arbitrary copy level to 
these (I currently use $$C=4$$, which correspond to $$2^4=16$$ copies). 
Next, we can ensure that subgrids that have a neighbour with a high copy 
level get assigned a high copy level as well, by e.g. limiting the copy 
level difference between neighbouring subgrids to 1. All of this can be 
done once, at the start of the simulation, and imposes very little 
overhead. Nevertheless, I found that it works very well.

A last subtlety about the use of subgrid copies is the treatment of 
source photons. Subgrids with a high copy level are almost exclusively 
subgrids that contain a source, and these subgrids get most of the 
photon packet load directly from photon packets originating at that 
source. To make sure the copies get a fair share of this load, we need 
to keep track of the number of copies of each subgrid that contains a 
source and distribute the photon packets from that source evenly among 
these copies. This requires some bookkeeping, but can be done without 
too much overhead.

# Thread safe memory spaces

The task based MCRT algorithm introduced here has one big disadvantage 
compared to traditional task based algorithms like the one in 
[SWIFT](https://gitlab.cosma.dur.ac.uk/swift/swiftsim): the generation 
of additional tasks. Since we have no idea what subgrids will be visited 
by any given photon packet, we cannot a priori predict how many 
propagation tasks are required for each subgrid or what the task 
dependencies for these will be. The only way to get around this is by 
creating the propagation tasks when necessary, i.e. a task that is 
already running can spawn new tasks.

In itself this is not a problem for the execution of the tasks, as the 
task based locking mechanisms prevent unspawned tasks from being 
executed. It does however require us to implement an additional stop 
condition for the photon propagation step, as an empty queue for all 
threads does not necessarily mean the step is done. And it creates a 
significant issue with memory management for the resources that are used 
by the tasks: the tasks themselves and the photon packet buffers.
