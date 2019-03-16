---
layout: post
title: "Timing and performance management"
description: >-
  A brief overview of methods to time code execution and measure code
  performance
date: 2019-03-16
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
  - Code Development
---

An important part of developing code is making sure that your code 
performs efficiently. There are several reasons you should do this. 
First of all, there is the obvious reason that it is good for you as a 
code user, because an efficient code means shorter run times and less 
time spent waiting for results. Second, and probably equally important, 
an efficient code uses less resources. This means that you pay less 
electricity to run your computer while the simulation runs. This is 
something you probably don't think about for small simulations on your 
own computer, but that is pretty important if you run large simulations 
on a large cluster. And if you are running on a large cluster, you are 
probably also sharing that cluster with many other users, all of which 
will appreciate it if you use less of the available computation time.

Measuring performance means several things. First of all, it means that 
you have to get some baseline idea of what the expected run time of your 
code should be: given the computations your code is performing and given 
the hardware specifications of a computer, how long do you expect the 
simulation to run? The second step is to measure how fast the simulation 
actually runs. It is very unlikely that these two numbers will match the 
first time you compare them; the actual run time is almost guaranteed to 
be longer than the expectation. To figure out why that is, you need to 
do something called *profiling*: you need to break down the simulation 
run into small parts and compare the time spent in each of those with 
your expectation. That will tell you how you can optimise your code.

Most hardware nowadays requires some level of parallelisation for 
optimal efficiency, as CPUs consist of many independent *cores* and 
(large) computers consists of many independent *nodes* that are 
independent computers that are linked up with a fast network. Efficient 
performance on these systems is measured in terms of *scaling*: a 
comparison of the run time of your simulations using various fractions 
of the parallel machine. The expected scaling is very easy to predict, 
but very hard to achieve, so again optimisations are usually required, 
based on a thorough analysis of your code.

Below, I will briefly outline the most important steps I mentioned here. 
I will start with the basic performance analysis of a serial code, and 
then move on to parallel code.

# Serial code

## Baseline predictions

Of all the topics I want to cover in this post, obtaining a baseline 
prediction for the expected run time of a bit of code is probably the 
hardest, as it is very hardware dependent. The question we need to 
answer is: given a piece of code, how long should it take to execute it? 
The only way to get a reasonable answer to this question is to go 
through the code and estimate the number of operations that needs to be 
performed for every step. Unfortunately, the former requires a lot of 
work, while the latter involves a lot of guessing, as the run time of an 
individual operation depends a lot on the specific hardware you are 
using, the optimisations carried out by the compiler (if your code was 
compiled) and even the layout in memory of the variables involved in the 
operation.

Despite this, you should be able to make some educated guess about the 
number of operations that you expect. You can assume that additions and 
subtractions take about one operation, while multiplications and 
divisions take more (some people advocate 5 as a good estimate). Loops 
multiply the number of iterations with the number iterations of that 
loop, etc. Once you have a rough number, you can try to obtain some 
hardware specific performance information: a CPU (or CPU core) running 
at 2.4 GHz (as advocated by the manufacturer) is expected to perform 2.4 
billion operations per second. So if your code involves anything around 
that number of operations (this might sound like a lot, but if you have 
a few nested loops in your code, you can very quickly reach this), you 
can expect a run time of about 1 second. Or maybe 10 seconds. But 
definitely not an hour.

Lacking a very good estimate, it will probably be impossible to show 
that your code performs optimally. There are two ways to deal with this. 
The first way is to assume that your code never performs optimally, and 
to always carry out the profiling that I will discuss next. If your code 
is spending most of its time in the bits where you expect it to spend 
most of its time, then you have some confidence that performance is 
good. The second way is by learning from experience: a small bit of code 
that involves very little steps is more likely to run at near optimal 
speed and will give you some idea of the speed you can expect. A larger 
code usually consists of many small pieces, and the run time will be 
close to the sum of the run times for the individual pieces. So once you 
have enough experience with specific algorithms and computations, you 
will probably develop some feeling of what performance is good and what 
is not.

## Execution measurement

Execution time can be measured in various ways. The easiest way is using 
an external tool like [GNU time](https://www.gnu.org/software/time/). 
You can simply prepend the `time` command to the command you want to 
run, and it will show you some performance measurements after the 
command finishes. This requires minimal effort and already gives you a 
rough idea of the total run time.

If you want more specific information, or you want to output timing 
information while the code is still running, then you will need to 
implement your own timers. Programming language usually have a timing 
function as part of their standard library that returns a time stamp 
whenever it is called (C++ has `clock()`). Operating systems usually 
offer their own versions of these that have better precision (Unix 
systems have `sys/time.h` and `timeval` with microsecond precision). And 
if you really want accurate results, you can even use the following bit 
of assembly code to get the CPU cycle counter straight from the CPU and 
store it in the provided 64-bit unsigned integer `time_variable`:

```
#define cpucycle_tick(time_variable)                                           \
  {                                                                            \
    unsigned int lo, hi;                                                       \
    __asm__ __volatile__("rdtsc" : "=a"(lo), "=d"(hi));                        \
    time_variable = ((unsigned long)hi << 32) | lo;                            \
  }
```

Whichever function you choose to use, all of these can be used to get a 
unique time stamp at the start and end of a block of code that you want 
to time, and the difference between end and start gives you a measure 
for the elapsed time between those two points. You can insert as many 
timing commands in your code as you want, and add these values together 
to get specific measurements for different parts of the algorithm. In my 
personal opinion, this kind of *instrumentation* is a must for any 
serious bit of simulation code.

Apart from dedicated timers, it is probably also a good idea to provide 
some time information in the output that your code generates at run 
time. It is very easy to use the same functions mentioned above to 
obtain a time stamp that can be prepended to every line of output your 
code writes to the terminal window. This information is very helpful to 
estimate the progress of the code from the terminal output alone.

## Profiling

Dedicated timers are incredibly useful, but they require manual 
intervention: you need to *instrument* the code by inserting timer 
instructions for the bits of the code you want to analyse. This 
inevitably means that you are limited in the amount of information you 
can gather by the number of timer instructions you are willing to add. 
*Profilers* are tools that improve on this by automatically 
instrumenting your code, either at compilation time or at run time (the 
latter is less common). By adding additional instructions they can 
measure the amount of time spent in various parts of the code, how many 
times a specific line of code was executed... Some profilers even allow 
you to measure the amount of memory that is in use throughout the code 
execution.

Profilers usually go hand in hand with a specific compiler. And just 
like the most powerful compilers that are specifically designed to run 
on specific hardware, they are usually not freely available. Free 
software alternatives like [GNU 
gprof](https://sourceware.org/binutils/docs/gprof/) are less powerful 
and impose a significant overhead. Powerful free tools like 
[scalasca](http://www.scalasca.org/) look promising, but are complex and 
hard to learn. Generally, profilers perform better in code that is not 
optimised (in that they provide more useful output), which unfortunately 
means that their output might not always be very representative for 
optimised code.

Despite all these issues, profilers can be incredibly useful to find 
hidden bottlenecks. If your code is efficient, then most of its time 
should be spent in those parts of the code that perform most of the 
required operations. If your profiler on the other hand shows that a 
significant fraction of the time is being spent elsewhere, then this 
might indicate that something does not quite work as you expected it to. 
For C++ programs, you might for example discover that a lot of time is 
being spent in the constructors and destructors of classes, which could 
indicate that you are not reusing objects in an efficient way. Or you 
might discover that an unreasonable fraction of time is being spent 
inside a function that you thought was very computationally cheap, 
because you wrote some incredibly inefficient code in that function.

Apart from exposing hidden bottlenecks, profilers also provide a good 
overview of where optimisations will be most beneficial. A function that 
is called only once does not have to be as efficient as a function that 
is called twice during each iteration of a long loop; shaving off a 
fraction of the run time for the latter will gain you much more than the 
same optimisation for the former.

# Parallel code

The general idea of parallelisation is to use multiple computing units 
*simultaneously* to perform a simulation in order to *speed up* the 
simulation. Ideally, using twice as many computing units should half the 
execution time, while it should also allow you to double the size of the 
simulation (whatever that means) and still run it in the same amount of 
time. These two ideas are called *strong* and *weak scaling*, and I will 
detail them below.

## Strong scaling

Strong scaling represents the idea that the total computation time $$T$$ 
can be distributed uniformly among the $$N$$ available computing 
units, so that the total time for each unit (and since they work 
simultaneously, the total run time for the parallel simulation) equals 
$$T/N$$. It can be measured very easily in terms of the *speed up*: if
$$T$$ is the serial run time of the code (using a single computing unit) and
$$T_N$$ is the total run time of the code using $$N$$ computing units, then
the speed up is

$$
S_N = \frac{T}{T_N}
$$

Ideally, the speed up should be $$S_N=N$$, but this is never true in 
practice. The reason for this is that every piece of code contains some 
*serial fraction*, i.e. code that cannot be executed in parallel, or 
that needs to be executed by all computing units and hence cannot be 
distributed. This serial fraction encompasses the code necessary to 
initialise the parallel environment and to set up basic variables, but 
also code that cannot be executed in parallel because that would lead to 
conflicts, e.g. two computing units that try to write to the same file 
at the same time.

As a result of this serial fraction, there will be a strict limit on the 
maximum speed up that can be achieved. This is quantified in terms of 
the parallel efficiency $$P_N$$, given by

$$
P_N = \frac{S_N}{N}
$$

The parallel efficiency usually decreases with increasing $$N$$, because 
conflicts become harder to avoid when more computing units are used.

A strong scaling curve consists of a measurement of the total simulation 
run time for a range of different values for $$N$$, all using exactly 
the same size of simulation (i.e. the same number of computations). 
Extrapolation of the curve allows you to estimate the minimum run time 
of a simulation using a given number of computing units. It also gives 
you an idea of what values of $$N$$ are reasonable to use: if the 
difference in speed up between $$N=32$$ and $$N=64$$ is very small, then 
it probably does not make sense to use the latter number, as you will be 
wasting most of the power of the additional 32 computing units.

## Weak scaling

Strong scaling is usually very hard to achieve, as it requires a very 
small serial fraction and a good way of dealing with or avoiding 
conflicts. Weak scaling is less problematic, and represents the idea 
that more resources allow you to perform more work.

In a weak scaling test, a series of measurements is performed as above, 
but now both the number of computing units and the amount of work are 
incremented: if $$W_N$$ represents the amount of work on $$N$$ computing 
units, then the simulation size for $$2N$$ computing units should be set 
to $$2W_N$$. Ideally, the run time $$T_N$$ in this case should be the 
same for each value of $$N$$, but again this ideal case is never 
achieved in practice. The main reason in this case is the *overhead* of 
a parallel run: using more computing units leads to the creation of 
additional work that was not present in the original serial simulation.

Apart from overhead, the weak scaling is also affected by the precise 
choice of work value $$W_N$$. Ideally, this value should have some clear 
meaning in terms of scientific size of the simulation; it could for 
example represent the resolution of your simulation. In this case, the 
overhead can also be caused by the algorithm itself; if parts of your 
algorithm do not depend linearly on the resolution, then an increase in 
your resolution can also lead to a more than linear increase in amount 
of work.

With a weak scaling curve, you can extrapolate a small simulation to a 
larger simulation and assess how many computing units you need to use to 
run this simulation in a reasonable amount of time. When you apply for 
computing time on a large system, you will usually need to show a weak 
scaling curve for your specific problem, to show that the simulation you 
plan to run can actually run on the requested resources.
