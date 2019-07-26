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
whatever instruction it was doing (it will make sure this instruction is 
carried out correctly), and call a *signal handler* that depends on the 
type of signal that was received. This signal handler is a limited 
low-level function that can perform a specific set of operations to deal 
with the signal, and that decides whether or not to continue program 
execution. Once the signal handler finishes, the CPU will pick up again 
the work it was doing, unless the signal handler decides to abort the 
process.

Note that a signal is only received by a single CPU. If a process is 
using multiple CPUs at the time of reception, the other CPUs will not be 
aware of the signal, unless the signal handler decides to notify them. 
This inevitably makes signal handlers vulnerable to race conditions, and 
seriously limits the number of safe operations they can perform.

Every signal that can be emitted comes with a default signal handler, 
and most of these signal handlers will simply immediately terminate the 
process. There are two signals for which this is always the case: 
SIGSTOP and SIGKILL, as these are the ultimate ways for the kernel to 
pull the plug on a process. For all other signals, it is possible to 
override the default signal handler and to either completely ignore the 
signal, or deal with in a less fatal way. Additionally, it is possible 
to suppress signals altogether, or suspend them temporarily.

# Handling signals

In C++, there are two ways to install a default signal handler for a 
process. The first way is using the `<csignal>` header which is part of 
the C++ standard library:

```
#include <csignal>
#include <cstdlib>

void signal_handler(int signal) {
  // do something
  abort();
}

int main(int argc, char **argv){

  std::signal(SIGINT, signal_handler);

  while(true){}

  return 0;
}
```

This example will install the function `signal_handler` as signal 
handler for `SIGINT` (*signal interrupt*), which is the signal that is 
typically sent to the process when you abort a process using `CTRL+C`. 
When you compile and run this example, the program will enter an endless 
loop, which you can break by pressing `CTRL+C`. The program will then 
execute whatever additional code is present in the `signal_handler` 
function, before calling `abort` to kill the process.

The UNIX standard defines a somewhat more general and robust variant of 
this API, which is contained in the header `<signal.h>` and that defines 
a function `sigaction` that is very similar to the `signal` function 
above:

```
#include <cstdlib>
#include <signal.h>

void signal_handler(int signal) {
  // do something
  abort();
}

int main(int argc, char **argv){

  struct sigaction signal_action;
  signal_action.sa_handler = signal_handler;
  sigemptyset(&signal_action.sa_mask);
  signal_action.sa_flags = 0;
  sigaction(SIGINT, &signal_action, nullptr);

  while(true){}

  return 0;
}
```

The additional `sa_mask` and `sa_flags` options in the `sigaction` 
structure allow for more control over the way the signal is handled; it 
is therefore generally recommended to use the latter approach when 
possible.

# What can a signal handler do?

Thus far, the `signal_handler` function in our example is not doing 
anything before it aborts the process. As mentioned before, it is very 
limited in what it is allowed to do, mostly because of its vulnerability 
to race conditions. A signal handler is for example not allowed to write 
output using the default output functions, as it might be called while 
another thread is using these output functions at the same time. In 
practice, this means the signal handler cannot (safely) do more than set 
some variables, but only if those variables can be set in an *atomic* 
(i.e. thread safe) way.

That being said, I personally think it is still possible to use unsafe 
operations within a signal handler, especially when that signal handler 
will ultimately still abort the process. If your program for example 
does not do any output for most of its time, then it is perfectly safe 
to interrupt the program while it is not doing output and do some output 
from the signal handler. While this is in principle not safe, it will 
work correctly in almost all cases, and when it does not work correctly, 
it will simply cause the program to crash, which would otherwise happen 
anyway.

Apart from setting some (global) variables (safe) and maybe providing 
some useful output to the user (unsafe, but in many cases acceptable), 
there is not much useful work that can be done inside a signal handler. 
This is also not required; the signal handler is likely invoked in the 
middle of something, and it is usually better to actively deal with the 
signal once that something has finished. Most signal handlers will hence 
use the global variables they are allowed to set as a flag to activate 
specific code later on that will deal with the signal.

The use of *global* variables is in this context a bit annoying, as they 
conflict with good program design. However, in this context, they are 
absolutely inevitable, simply because of the very nature of signals: 
signals can be caught at any time during program execution, and there is 
no guarantee that any context will be available at that time, except the 
global program context. It is however still possible to encapsulate 
global signal handler variables inside appropriate namespaces and to 
hide their global character as much as possible.

# Which signals should I handle?

The list of possible signals in the POSIX-standard is reasonably long, 
and most of these signals will never be emitted in any relevant context. 
A SIGILL (*illegal instruction*) for example is emitted when a program 
invokes a CPU instruction that is not available on the CPU it is running 
on. This is only possible if - somehow - the program machine code 
generated by the compiler contains an instruction that is actually not 
available on the machine for which the code was compiled. This is only 
possible if something went seriously wrong during the compilation 
process.

Other signals can be very useful, like e.g. SIGFPE (*floating point 
exception*), which tells you that a fishy floating point operation took 
place: a divide by zero or an underflow/overflow (where the result of an 
operation can no longer be stored within the precision of a floating 
point variable). These signals usually point at some problem in the 
actual computation, and handling them might be a good way of avoiding 
unwanted behaviour during your computation. Note that this signal can 
only be emitted if the kernel knows about these problems occurring. This 
in itself depends on *interrupts* being set for these events (see 
below), and on many systems these interrupts are not enabled by default.

One signal that is always useful to handle is the SIGINT encountered 
before, because it is the signal that is emitted when you try to 
manually kill the running program using `CTRL+C`. I found that I often 
do this because I want to *pause* a program, rather than completely stop 
it: I might have started the program with the wrong parameters or the 
wrong number of threads, so that it is slower than I would want it to 
be, or takes to long to complete. In this case, what I actually want is 
to store the state of the program when I press `CTRL+C`, and then 
restart from that state with better parameters. This is actually 
possible by installing a custom signal handler for SIGINT.

# Interrupts

Signals are somewhat related to another concept in program execution: 
*interrupts*. While signals are messages from the kernel (or hence the 
operating system) to the running process, interrupts are messages from 
the hardware or some specific software component to the kernel. When a 
floating point exception occurs for example, this will be noticed by the 
CPU that is executing the operation that causes the floating point 
exception. This will trigger a hardware event inside the CPU, that than 
will lead to an interrupt being sent to the kernel.

Since interrupts are ultimately hardware related, they are not limited 
to specific operating systems. Unless you are a kernel developer, there 
is however very little you can do with interrupts yourself, as they are 
ultimately dealt with by the kernel.

That being said, some interrupts can be manually enabled/disabled by 
your program, by invoking operating system or even hardware specific 
functions. This could for example be useful if you want to actively 
catch floating point exceptions (instead of making them cause `NaN` 
values).
