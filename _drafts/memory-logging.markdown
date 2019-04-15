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
will necessarily use physical memory. If you are very cautious and 
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

Knowing that there is not just one type of memory, it is important to 
define what you actually mean with tracking memory. You might be 
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
obviously a lot of work. Too much work in fact to make it worth
