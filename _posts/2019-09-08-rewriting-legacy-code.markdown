---
layout: post
title: "Rewriting legacy code"
description: >-
  What to do if you are asked to rewrite a code that was written by a scientist
  half a century ago.
date: 2019-09-08
author: Bert Vandenbroucke
tags: 
  - Code development
  - Astronomy
---

Legacy code is a general term used for software that was conceived and 
developed a long time ago, but that for some reason is still actively 
being used. Many sciences (astronomy is one of them) depend a lot on 
this type of code, since many of the algorithms and libraries that are 
often used in current research were developed in the past and come in 
the form of legacy code.

Scientists tend to be very careful when it comes to legacy code. Science 
is to a large extent standing on the shoulders of giants, so it seems 
inconceivable that a young scientist would be able to reinvent an 
existing library and do a better job at it than the original developer 
who presumably devoted an entire career to it. And even if that were 
true, why would you take the risk of potentially introducing new and 
unknown bugs in a piece of code that is heavily used, while the legacy 
code worked just fine? Legacy code usually consists of many thousands of 
lines of code, so rewriting it might simply be impractical.

Sometimes however, there is no choice but to touch legacy code. Some 
applications of existing tools are so drastically different from the 
applications for which the tools were originally developed, that it 
becomes simply impossible to just incorporate the original tool in a new 
project. Or incorporating the old tool in the new project can become so 
messy that it is simply no longer justified to do so. In that case, 
there is no choice but to open the original source code files, and with 
them a whole can of worms.

# What is legacy code like?

Since this post stems to a large extent from my own current frustrations 
with rewriting legacy code, the following paragraph might exaggerate or 
generalise some of my personal experiences. Nevertheless, I have heard 
some of my complaints reflected by other people, so I am confident that 
some people share my opinions.

The first thing to know about legacy code is that it (per definition) 
was written in a different time period. This means that the programming 
language used to develop it was probably different from the languages 
that are popular today (Fortran77 was a very popular language in 
scientific computing for a long time), but also that code development 
had different issues back then than it has now. While code developers 
nowadays care a lot about scalability and making sure that codes exploit 
the parallel nature of modern computers efficiently, legacy code was 
almost exclusively developed for serial CPUs and computers with limited 
amounts of memory. Legacy code therefore often does complicated things 
to save on memory, or sacrifices accuracy by using single precision 
rather than double precision variables. This immediately provides a 
counter argument for the *reinventing the wheel* argument I made above: 
while legacy code developers did indeed know very well what they were 
doing for computers in their time, their optimal code might no longer be 
optimal for modern computer systems that have very different issues.

Another general property of legacy code is the lack of modularity (a 
concept that was not very familiar in legacy times), trust in compilers 
(they are a lot better now than they were then) or complicated version 
control (tools like `git` simply did not exist yet). This means that 
legacy code might look and feel a bit strange to someone that is used to 
modern programming. Especially the lack of trust in compilers can be 
hard, as it means that legacy developers often did a lot of manual code 
optimisation that makes code a lot harder to read. The lack of 
modularity can make it hard to figure out how variables are stored and 
shared between different parts of the program. The lack of advanced 
version control means that you will either have only one particular 
version of a legacy code, or many versions that are linked in ways that 
are not obvious.

A final problem with legacy code is the general lack of good 
documentation. Although this is not really a problem that is exclusive 
to legacy code, as many modern codes also suffer from this issue. That 
being said, the lack of documentation, combined with the somewhat 
different programming style in a somewhat different language can make 
deciphering legacy code very challenging...

# How do I start rewriting legacy code?

To rewrite legacy code in a modern way, you first need to understand how 
the legacy code works. This inevitably means reading it, following it's 
train of thought (or the train of thought of the legacy developer that 
wrote it), and somehow recording this information in a way that makes it 
easier for you to rewrite it. It also means suppressing your constant 
frustrations with how different legacy code is from what you would 
write, or finding a way to vent these frustrations that does not involve 
you physically attacking your computer or giving up on the whole 
rewriting project (I find that complaining to other people about it 
generally works).

I personally find that it helps to simply translate the legacy code into 
a very flexible language like Python during first read, since this 
forces you to read the code, but also already makes it easier to start 
the rewriting process properly later (since you already have a version 
of all the code). This Python version is in no way meant to be the final 
rewritten version (it will probably be too slow for that anyway), so you 
can be quite free in how you write it, and adopt the memory management 
and non modular structure of the original code. You will have to get rid 
of some features of the original code though, for example the `goto` 
statements (which you will unfortunately often encounter).

Of course, the whole rewriting process only makes sense if at the end of 
the day (or week, or month, because that is how long the process is 
probably going to take) you have a new version of the code that still 
does *exactly the same thing* it did before. To make sure that is the 
case, you need to make sure that you have a system in place to check the 
results of the rewrite against the old version, based on the assumption 
that the legacy code is always correct. This means making sure both your 
version and the legacy code provide ample output (you might need to 
change the legacy code to do this!) that can be compared and that covers 
all the possible paths through the legacy code.

Once you have a first rewritten version of the legacy code, you can 
start the long process of *refactoring*, i.e. changing the structure of 
your version without changing what it does. This process involves 
identifying possible modules within the legacy code, and then slowly 
introducing modern programming concepts like classes and templates where 
appropriate, always making sure that the code reproduces the same output 
as before. This step also involves identifying special functions or 
library calls in the legacy code and deciding whether you need to 
implement these functionalities yourself, or whether you want to use an 
external library for this.

Given the general lack of documentation in legacy code that probably 
frustrated you immensely during the first rewrite, it is also a good 
idea to add extensive documentation during the refactoring step. Use 
your own experiences when trying to understand the original code in 
identifying the bits of the code that are especially tricky to 
understand, and document them extra carefully. Remember that you are 
doing this rewrite because the legacy version is just no longer useful 
as it is, so make sure that your version is better than that.

# What if the legacy code is *not* correct?

One thing I noticed during all the rewrites I have done in the past, is 
that legacy code hardly ever is as good as we would like to think. All 
code is buggy to a certain extent, and this is especially true for code 
that does no attempts at all to avoid bugs: there are no unit tests in 
place, no documentation is present that might help finding a bug in 
expressions, and the code was very often written by a single person, so 
that nobody else had the opportunity of finding small typos or mistakes. 
You might very well end up finding (lots of) bugs during your rewrite of 
the legacy code, and that can cause some problems.

The first, obvious problem is that bugs in the legacy code mean that 
your reference for the rewrite is no longer correct. This is a 
fundamental problem that you need to address, as having a reference is 
vital to ensure that your rewrite works. The only solution is to solve 
the bug in the original code, and rerun it to make sure you have a valid 
reference again.

A second, more significant issue is that legacy bugs might change the 
results of the code altogether. Note that I don't mean a difference of 
fractions of a percent (which would be an issue when you compare your 
rewrite to the legacy code), but significant differences of tens of 
percents or more. These differences are significant enough to 
potentially invalidate old published results obtained with the code. If 
this is the case, then you need to be very careful and very thoroughly 
document what the bug is, and how it affects the results. You then might 
want to contact the original developer of the legacy code and signal 
this issue. If this turns out to be impossible, then you should 
definitely describe this issue in detail in some publication that makes 
use of the rewrite of the legacy code, to make sure the scientific 
community is aware of this bug.

# Why should I bother rewriting legacy code?

It might be clear from the above that rewriting legacy code is a very 
long and frustrating process. This might make you question the whole 
idea of doing it in the first place. I can think of a few good reasons 
why it is definitely a good idea to rewrite legacy code:
 1. If legacy code is poorly documented and has no unit tests, then it is very
likely buggy. A rewrite might be the easiest way to discover the bugs, and is
not that much more work than equipping the legacy code itself with unit tests
and proper documentation, which should definitely be done.
 2. Code that was written decades ago probably uses old libraries or 
uses custom implementations for functionality that is part of modern 
libraries. Knowing that optimisation strategies in the past were very 
different from what they are now, these implementations might not be as 
accurate or efficient as you would want them to be, so it is definitely 
worthwhile to replace them with modern versions.
 3. Some programming languages (like Fortran77) are slowly dying out. It 
is probably a good idea to rewrite legacy codes using these languages 
now, while there are still people who know how to read these languages 
and compilers that can actually compile the legacy code so that you can 
obtain reference output.
 4. Modularity generally makes it a lot easier to extend existing 
software with additional functionality. Knowing that, it is probably a 
good idea to rewrite non modular legacy code in a modular way, so that 
it becomes much easier to maintain and extend it in the future.

On top of that, I personally strongly believe that scientific software 
should be open source and thoroughly documented, as that is the only way 
to guarantee reproducibility of scientific results (an important pillar 
of the scientific method). Legacy code very often is not open source, 
and rewriting it might be the only way to make it available to the 
community.
