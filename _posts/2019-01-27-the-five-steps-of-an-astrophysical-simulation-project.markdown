---
layout: post
title: "The five steps of an astrophysical simulation project"
date: 2019-01-27
description: An overview of my general workflow during an astrophysical simulation project
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Workflow management
---

During the past few years, I have conducted or been involved with quite 
a few astrophysical simulation projects. Despite being very different in 
subject (some of these simulations involved single stars, while others 
covered entire galaxies), I noticed these projects always roughly 
followed the same steps. Some of these steps were very obvious, while I 
only became aware of others after a few projects.

Generally, the steps you follow to complete some task are what we call a 
*workflow*. Basically everyone that does something is executing some 
workflow, but most people are not aware of that fact. And while being 
aware of your workflow does not necessarily make the workflow any easier 
to execute, it sometimes helps to improve it, or to make it more 
efficient.

Below I will try to give an overview of my typical workflow during an 
astrophysical simulation project. Workflows are usually very specific to 
a certain task and maybe even to a certain person, so I cannot promise 
that this workflow will be of any help. But then again, you never 
know...

# Step 1: design/study phase

Every project starts with an idea. This idea does not have to be your 
own (in fact, if you are an early career researcher, it usually isn't). 
The idea is a very vague plan for something to do, and can be based on 
various things: a discussion with a fellow scientist, a paper you 
recently read, or just something you came up with in the shower that 
morning.

In this first step, you need to make this vague plan more concrete. And 
the best way to do this is by digging into the library, or, more 
correctly, into some online search engine. There are two things you will 
be looking for: papers or presentations from people that have done 
something that sounds or looks similar to what you have in mind, and 
maybe more importantly, papers or presentations that show what you plan 
to do is impossible or much harder than you think it is. You need to get 
an idea of what people have already done in this area, and if possible, 
how they did it.

Based on that information, you can then start working out a more 
detailed plan of what you are going to do. What will you try to 
simulate? What questions do you hope to answer? What is needed to do the 
simulations (in terms of both hardware and software)?

Although this step is the most obvious step, I have to admit it is not 
my favourite, and I usually don't spend enough time on this. I prefer to 
skip ahead to the next step as soon as possible...

# Step 2: development phase

Once you know more concretely what you want to do, you can actually 
start doing it. Depending on what your more concrete plan turned out to 
be, setting up the components you need for you specific simulations 
might involve a lot or very little work, so this step can vary a lot in 
length.

During the development phase, you should make *all* the important 
implementation changes that are required in your algorithm or code so 
that you can actually run the simulations you want to run. It is very 
important to take your time for this; any bug you create now might haunt 
you later in the project. Also try to resist the temptation of already 
running production simulations with only part of the changes in place; 
even if you suspect your later changes will not affect these 
simulations, this can create version conflicts that might very well mean 
you have to rerun these simulations later anyway.

What you should do is run a lot of *dedicated* tests. If your change 
should have effect A, you should have a specific, inexpensive test that 
shows you indeed get effect A, and not effect B. Or that your effect A 
does not result in an unwanted side-effect C. Don't trust your instincts 
too much on this one; even if you think your change to the code cannot 
possibly be wrong, it is still worth testing. Your change might be 
perfectly fine, but it could still trigger a bug somewhere else that 
never caused problems before, or might have a larger impact on the run 
time or memory footprint of your simulation than you expected. A change 
is only successful and ready for production runs if you can actually 
convince yourself that it works based on simulation output.

If dedicated tests are not possible, you should try to find reference 
results that you can compare with. If someone else already did similar 
simulations, you can try to run something similar to what they did and 
see if you find similar effects. This type of testing is less rigorous, 
but might still tell you when something is clearly wrong in your 
implementation.

Apart from algorithmic changes, this is also the phase in which you 
should start thinking about useful output. Usually numerical simulations 
produce *snapshots*, i.e. dumps of the properties of the simulation at a 
fixed time. Depending on the size of the simulation, these snapshots can 
be quite large and you are hence limited in the amount of snapshots you 
can reasonably produce; a low snapshot frequency however means that you 
loose quite a lot of time resolution. Sometimes it is more useful to 
limit the output to some specific quantities (either for a subset of the 
available data or as a simulation average) but output these more 
frequently. This will require additional code that needs to be tested 
before you can use it in production simulations.

Note that the time you spend on the development phase is usually not 
proportional to its scientific impact and that is very easy to feel 
unproductive during this phase. But if your algorithmic changes are 
well-implemented, you might benefit a lot from them later. Moreover, I 
personally enjoy this phase the most.

# Step 3: prototype phase

This step can sometimes be seen as part of the development phase, 
although I think it is useful to make a distinction between the two. In 
the prototype phase, you need to show that your plan will actually work. 
You have designed the plan, made all the necessary changes, so now it is 
time to actually show that your new simulations can be done by running 
something very similar to what you actually want to simulate.

The difference between these prototypes and the tests mentioned as part 
of the development phase is their size: tests should be small and easy 
to run, while prototypes should be very similar in complexity to what 
you actually want to do for your production runs. There are two main 
targets for this phase. You first of all need to check that your code 
also works in production, i.e. that it can handle the size and the 
environment of the production runs. This means that you might very well 
discover additional bugs or issues that might force you to fall back to 
the development phase. Secondly, the prototype phase allows you to get a 
feeling for the *parameters* of the run, i.e. typical sizes, time 
scales, physical input values that lead to interesting results.

Depending on the outcome of the design and study phase, you might or 
might not have an idea of what sensible parameter values would be. In 
case you don't, there are several ways to proceed. First of all try to 
find some estimates for the sizes and physical conditions you are likely 
to find, based on what you find in literature. If you can't find 
specific values, you are still very likely to find upper and/or lower 
limits. Combined with some physical intuition, this gives you a good 
idea of the likely parameter value range.

Time scales are usually harder to obtain from literature (most 
astronomical phenomena happen on very long time scales and can hence not 
readily be observed). But they can usually be estimated from typical 
velocities, accelerations and length scales, so look for those. Most 
software that deals with time integration has some sort of time step 
criterion built in, so sometimes simply setting of a simulation and 
looking at those can give you a good idea of what the dynamical time 
scale of your system is. Simulations that only take of the order of 10 
time steps are usually not that very interesting...

Finding an appropriate numerical resolution is both straightforward and 
difficult. Theoretically, the optimal resolution is the resolution at 
which convergence is reached, i.e. the results *of interest* do no 
longer change if you increase the resolution. Usually this resolution is 
hard to achieve as (a) quantities of interest are rarely known in the 
prototype phase, and very often are the quantities for which convergence 
is doubtful, (b) numerical resolution is often limited by hardware, i.e. 
how much available memory or CPU power you have or how long your are 
willing to wait for your simulation results. Prototypes can again give 
you useful input to determine good values for these, but a proper 
convergence study should also be part of your production runs.

The output of the prototype simulations is also invaluable for the 
development of your analysis pipeline. It provides you with 
representative snapshots and other output files that will have the same 
type of content and a similar size to your production simulations. This 
means that you can use it to test your analysis scripts before the 
actual production runs have finished.

At the end of the prototype phase, you should be ready for production 
phase. This means that you should know exactly which simulations you 
want to run, know how long they will approximately run, and are 
confident that they will run without crashing.

# Step 4: production phase

The production phase is the phase where you actually do the research you 
want to do with the simulations. Based on the prototype phase, you 
should have one or more *reference* models that use parameter values 
that lead to interesting and physically plausible results. Usually you now
want to run a few categories of simulations:
 * convergence simulations: simulations with reference values for the 
physical parameters in which you vary the parameters that control the 
numerical resolution of your simulation. The aim of these is to 
demonstrate, in a rigorous scientific way, that your results are 
physical solutions of your model equations and that they can be trusted.
 * parameter studies: simulations with a (near) optimal resolution in 
which you systematically vary the values of physical parameters. The aim 
of these is to quantify the impact physical changes have on your 
results. These simulations usually cover the scientifically relevant
content of your research.
 * model studies: simulations with small variations of the physical 
model, like additional physical ingredients or a different way of 
computing a physical ingredient. Note that these model variations should 
have been covered by the development and prototype phases and that these 
simulations can only be usefully interpreted if you limit the amount of 
changes.
 * algorithm studies: simulations with small variations in the algorithm 
or with different values for algorithmic parameters. These are usually 
only relevant if you developed the new algorithm. Variations in 
algorithmic parameters for existing algorithms can be classified as 
convergence simulations, since then their only use is showing that you 
are using sensible values.

The production phase is usually not very labour intensive, but can take 
quite a lot of time, depending on the computational complexity of the 
simulations. This makes it a rather frustrating part of the usual 
simulation workflow, as it forces you to sit around and wait. You can of 
course (and you probably should) regularly check the status of your 
simulations. If your output is stored on an unsafe hard disk (i.e. a 
hard disk that is not automatically backed up on a regular basis), you 
should make sure that preliminary output data is backed up. It is 
important to be very methodical in this phase, to make sure you don't 
forget to monitor some of your simulations, and to avoid mixing up the 
output from various runs. I would therefore strongly recommend using a 
workflow management tool to manage your simulations (see a future post).

As mentioned above, there is also some work you can do while you wait: 
the development of a dedicated analysis pipeline. Of course, it is hard 
to predict exactly what analysis will be interesting to highlight the 
yet unknown results of your simulations, but you usually have some idea 
of what you want to analyse, so you can already start developing some of 
the necessary analysis scripts. The output of the prototype simulations 
should be very helpful in this process, as it provides you with data 
that has the same layout and should cover a very similar range of 
values. And it is also a good idea to apply your analysis pipeline to 
the early results of your production simulations, to monitor their 
progress (and catch problems early).

Ideally, the production phase ends when all your simulations finish 
without problems. It is however very likely that things will go wrong: a 
simulation might still crash because of unknown reasons, or it might 
turn out that some of your parameter values were not that interesting at 
all. So even in this phase, you might still be forced to adjust your 
parameter ranges, or worse, fall back to the development and prototype 
phase. Once you are forced to go back to previous phases, you will 
create inconsistencies between production runs that have already 
finished and new production runs, so things will get a bit messy. It is 
therefore good to try to avoid this scenario. This is why it is very 
important to spend enough time on tests during the development phase, 
and why you should see the prototype phase as a separate phase.

# Step 5: analysis phase

Once your production runs have finished (or when a sufficient fraction 
has finished), you can start analysing the results. This phase is very 
problem dependent, but usually involves making plots of quantities using 
some scripting language (I strongly recommend Python). These plots can 
show individual quantities for one snapshot of a simulation, or can 
track quantities or average quantities over time by analysing multiple 
snapshot (or by using dedicated simulation output with a higher output 
frequency).

Again, it is very important in this phase to be methodical, so that you 
don't mix up results from different simulations or different times. Also 
think about which analyses are computationally expensive and which ones 
can be redone very easily; it might be worth to save some of the 
analysis output rather than rerunning the full analysis every time you 
change a plot.

During the analysis phase, you can also start writing up your results in 
a scientific publication. Usually, this will inspire you to perform 
additional analyses, so the two go hand in hand. Or a co-author on this 
publication will suggest additional analyses (they should not suggest 
additional simulations; this should be done in the prototype phase).

At the end of this phase, you should be able to submit your scientific 
paper containing the results and findings of the project.

# Step 6: review phase

Scientific papers need to be peer-reviewed before they are published, 
and during this process additional issues might be raised. This might 
force you to go back to the analysis phase. Or worse. In principle, 
there is nothing wrong with running additional simulations to cover 
additional physical parameters and hence going back to the production 
phase. Things become more problematic when you need to go back to the 
development phase, as the changes you make there might affect the 
existing production simulations. I think it is acceptable to not do this 
for very expensive simulations, and limit your changes to anything that 
does not interfere with existing production simulations. If your paper 
does not crucially depend on going back to the development phase, you 
can probably argue as such in your comments to the reviewer.

In any case, the review phase poses a bit of a problem for the general 
workflow I have outlined here, as it does not fit in well with the five 
other steps (this is also why this post only mentions five steps in its 
title). This makes it even more important to be very critical about the 
work in the development and prototype phase yourself, so that you 
minimise the chance of a reviewer forcing you to revise these.

# General remarks

The workflow outlined here is more of an ideal scenario than a real 
workflow; I have to admit none of my projects ever exactly stuck to it. 
It does however contain some important pieces of wisdom that I think are 
useful to keep in mind. There are a few things in particular:
 * Be very aware of the difference between development tests, and 
production simulations. Development tests are just that: tests you run 
for yourself during development. If you plan to include those in your 
paper, you should include them in your prototype and production phase 
and make sure you have a good, reproducible way of running them. There 
is a difference between convincing yourself something works and 
convincing other people, and running the tests with a very specific 
version of the software that only worked during a brief moment during 
the development does not mean your code actually passes the test.
 * Also be aware of the difference between prototypes and production 
simulations. It is acceptable to include a successful prototype 
simulation in your production runs, but only if it fits within the range 
of parameters you want to explore in production. A prototype allows you 
a lot of freedom to change around things, which you shouldn't allow in 
your production simulations.
 * Be aware of the existence of the prototype phase and use it wisely. 
This is a good phase to plan the details of your production simulations 
and set up a rigid workflow that will allow you to execute the 
production phase very efficiently and systematically. It also provides 
you with all the information you need to start outlining the project 
paper and to develop your analysis pipeline. This is useful, as this is 
the only thing you can do while you wait for your simulations to finish.
