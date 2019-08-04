---
layout: post
title: "Units and unit conversions"
description: >-
  An attempt to make the use of units in scientific computations less painful.
date: 2019-08-04
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
---

Units are very important in science, as they provide a mapping from the 
abstract realm of maths to the reality of physics. Maths tells us 
exactly how to compute the circumference of a circle with a radius of 
$$2$$, but it does not tell us the actual size of that circle compared 
to any other size that we know. If the radius is $$2$$ cm, then the 
circle is very small and probably drawn on a piece of paper as part of a 
calculation. If the radius is $$2$$ km, then the circle is rather large 
and might be part of a particle accelerator. And who knows what the size 
of the circle is if the radius of the circle is $$2$$ miles?

All of this is pretty well established and sounds incredibly trivial. 
Nonetheless, units are still a very big issue in computational sciences 
(again, I can only really judge computational astrophysics). With good 
reasons.

First of all, computers are pretty much very basic but fast 
mathematicians: they can do a lot of calculations (with a very limited 
number of operations), but they do not attempt to interpret any of 
those. For a computer, just like for a mathematician, the radius of the 
circle is $$2$$, and it is up to you as a user to interpret this value 
with appropriate units. The computer itself does not care about units, 
neither does it keep track of them. Second, this inevitable transfer of 
responsibility from the computer to the user means that the user somehow 
has to keep track of units throughout a computation, i.e. has to 
actively do something to make sure the input and output of the 
computation makes physical sense. I probably have complained before 
about the lack of proper documentation in many astrophysical simulation 
codes; since code documentation is the most straightforward place to 
provide the necessary unit information, it is clear to see the issue. 
Third and last, there seems to be a lack of interest in units within the 
community, maybe because most test cases for simulation codes use "code 
units" and unit handling is never properly tests, or because scientists 
don't like to admit they have problems with something as trivial as 
units, or because units are just a distraction of the real science, 
which tends to be captured by complicated equations.

The fact is that units are a problem; I have personally struggled with 
simulation units way to often, and I have experienced at least one 
occasion during which a senior scientist messed up unit conversions and 
caused a lot of problems for a student that was trying to replicate that 
scientist's work.

So in this post I intend to discuss how I deal with units, and what I 
think should be the minimum requirements for unit handling in any 
scientific simulation code.

# The basics: quantities

While units and especially the conversion from one unit into another can 
be incredibly painful, the basics underlying any set of consistent units 
(or *unit system*) are incredibly simple. In physics, there are only 
seven independent *quantities*: length, time, mass, temperature, 
electric current, luminous intensity, and amount of substance. As an 
astronomer, I prefer to also add plane and solid angle to this list, as 
angles are typically treated as independent quantities. A unit system is 
consistent if all these quantities have a unit, and if the unit for each 
quantity is the same whenever that quantity appears in a derived 
quantity (e.g. a velocity: length divided by time).

Whenever you do a computation on a computer, you *have to* choose a 
consistent unit system, as the equations that are evaluated during the 
computation are agnostic about the conversions between different units 
for the same quantity. This means that all quantities you input at the 
start of the computation should be expressed in the unit system of 
choice, and also that all quantities that come out of the computation 
(your result) will be expressed in that same unit system.

So before you start worrying about units, you should be aware of the 
quantities that are involved in your calculation. In many problems that 
involve *dynamics*, there will only be a limited number of these: 
length, time and mass. If temperatures are involved, they are usually 
only used in conjunction with Boltzmann's constant $$k$$ and converted 
into energies, which again only involve length, time and mass. Electric 
current is only used in problems that involve electromagnetic forces, 
and again, are usually converted into forces by means of appropriate 
physical constants. Luminous intensity is not used for dynamics. And 
amount of substance is pretty much a counting exercise, which again has 
no impact on dynamics. So length, time and mass are your most likely 
candidate quantities. In which case you have to know only three units to 
fix your unit system.

# Fixing a unit system

So how do you fix your unit system? And what does it actually mean to 
fix your unit system? Let's look at the first question first. For 
simplicity, I will assume a (hydro)dynamical simulation that only 
involves length, time and mass. To fix the length scale, you typically 
need to choose a spatial scale for your simulation. Many simulations 
involve a *simulation box* that is a cube or rhombus with specific 
dimensions. Choosing the length scale then boils down to defining what 
physical size scale this box has, and what number to use for that same 
physical size scale. If your box is for example a cube with a side 
length of $$1$$ km and you choose $$2$$ as the box length in your 
simulation program, then this fixes the length unit to 
$$\left(\frac{1}{2}\right)$$ km (because $$2$$ times the length unit 
equals $$1$$ km).

Fixing the mass unit is very similar: you choose some mass scale of 
interest (if you are simulating the solar system, then the mass of the 
Earth or Sun could do) and choose a value for this same mass within the 
computation.

The time unit can in principle also be fixed like this, although this is 
not what usually happens. And this is where things get more 
*interesting*. When choosing *internal* (computational) values for 
quantities, a scientist usually tries to choose values that are close to 
$$1$$, as these values are more accurately represented within the finite 
precision of a floating point variable. This is especially true if a 
computation uses single precision floats, as they have a limited 
precision. This means that $$1$$ is a good size for a simulation box (or 
$$100$$ or $$10,000$$, but definitely not $$10^{50}$$). Similarly, $$1$$ 
is a very good average mass for all masses in the computation. But for 
the time unit, there is not really such a choice.

The problem is that dynamical simulations try to *solve* for the time 
evolution of a system. This means that you know the state of the system 
at one time (in arbitrary time units), and want to evolve it forward in 
time to an arbitrary later time, but preferably a time that is 
sufficiently far away from the initial time so that interesting dynamics 
can happen in between. Since you don't necessarily know what that later 
time should be, it is not so clear how to choose the time unit so that 
the size of the total time interval is small when represented on a 
computer.

This can however be estimated from other variables. (Hydro)dynamics is 
all about movements: the current state of a system gives rise to some 
net *forces* that accelerate material and cause it to evolve into a 
different state. The time scale of these movements will depend on the 
size of the net forces, and can usually be characterised in terms of a 
*dynamical time scale* or a *characteristic velocity*. In purely 
gravitational problems like the movement of the Earth around the Sun, 
the dynamical time scale could e.g. be the orbital period of the Earth. 
In problems involving hydrodynamics, the *sound speed* provides a 
characteristic velocity; this sound speed depends on the temperature of 
the medium.

If a dynamical time scale can be derived for your problem, then it makes 
sense to use that as simulation time unit, and then the conversion 
factor can be determined as before. If on the other hand you only have a 
characteristic velocity, then some more thinking is required. Knowing 
the characteristic velocity, you can compute how long it will take for 
something moving with that velocity to cross the side of your simulation 
box (because you already fixed the length unit). You can use that value 
as your internal time unit, and use the technique from before to fix 
your length unit. Note that in some cases this technique can be somewhat 
counter-intuitive. If your characteristic velocity is for example the 
sound speed, then fixing the time unit will require you to know the 
temperature of the system and the size of the simulation box!

Now: what does it mean to fix the unit system? At the end of the 
procedure described above, you will have *internal units* for all three 
basic quantities that are involved. These internal units are nothing 
more than conversion factors *from* internal simulation values *to* real 
physical values. Note that the internal units themselves have units, 
while the internal quantities do not. You do need to keep track of what 
the units of the internal units are!

At the start of the simulation, you need to convert *from* physical 
units *to* internal units, which means you need to divide all input 
quantities by an appropriate internal unit (or combination of internal 
units). Whenever the simulation produces an output, you then need to 
convert back *to* physical units by multiplying with the same (or 
appropriately different) internal units.

# What can go wrong?

Many things. An obvious mistake that everyone makes at some point is 
mixing up divisions and multiplications. Things are fine as long as you 
realise that simulation units - as conversion factors - have units 
themselves, because then it is clear that you need to *divide* physical 
quantities by them and *multiply* simulation quantities with them. But 
it is very easy to forget to remember of keep track of the units of a 
conversion factor, and only remember its value. Especially if the 
original physical unit is some reciprocal (e.g. cm$$^{-3}$$), then this 
can lead to confusion (while $$1$$ m is definitely larger than $$1$$ cm, 
$$1$$ m$$^{-3}$$ is a lot less than $$1$$ cm$$^{-3}$$).

Another thing that can go wrong is the determination of your internal 
unit system. As pointed out before, many simulations only use three 
basic quantities, which means you can only fix three units (all the 
others are derived from those). If you still try to fix more than three 
units, you end up with inconsistencies which will either cause a lot of 
trouble during your simulation, or will result in output having 
different units than you expect. A typical example is a hydrodynamical 
simulation where the time unit depends on the sound speed and hence the 
temperature. While temperature as a quantity has its own unit, fixing 
the time unit using the temperature is completely independent of the 
temperature unit, and does only affect the time unit. After setting the 
time unit, you can no longer set any other units that might affect the 
time unit (like e.g. the velocity unit)!

Another typical problem with units is the use of physical constants. As 
I already pointed out before, physical constants can be used to get rid 
of some quantities and convert them into more dynamically relevant 
quantities. If you multiply a temperature with Boltzmann's constant, you 
get an energy (mass times length squared divided by time squared). 
Similarly, if you multiply a mass with Newton's constant, G, you get a 
volume acceleration (length cubed divide by time squared). If you choose 
to set the value of a physical constant to something other than its 
actual value in internal units (typically $$1$$), then this 
automatically sets one unit in the internal unit system. This is a very 
subtle way to overfix your unit system without realising it.

# What should we do?

Personally, I have experienced the pain of unit conversions enough 
during my scientific career, and I have developed some ways of avoiding 
a lot of the problems that I encountered in the past. I am not saying 
that these are perfect solutions, but I have seen some of these being 
used by other simulation codes, and I know that they work for me.

The first and most obvious way of dealing correctly with units is being 
aware where units are important: for simulation *input* and simulation 
*output*. Every value that gets fed into a simulation code should have a 
unit, and it should be clear what that unit is. This is also true for 
values that are read in from parameter files or even from the command 
line. In my recent simulation codes (e.g. 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize)) every value in the 
parameter file has to have an explicit unit (e.g. you need to write 1 m 
and not just 1 for a length quantity), and the same is true for all 
quantities read from other input files (unless units are provided in a 
different way). It took quite a lot of effort to write the code that 
deals with reading in these units and automatically converting 
quantities from input units to internal units, but all of this code only 
needs to run at the start of the simulation, so it has no significant 
impact on the simulation run time. And it has made dealing with units a 
lot easier.

Making sure units are present in simulation output is even easier: you 
just need to make sure that whatever output format your simulation code 
uses has a way of attaching some *metadata* to output fields in which 
you explicitly specify the units.

For code developers, dealing with units is - as already mentioned above -
pretty much only a matter of providing sufficient documentation in the 
code. If a function takes a physical quantity as input and outputs 
another physical quantity, you should either document the units of both 
quantities, or (if internal units are used) specify the basic quantities 
for both values. This makes it possible to rigorously check that all 
units inside the function are correct.

So in conclusion: keeping track of units is pretty much just a matter of 
providing *information* to whoever uses or further develops your code, 
so that everyone knows what goes in and comes out of the code.
