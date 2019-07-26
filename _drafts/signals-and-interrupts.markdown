---
layout: post
title: Signals and interrupts
description: A brief introduction to signals and what they can do.
date: 2019-07-26
author: Bert Vandenbroucke
tags:
  - Scientific computing
  - Code development
---

When an operating system starts a software program, it will typically 
create a process for that program and hand that process control over 
some hardware resources. The process can then use those resources to run 
and request more resources until it decides it has finished and 
terminates. This is the normal flow of program execution.

However, sometimes the operating system might need to take back control 
of the hardware resources used by a process before the program finishes, 
e.g. because another process has a higher priority, or because the 
program is ill-behaved: it does not finish within the expected time, 
tries to access memory that was not allocated (a *segmentation fault*) 
or executes an illegal CPU instruction. Since the operating system is 
ultimately in control of the hardware resources, there are some 
mechanisms through which it can take back control from a running 
process, with different levels of urgency.

# Signals

When a process is running, it could be doing a large variety of things, 
but all of these will involve usage of the CPU. The most efficient way 
for the operating system *kernel* to send signals to a running process 
is hence by sending a message to one of the CPUs that is currently 
executing the process. This kind of message is called a *signal*. The 
availability of these signals depends on the type of operating system: 
all Unix and Unix-like operating systems that are *POSIX-compliant* have 
them. A full list of possible signals is available from 
[Wikipedia](https://en.wikipedia.org/wiki/Signal_(IPC)#POSIX_signals). 
What signal is sent will depend on the reason for the signal, and will 
also determine the urgency with which the signal needs to be handled.

When a CPU receives a signal, it will almost immediately stop executing 
whatever instruction it was doing, and call a *signal handler* that 
depends on the type of signal that was received. This signal handler is 
a limited low-level function that can perform a specific set of 
operations to deal with the signal, and that decides whether or not to 
continue program execution. Once the signal handler finishes, the CPU 
will pick up again the work it was doing.

Note that a signal is only received by a single CPU. If a process is 
using multiple CPUs at the time of reception, the other CPUs will not be 
aware of the signal, unless the signal handler decides to notify them. 
This inevitably makes signal handlers vulnerable to race conditions, and 
seriously limits the number of safe operations they can perform.

Every signal that can be emitted comes with a default signal handler, 
and most of these signal handlers will simply immediately terminate the 
process. There are two signals for which this is always the case: 
SIGSTOP and SIGKILL, as these are the most urgent signals that allow the 
kernel to immediately take back control of the process resources. For 
all other signals, it is possible to override the default signal 
handler.
