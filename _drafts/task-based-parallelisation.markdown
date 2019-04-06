---
layout: post
title: "Task based parallelisation"
description: A very brief introduction to task based parallelisation.
#date: 2019-04-05
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
  - Code Development
---

This week's post will need to be quite short, as I am currently on a 
conference in Finland and have very little time to spare. Additionally, 
[DiRAC](https://dirac.ac.uk/) last week approved our short project worth 
half a million CPU hours (and starting... this week), so that I am very 
busy trying to get this project running as quickly as possible.

So this seems like a good opportunity to start introducing one of the 
important concepts that enabled us to get this computation time in the 
first place: *task based parallelisation*. This will eventually allow me 
to explain the inner workings of my task based Monte Carlo radiation 
transfer prototype, 
[ParallelCMacIonize](https://github.com/bwvdnbro/ParallelCMacIonize), 
the precursor to the next version of 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize) that I am currently 
rolling out a bit more quickly than expected.

# Rationale

Computers nowadays are very different from the computers that were used 
at the end of the last century. Since the development of the first 
modern, programmable computer in the 1940s until the early 2000s, 
computers were mainly treated as serial machines: you fed them a list of 
instructions, and the computer would execute that list of instructions 
in a specific, pre-determined order. In principle, you would be able to 
replace any serial computer program with a single human being and it 
would still pretty much work the same (albeit a lot slower and with a 
possible negative impact on the well being of that human being). Note 
also [the historical context behind the name 
*computer*](https://en.wikipedia.org/wiki/Human_computer).

In the early 2000s, CPU manufacturers started producing multicore and 
multi-CPUs systems on an industrial scale, and by 2010 they were pretty 
much the standard in any commercial PC or laptop. Nowadays, even most 
phones have more than one CPU core. This resulted in a radical change in 
how computers work: software that wants to exploit the full power of the 
CPU can no longer consist of a simple list of serial commands, but 
instead needs to use some kind of *parallelisation strategy*. Ignoring 
this leads to a massive waste of computational resources. Furthermore, 
it puts a hard limit on how fast your software works: individual CPU 
cores no longer get any faster, but instead spread out the increasing 
speed of the CPU over more and more cores. Only by exploiting the 
inherent multicore nature of a CPU can you still increase your 
computational efficiency and hence the run time of your calculations.

While all of this was going on, memory issues also became more and more 
apparent. Early computers had a very small amount of RAM memory (which I 
will just refer to as *memory*) and you usually had to think very 
carefully about how to use this efficiently. When memory sizes started 
to go up (in line with, but slower than [Moore's 
law](https://en.wikipedia.org/wiki/Moore%27s_law), memory usage got 
cheaper and software started using the available memory more freely. 
However, increasing memory sizes also led to a more complicated memory 
layout: while all memory should in principle be readily and quickly 
accessible to the CPU (cores), a clear speed difference exists between 
memory that is located close (physically) to the CPU and memory that is 
further away. This difference grows with increasing memory size and 
hence became more apparent. To avoid long waiting times for the CPUs, 
fast memory caches were developed that sit close to the CPU and that can 
be used to temporarily store variables used by the CPU. These caches are 
small and a programmer has very little control over how they are used.

Both these developments lead to very strict requirements for software 
that wants to use modern CPUs efficiently. Efficient algorithms should
 1. be written in a manifestly parallel way, so that they automatically 
avoid common pitfalls of parallel software, like race conditions and 
deadlocks,
 2. only actively use small amounts of memory at any given time and 
preferably reuse as much variables as possible, to improve cache 
efficiency.

While some programmers (most of them scientists) argue that this should 
probably be achieved by either developing better programming languages 
(e.g. [CHARM++](http://charmplusplus.org/)) or by making compilers 
better at doing these things for you, I think the best way to achieve 
this is by changing the way we think about algorithms and software, 
within the limits of existing languages and compilers. Task based 
parallelisation is an example of a strategy that can naturally help to 
improve algorithms towards the aims set out above.

# Tasks

The main idea of task based parallelisation is to split up the 
computational problem at hand into *many* small parts. Small in this 
context has a double meaning:
 1. the parts have to be small in terms of computation and should take 
only a small fraction of the overall computation time required by the 
algorithm,
 2. the parts should be small in terms of memory footprint and should 
preferably fit into a fast memory cache (with a typical size of the 
order of MB).

In addition to this, the tasks should be either *independent* (the task 
can be executed without any interference with any other task) or should 
have clear *dependencies* (i.e. it is clear before the start of the task 
which tasks will have conflicts with this task, so that we can decide 
not to execute the task while these other tasks are being executed).

It's probably easier to explain this using an example. Imagine that you 
have a large two dimensional matrix with numbers, and you want to apply 
two operations to this matrix. The first operation takes a single value 
of the matrix as an input, applies some complicated function to it and 
then replaces the original value with the result. The second operation 
takes two elements of the matrix that are next to each other, takes them 
as input to a complicated function and then adds the result to both 
input values. Since a typical cell in a two dimensional matrix has four 
direct neighbours (top, bottom, left and right), you need to apply the 
second operation four times to each cell (the cells at the border are a 
bit different). Additionally, the high computational cost of the second 
operation means that you only want to do the *interaction* for two 
neighbouring cells once, i.e. you want to update both the left and right 
or top and bottom cell as part of the same operation. Since the second 
operation overwrites the original value and all interactions should be 
based on the old value, we store a copy of the old value (this doubles 
the size in memory of the matrix).

To turn our two operations into a task based algorithm, we first need to 
divide our matrix into a number of smaller parts. This can be done in 
many ways, and I have already introduced some of them BEFORE. A task 
will then constitute one of our operations (or part of it) on a single 
part of the matrix. Provided there are enough small parts, these tasks 
will be small in memory and constitute a small fraction of the total run 
time, as required.

The first operation also satisfies the independence criterion, as the 
operation can be applied to any of the cells in a matrix part completely 
independently of all the other parts. For the second operation, things 
are a little bit more complicated, as this operation has two dependencies:
 1. we can only apply the second operation to a part of the matrix if we
already applied the first operation to it,
 2. while we are applying the second operation to one part of the 
matrix, we cannot apply it to the neighbouring parts of the matrix, as 
this could lead to conflicts (where we try to update the result for a 
single value with two different threads simultaneously).

We however know these dependencies beforehand: each second operation 
task for a part of the matrix depends on:
 1. the first operation for that matrix part being finished,
 2. the second operation for all neighbouring parts (there are generally 
four of those) not being running.

Before we go on, we need to address an additional issue with the second 
operation: the way it handles cells at the border of a matrix part. If 
we were to naively apply the second operation for all cells on each 
border across that border, then all cells at the border would get the 
border contribution twice: once for the left-right/top-bottom 
interaction and once for the right-left/bottom-top interaction. We can 
easily solve this by eliminating two of these interactions, this halves 
the dependencies.

Furthermore, we could split the second operation into three different 
tasks: an interaction operation that only does the cells internal to the 
matrix part, and two versions that only do the cells for a left-right or 
top-bottom interaction between two neighbouring parts. This further 
reduces the number of dependencies for these tasks and leads to smaller 
tasks.

In the end, we end up with the following tasks:
 1. first operation: reads the value of a cell and applies a function to it.
Overwrites the original value of the cell and stores a copy in a second
variable so that the second operation can add to this. Dependencies: a single
part of the matrix.
 2. second operation (internal): interacts all cell pairs of which both 
members are within a matrix part. Takes the original value of both 
cells, feeds these to the interaction function and adds the result to 
the second cell variable. Dependencies: a single part of the matrix for 
which the first operation was already done.
 3. second operation (external): interacts cell pairs of which the 
members are in different parts of the matrix. For the rest the same as 
the internal second operation. Dependencies: two matrix parts for which 
the second operation was done.

Note that by splitting up the second operation into multiple tasks, the 
result of the second operation will no longer be completely 
deterministic: if the order of execution for the internal and external 
tasks is different, then the order in which results are added to the 
second cell variable will also be different. This leads to different 
numerical round off error and hence a slightly different result. This is 
an inherent feature of task based parallelisation that we have to 
accept.

# Scheduling tasks

Once we have reformulated the algorithm in terms of small tasks with 
clear dependencies, we can execute the algorithm (in parallel) by 
*scheduling* these tasks. Scheduling a task means adding it to some sort 
of *queue* with tasks that are eligible to be executed from which 
computing units (*threads*) can obtain work. Clearly, we only want to 
schedule tasks that have no active dependencies, i.e. tasks that do not 
require other tasks to be run first and that do not use *resources* that 
are being used by other active tasks. It is very instructive to 
categorise these dependencies explicitly into
 * real *dependencies*: tasks that need to be executed before this task 
can be executed
 * *resources*: variables in memory to which this task needs unique 
access

Real dependencies can easily be handled by the scheduler that is 
responsible for adding tasks to the queue: we can add a dependency list 
to each task and only allow a task to be queued if that dependency list 
is empty. Resources can be handled by adding a similar list of resources 
to each task, and only allowing threads to execute a task that was 
queued after they have obtained unique access to those resources. In 
practice, this means that we need to add a locking mechanism to the 
resources. When a thread tries to execute a task, it tries to lock the 
resources. When this fails, it jumps the task in the queue and tries to 
execute the next task. When it succeeds at locking the resources, it can 
execute the task and other threads will not be able to execute tasks 
that use the same resources.

For our matrix example, the resources are the parts of the matrix. The 
locking mechanism consists of a single lock for each part. First 
operation tasks have a single resource (one matrix part), and so do 
internal second operation tasks. Internal second operation tasks can 
only be scheduled when the first operation task for the same resource 
has finished, this is their dependency. The external second operation 
tasks have two resources: the two matrix parts that they require. They 
also have two dependencies: the first operation task for the same two 
matrix parts.

# Queuing and stealing

The easiest approach to execute a task based algorithm is by using a 
single, shared queue for all threads. To make sure each task is only 
executed once, we need to lock that queue, which can lead to significant 
overheads. Task based algorithms therefore often opt for a single queue 
per thread instead. Each resource then keeps track of the thread that 
uses it most often and the scheduler tries to preferentially schedule 
tasks that involve this resource on that thread. This has as the added 
advantage that tasks that are executed by the same thread often use the 
same resources and hence make better use of the local memory cache.

However, this approach can also lead to load imbalances, as the queue 
for some threads might be considerably longer than that for other 
threads. Or because some threads happen to be a bit slower than other 
threads. To address this, a task stealing mechanism can be implemented, 
whereby *idle* threads (threads for which the queue is completely empty 
or all tasks in the queue have locked resources) can steal an available 
task from another thread. Although this imposes an actual computational 
overhead, it is still beneficial as otherwise this thread would be idle 
anyway.

# Truly parallel algorithms

A task based parallelisation strategy is in my opinion a truly parallel 
way of programming, as it no longer makes assumptions about the order in 
which operations are executed, but rather dictates the order in which 
operations *should* be executed. This shift in paradigm allows the 
programmer to give up this strict dependence on order and makes it 
possible for software to fully exploit the available CPU cores by 
executing whatever operation is most convenient at any given time. This 
means that a task based algorithm is not first doing operation A and 
then doing operation B, but can be doing operation A with 4 threads and 
operation B with 12 others simultaneously.

This is in stark contrast with more traditional parallelisation 
strategies that only try to parallelise operations A and B separately, 
but still require operation A to finish before operation B starts. This 
A-B dependency leads to additional bottlenecks that are completely 
absent in a truly task based algorithm.

Furthermore, a proper task based algorithm that defines clear resources 
also addresses the memory limitations of current systems, which again is 
in contrast to more traditional approaches that tend to use large 
structures (e.g. large matrices or grid structures) that are shared 
among all threads. Splitting large structures into many small pieces 
leads to a more efficient usage of memory but also to a more natural way 
to avoid memory conflicts. And it makes it also a lot easier to port 
algorithms to distributed memory systems, where different parts can be 
located on different nodes.

This was only a short introduction to task based parallelisation in 
which I only covered the basics. I hope to cover more specifics 
(including distributed memory parallelisation) in future posts. For now, 
I think it is also useful to reference THIS PAPER by some of my SWIFT 
collaborators that gives a very good introduction of the subject, and 
that also serves as the documentation for the QUICKSCHED library that 
can be used for your own task based parallel algorithms.
