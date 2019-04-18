---
layout: post
title: "Memory logging"
description: >-
  A summary of my attempts to come up with a useful memory usage logging
  method.
#date: 2019-04-07
author: Bert Vandenbroucke
tags: 
  - Code Development
---

One of the big issues I have as a code developer is keeping track of 
memory usage. While it is relatively straightforward to track 
performance using appropriately placed timers, there is no clear-cut 
solution to tell you how much memory you are using at any given moment 
during program execution. Which is annoying in cases where memory usage 
is a limiting factor, e.g. when running a very large simulation.

In this post, I will give an overview of things I have already attempted 
in order to make this problem more tractable. None of the solutions I 
provide are really elegant, but some of them are sufficiently flexible 
to be useful, which is all that matters.

# Different types of memory

First of all, it is important to note that there is no such a thing as 
just *memory*. RAM memory comes in two flavours: *physical* memory 
(which is the hardware equivalent of RAM memory) and *virtual* memory, 
which is the amount of memory allocated to your program by the operating 
system. Whenever you *allocate* a block of memory within your software, 
you ask the operating system to provide an address in the *virtual* 
memory which you can use to read and write to. It is the operating 
system's job to make sure this memory is actually available, either as a 
real piece of memory in the *physical* memory, or in some other way.

Note that this means that not every memory allocation in your program 
will necessarily use physical memory. If you are very greedy and 
allocate more memory than you need without ever using it, the operating 
system might decide not to allocate all of this memory in the physical 
RAM and your program might fit in less memory than you expect. But it 
also means that the operating system usually allows for quite a lot more 
virtual memory than it has available in RAM, in which case your program 
might very well force the operating system to use space on your hard 
drive (*SWAP*) in an attempt to free up enough physical RAM. And that 
usually leads to a situation in which your program deadlocks your whole 
system for a considerable amount of time.

This last issue is not at all unlikely, and has happened to me way more 
often than I would like to admit (in fact, it has happened at least 
three times in the past two weeks). I therefore strongly recommend you 
to manually limit the amount of virtual memory that is available to your 
experimental software by using

```
ulimit -v MAXIMUM_SIZE_IN_KB
```

within the terminal window in which you plan to run your software. If 
your program tries to allocate more virtual memory than the set limit, 
it will simply crash instead of rendering your system useless for the 
next half hour (or until you manage to kill your program).

# Keeping track of memory

Once you know that there is not just one type of memory, it is important 
to define what you actually mean with tracking memory. You might be 
interested in knowing exactly how much memory every memory allocation in 
your program actually allocates, in which case you want to track virtual 
memory. But if all you care about is making sure that your software will 
fit in the available memory on some machine, then you probably care 
about the amount of actual physical memory used by your software. 
Unfortunately, the latter turns out to be the hardest to achieve in 
practice.

## Keeping track of virtual memory

So let's start with the easy one. If you want to track virtual memory, 
then you simply need to log every allocation that is made, i.e. you make 
a little note of how much memory is requested for every call that is 
made to the relevant allocation routines. In a low-level language like C 
where every allocation is done using `malloc` or an equivalent routine, 
it is straightforware to create your own wrapper function for `malloc` 
that achieves exactly this.

In somewhat higher level languages like C++, things are a bit more 
complicated, as many objects are allocated indirectly through class 
constructors and standard library functions. It is possible to overload 
the `new` operator globally, but I cannot think of an elegant way to 
wrap this into a modular structure that is compatible with C++ thinking 
or that allows for a good way of labelling allocations. As a result, 
most solutions I have come up with so far do not attempt to accurately 
count every allocated bit, but instead focus on a select number of 
objects (let's call them the *usual suspects*: the objects of which you 
know that they require a lot of memory) or keep track of the total 
amount of allocated virtual memory.

My first attempt at manually logging object memory made extensive use of 
the `sizeof` operator in conjunction with some hard-coded functions to 
compute the memory size of `std::vector`s and manually allocated memory. 
I would basically provide every *suspect* class with a `get_memory_size` 
member function and then manually track the allocated memory whenever an 
object of that class was created. This worked very well, but was 
obviously a lot of work. Too much work in fact to make it worth the 
effort and use it as a sustainable solution.

A second way I only recently (read: this week) stumbled upon makes use 
of the pseudo file system that Linux distributions provide under 
`/proc`. This is a Linux only feature that is provided by the Linux 
kernel and provides a very powerful way for the kernel to communicate 
with other parts of the system. The way it works is as follows: your 
program (which runs within a unique *process* on your system) requests a 
file (or set of files) located under the `/proc` directory. The kernel 
catches this request and generates the corresponding file, tailored to 
the needs of the requesting process. No actual file is ever generated, 
but since the output of the request still takes the form of a simple 
text file, the requesting process can parse it as it likes and get all 
the relevant information.

You can easily generate a list of all available `/proc` *files* by 
querying the `/proc` file system:

```
ls /proc
```

This will generate a list of all currently running processes, and a list 
of *global* information files, like `/proc/cpuinfo` which contains 
information about the available CPUs on the system. To get information 
for the currently running process, you can simply query `/proc/self`, in 
which case the operating system will automatically display the contents 
of the `/proc` subdirectory for the requesting process, without you 
having to bother to figure out the ID of this process.

`/proc/self` contains a lot of useful *files*, but for our purposes we 
are only interested in `/proc/self/status` and `/proc/self/statm`. The 
former contains a human readable description of the resources used by 
the requesting process, including the current virtual memory usage 
(`VmSize`) and the maximum virtual memory usage since the start of the 
process (`VmPeak`). The latter is a stripped down version of this 
information that focuses on virtual memory usage only and that is not 
meant for human consumption. It is however ideal for our purposes.

Using `/proc/self/statm`, we can get the current virtual memory usage of 
the program using the following code:

```
#include <fstream>

std::ifstream statm("/proc/self/statm");
unsigned int vmem_size;
statm >> vmem_size;
```

The value present in `statm` is expressed in *page sizes*, where one 
*page size* corresponds to the size of a single *block* of memory on the 
system. These blocks are the way the operating system uses to organise 
the memory; it will always allocate memory in multiples of the page 
size. The actual page size is system dependent, although a typical value 
for it is 4096 bytes (4 KB). You can get the system page size as 
follows:

```
#include <unistd.h>

unsigned int pagesize = getpagesize();
```

Note that while the page size will probably fit in a 32-bit integer 
variable, the total virtual memory size might not. So it is probably a 
good idea to use 64-bit integers or `size_t` variables to manipulate 
these values.

Now that we have a way to get the *total* virtual memory size for the 
program at any given time, it is reasonably straightforward to set up a 
memory tracking routine: we simply determine the memory size before and 
after a certain object is created, and assume that the difference is due 
to the size of the object in memory. We still need to explicitly call 
the memory tracking routine in our code, but the overhead has been 
significantly reduced (and this immediately provides us with a good way 
of attaching labels to the memory logs).

Alternatively, we can also simply take snapshots of the total memory at 
various points during the program, and use these to assess which 
operations have the most significant impact on the program. The 
`MemoryLogger` class I wrote for 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize/) this week can be 
used for both and works well enough for my purposes.

## Keeping track of actual memory usage

As already mentioned, tracking the actual memory that is used is not 
straigthforward at all, as this depends on the inner workings of the 
kernel (and additional factors, e.g. how busy the system is at the 
time). If your program is written efficiently (in terms of memory 
allocations), then the actual memory usage and the virtual memory usage 
should be similar. If you don't know whether your program is memory 
efficient or not, then you will need to do something else.

HMM, maybe resident size can help here?
