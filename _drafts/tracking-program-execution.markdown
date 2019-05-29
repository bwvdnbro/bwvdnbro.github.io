---
layout: post
title: "Tracking program execution"
description: Some code I wrote this week to track the execution of a program.
#date: 2019-05-24
author: Bert Vandenbroucke
tags:
  - Scientific computing
  - Code development
---

I am currently exploring the limits of my code 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize) on an HPC system 
with a number of *fat nodes* (nodes with a large amount of RAM memory). 
The idea is to see how big a simulation I can run with the code on the 
available hardware (and within the available time). Part of this work 
consists of running large simulations for only a few steps, in order to 
gather as much as possible diagnostic information about the performance 
of the code: how long it takes to execute a single step, how long it 
takes to write an output file (a big worry for my simulations), how much 
memory the run requires... While doing this, I noticed that I had very 
little information about the overall balance of my simulations: I have 
very specific diagnostics to check the scaling of the code in various 
parts of the algorithm, but I do not have a good way of checking how the 
run time of various steps in the code compares.

To overcome this, I wrote a little helper class that makes it a lot 
easier to track the overall progress of a simulation and distinguish 
different steps in terms of performance. And then I wrote a Python 
script that can be used to visualise the output of this helper class. I 
will present both in this post.

# The helper class

In order to track the progress of a simulation, I will use a time log, 
very similar to the `MemoryLogger` I mentioned briefly in a [previous 
post]({% post_url 2019-04-18-memory-logging %}). In essence, the time 
log is a simple list with entries that have a name and a start and end 
time.
