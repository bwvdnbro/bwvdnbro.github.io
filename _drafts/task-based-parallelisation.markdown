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
possible negative impact on the wellbeing of that human being). Note 
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

# How does it work?

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

Bla.
