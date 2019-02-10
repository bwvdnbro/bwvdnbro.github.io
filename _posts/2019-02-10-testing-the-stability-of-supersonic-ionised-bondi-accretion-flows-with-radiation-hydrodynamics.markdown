---
layout: post
title: "Testing the stability of supersonic ionised Bondi accretion flows with radiation hydrodynamics"
#date: 2019-02-03
description: An introduction to and summary of my latest astrophysical paper.
image: /assets/images/bondi_paper.png
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Workflow management
  - Astronomy
---

Last week, a scientific paper that has been in my scientific pipeline 
for over a year got accepted for publication in Monthly Notices of the 
Royal Astronomical Society ([Vandenbroucke *et al.*, 
2019](https://doi.org/10.1093/mnras/stz357)), and this seems a good 
opportunity to explain what this paper is about. It also allows me to 
tell you some more about the machinery behind the paper, which made 
extensive use of workflow management systems.

# Introduction

The paper itself is theoretical in nature, but its content relates to 
the very active field of star formation. It seems a bit odd that a 
scientific field that is literally named after stars (*astronomy* 
derives from *astron*, the ancient Greek word for *star*) still hasn't 
figured out completely how stars form, but this is the case to a certain 
extent. We know that stars form in dense clouds of cold interstellar gas 
when small cores within these clouds fragment and collapse under the 
clouds gravity, but there are many details about this process that still 
need to be ironed out. The main problems are that these dense clouds are 
incredibly hard to observe (they are opaque to most wavelengths of 
radiation) and that there are many physical processes that contribute to 
the formation and collapse of these clouds, making it extremely 
challenging to model the star formation process theoretically.

One of the big outstanding questions in star formation deals with the 
end of the star formation process. We know that the collapse of a 
protostellar cloud is halted when the density and temperature in the 
core of the star reach the extremely high values needed for *nuclear 
fusion*, the process that makes stars shine. This fusion process 
releases a huge amount of energy that leaks out of the star as light, 
and on its way out causes a huge counter-pressure that counteracts the 
gravitational attraction that tries to make the star collapse even more. 
This explains nicely why a protostar stops *collapsing*, but does not 
tell us why it stops *growing*: material from the edge of the cloud can 
still *accrete* onto the central core and make the star more massive. At 
some point, some process is responsible for stopping this accretion, and 
this will ultimately stop the formation of the star and determine its 
final mass. Since the mass of a star is the main parameter that sets the 
stellar properties (like its brightness and how long it lives), getting 
a handle on the final mass of a star is quite important.

Together with collaborators in Brazil and additional collaborators 
around the globe, my group in St Andrews is trying to find out if the 
radiation emitted by a protostar can shut down the accretion onto that 
protostar. We are particularly interested in massive stars. These stars 
are the most short-lived and we know that they are already very bright 
while they are still accreting material, making it very likely that the 
pressure caused by high-energy radiation will affect the final stages of 
the accretion onto the protostar.

In 2002 and 2003, Harvard professor Eric Keto wrote two papers in which 
he investigated the accretion process onto a massive star in the case 
where the star emits *ionising* radiation. This is radiation that has 
sufficient energy to *ionise* the interstellar medium, i.e. strip 
neutral hydrogen and helium atoms in the gas surrounding the protostar 
from its electrons to turn it into a hot plasma. We know that this 
photoionisation process transfers a lot of energy to the gas, turning it 
from a cold gas with temperatures around $$100-500$$ K into a hot gas 
with a temperature of $$\sim{}8,000$$ K. At constant density, the 
*pressure* in the gas linearly depends on its temperature 
($$P\sim{}T$$), so that this hot plasma has a much higher pressure than 
the surrounding cold, neutral gas. In a normal scenario, the ionising 
radiation will create a hot bubble around the star, and the higher 
pressure inside this bubble will cause the bubble to expand into the 
interstellar medium. We call this an HII region (*HII* is the spectral 
name of ionised hydrogen, the main gas component inside the hot bubble).

This expanding bubble scenario works well when the radiation of the star 
is the only thing that exerts a force on the gas. And that is not quite 
the case for a massive forming star that is pulling in surrounding gas 
because of its strong gravitational pull. This gravitational force is 
stronger close to the star. Based on earlier work by Hermann Bondi and 
Leon Mestel in the 40s and 50s of the 20th century, Eric Keto 
investigated what happens when an expanding HII region is confined to a 
small spherical volume close to a massive accreting protostar. He showed 
that when the HII region is very small, the gravitational force 
completely dominates, and is much stronger than the pressure force that 
would normally cause the expansion of the HII region. The result is that 
the HII region does not expand, but stays confined around the protostar: 
a trapped HII region. Even though the star heats the surrounding gas, 
the accretion onto the star can still continue.

As the star keeps growing through the trapped HII region, its mass and 
hence its brightness (or what astronomers call *luminosity*) keeps 
increasing as well, so that the HII region slowly grows. Once its size 
exceeds a critical limit, the pressure force in the hot bubble starts to 
win from the star's gravitational pull, and the bubble will manage to 
expand. The expanding bubble will then push away the gas surrounding the 
star and shut down the star formation process. That is at least Eric 
Keto's theory.

There are a few issues with this theory. First of all, these models were 
based on a static mathematical description of the problem, and do not 
actually contain the dynamic evolution of the hot bubble. This means 
that Eric Keto only showed that this scenario is *possible*, not that it 
will actually also *happen*. Secondly (and more importantly), these 
models crucially assume a spherical HII region surrounding the star. 
Observations of star-forming regions almost unanimously favour a 
different scenario, whereby a star forms in a flattened, disc-like 
structure called an *accretion disc*. In this scenario, the *polar 
regions* (perpendicular to the disc) have a much lower gas density, so 
that the shape and evolution of an HII region in such a scenario will be 
different from the nicely spherical hot bubble described above.

To address these issues, we decided to model the development and 
evolution of these HII regions surrounding massive protostars in more 
detail, using complex numerical simulations. These simulations are 
idealised models of the formation of a massive star that study what 
happens to interstellar gas that is subjected to a strong gravitational 
pull and photoionising radiation. The study of the dynamical evolution 
of gas is called *hydrodynamics*, and when the effect of radiation on 
hydrodynamics is modelled, we call this *radiation hydrodynamics*. It is 
important to realise that radiation and hydrodynamics are tightly 
coupled: the pressure changes caused by radiation heating cause 
movements in the gas and this in its turn causes changes in the gas 
density that change the way the radiation interacts with the gas.

My paper is part of a series of (currently) three papers that describe 
our models. The first paper in this series was written by my colleague 
Kristin Lund and models the spherically symmetric models that Eric Keto 
came up with. She confirms his original theory and shows that our 
state-of-the-art radiation hydrodynamics method can accurately model 
this scenario. The paper by my Brazilian colleague Nina Sartorio uses 
the same technique (with my simulation code 
[CMacIonize](https://bwvdnbro.github.io/CMacIonize)) to model the more 
realistic scenario where the star forms inside an accretion disc. Her 
results seem to suggest that in this case radiation is not able to shut 
down accretion, provided that the HII region is trapped at the moment 
when the protostar starts emitting ionising radiation. My paper 
(historically the second in the series, but due to the randomness of the 
scientific review process the first to be accepted for publication) 
deals with a theoretical detail of the spherically symmetric case, that 
I will describe in more detail below.

# Steady-state accretion

The accretion onto a massive protostar is a highly dynamic event: gas at 
large distances is pulled in by the gravitational attraction of the 
protostar and accelerates towards the protostar. The gas is hence 
constantly moving. Nevertheless, we can write down a mathematical 
description for this event that does not change over time. To do this, 
we have to make three assumptions: (1) that we have an infinite 
reservoir of gas with a constant density at large distances from the 
accreting protostar, (2) that the properties of the protostar (like its 
mass) do not change when gas is accreted onto it, and (3) that the 
accretion *rate* onto the protostar is constant in time and space. To 
understand the latter, picture a large sphere surrounding the protostar. 
The radius of the sphere should be large enough so that it comfortably 
fits the entire protostar and its immediate surroundings. We will assume 
that the radius of the sphere does not change over time. While gas is 
being accreted, it will cross the surface of the large sphere. We can 
count how much gas crosses the surface during some time interval 
$$\Delta{}t$$, and call this $$\Delta{}M$$. The accretion rate is then 
defined as

$$\dot{M} = \frac{\Delta{}M}{\Delta{}t}.$$

Our assumption hence means that $$\dot{M}$$ is a constant number, both 
when we track the accretion across the surface of the same large sphere 
over time, or when we do the same for a large sphere with a different 
radius. The assumption that the accretion of gas does not alter the mass 
of the protostar is clearly not very realistic, but it is nevertheless a 
good approximation if the accretion rate is low enough: if the mass of 
the protostar is $$20~{\rm{}M}_\odot{}$$ (20 times the mass of our Sun, 
a typical mass for a massive protostar) and the accretion rate is 
$$\dot{M} = 10^{-5}~{\rm{}M}_\odot{}~{\rm{}yr}^{-1}$$ (meaning the star 
accretes a mass equivalent to 0.001% of our Sun every year), then it 
will take 20,000 years before the mass of the protostar changes by 1% of 
its value. In that same time, the gas can cover more than 10,000 times 
the distance between the Earth and the Sun!

When we make these three assumptions, we can find a so called 
*steady-state solution* for the dynamics of the gas. This solution is an 
example of a *dynamic equilibrium*: it is a *stable* solution that does 
not change over time, while at the same time describing gas that is 
*moving*. The steady-state solution for *spherically symmetric* 
accretion onto a massive object is called *Bondi accretion*, since 
Hermann Bondi was the first one to derive this steady-state solution. 
*Spherically symmetric* in this case means that we also assume that the 
gas properties only change when the gas is closer or further away from 
the star (we only allow *radial* changes), and that they are the same 
for gas that is at a constant distance from the star.

When we want to describe a trapped HII region, we need to make 
additional assumptions about how the radiation heats the gas. Leon 
Mestel proposed to assume a *two temperature solution* in which ionised, 
hot gas has a constant temperature $$T_i$$, while neutral, cold gas has 
a constant temperature $$T_n$$ (and $$T_i > T_n$$). He further assumed 
that the transition from ionised to neutral gas (what we call an 
*ionisation front*) is very sharp and happens at a distance $$R_I$$ from 
the star.

Using these assumptions, Leon Mestel and later Eric Keto were able to 
show that a new steady-state solution exists for small enough values of 
$$R_I$$. However, their solution was *numerical*, which means that they 
were able to show that the solution exists, and could write down 
computer code that can generate the solution (e.g. to make images). In 
my paper, we extended this work by actually deriving a mathematical 
equation that describes the two temperature steady-state solution. The 
equation itself is not very elegant (it involves a special function 
called the *Lambert-W function*), but nevertheless is useful, as it 
gives us a good way to compute the gas density and fluid velocity at any 
arbitrary distance from the star. We will use this for our simulations 
later on.

Another issue that neither Leon Mestel nor Eric Keto discussed in their 
original work was the *stability* of the two temperature steady-state 
solution. Recall that steady-state means that the solution is not 
supposed to change over time: the gas density and fluid velocity remain 
the same at any given radius, although the gas itself is constantly 
moving. In a perfect world (or rather, a perfect Universe), this would 
mean that our analytic expression is valid for eternity. Unfortunately 
however, our Universe is not perfect, and real stars will never have a 
perfectly constant accretion rate (one of the assumptions we made 
above). This means that we can realistically expect to have small 
deviations from the constant accretion rate we assume, and hence small 
changes in the two temperature steady-state solution (the solution won't 
be *perfectly* steady-state).

*Stability* analysis deals with what happens when a steady-state 
solution is *perturbed* by small deviations in the assumptions on which 
it is based. There are two possible scenarios, called *unconditionally 
stable* and *conditionally* or *marginally stable* solutions. The 
easiest way to understand these is to think about a landscape with hills 
and valleys in which you place a ball that is free to roll under the 
force of gravity. When the ball is on a slope, gravity will pull it down 
towards the lower end of the slope; this is clearly not a stable 
position for the ball to be in. When the ball is on the top of a hill or 
in the bottom of a valley however, the force of gravity cannot affect 
it, and it will happily stay there; these are stable positions.

When the ball is at the bottom of a valley, and for some reason it is 
pushed out of the valley over a small distance (a small deviation from 
its stable position), it will be pulled back towards the bottom of the 
valley, since that is the direction of gravity. The valley is an 
*unconditionally stable* position: the ball will always roll back to the 
bottom of the valley (as long as it does not move too far away from it). 
For the top of a hill position, things are very different: in this case 
gravity pulls away from the top of the hill, and hence away from what 
was originally a stable position. The top of a hill is a *marginally 
stable* position: any small deviation from it inevitably leads to the 
ball rolling away towards a more stable other position.

We can try to perform a similar stability analysis for the two 
temperature steady-state solution, by introducing a small *perturbation* 
in one of the fluid variables. We chose to do this by introducing a 
small bump in the density at a radius larger than $$R_I$$. This bump 
will move with the local flow of the gas, which is in the direction of 
the protostar, and will eventually cross the ionisation front at radius 
$$R_I$$. When this happens, the density inside the HII region will 
increase compared to what it should be in the two temperature 
steady-state solution, and as a result, the HII region will shrink in 
size. We show this mathematically, but it is not that hard to understand 
why this happens: when the radiation of the star ionises an atom of the 
gas, a tiny fraction of the radiation is *absorbed* and can no longer be 
used to ionise another atom. The size of the HII region is nothing more 
than the volume around the protostar that contains enough gas to absorb 
all the ionising radiation the star emits. If we suddenly add some more 
gas to this volume because of the density perturbation, then the 
radiation of the star will be spent before it can ionise the entire 
volume of the original HII region, and the HII region will shrink in 
size.

The question in stability analysis is not so much what happens when a 
perturbation is introduced, but rather whether the system (in this case 
the size of the HII region) eventually is restored to its original 
state. Or in other words: whether we are on the bottom of a valley or on 
top of a hill. We argue that it has to be the latter: since the density 
increases steeply as the gas gets closer to the protostar, a small 
perturbation in density far away from the star is compressed and grows 
while it is accreted onto the star. This means that the HII region can 
only shrink further, and will not grow again to its original size, as 
long as the original density perturbation is present. Unfortunately 
however, the mathematics to actually rigorously prove this are very 
complicated (you can ask my colleague Nina Sartorio who spent many 
months trying to figure this out). This means that we need numerical 
simulations to prove this. I will tell more about this below.

Additionally, we were also interested in what happens when the original 
density perturbation is accreted onto the star. Or rather, my colleague 
Kristin Lund discovered that something really weird happens (this is why 
we started looking into this in the first place) and we wanted to 
understand what this is. You might expect that when the original 
perturbation disappears, the system would be reset to its original 
state, as if we were putting the ball in our example back on top of the 
hill. We found that this is not the case: while the perturbation is 
being accreted, the HII region shrinks and this causes the density 
outside the now smaller $$R_I$$ to adapt to the Bondi solution for a 
*neutral* gas (which is lower than for an *ionised* gas). When the 
perturbation eventually disappears, the radiation from the protostar is 
suddenly no longer absorbed in the whole volume between its current 
ionisation front and the original ionisation front radius $$R_I$$ and 
ionises this volume again. But it does not stop at the original 
ionisation front radius $$R_I$$ either: the amount of gas in this volume 
is lower than what it should be for the steady-state solution (which has 
a *high density*, *ionised* gas). This means that the ionisation front 
will end up being outside the original ionisation front radius: the HII 
region is larger than at the start.

Unfortunately, this larger HII region is not a steady-state solution, so 
the HII region will eventually shrink again as the density inside the 
newly ionised gas adapts again to the higher density, *ionised* case. We 
show that while this happens, the dynamics will inevitably create new 
density perturbations that will cause the instability of the ionisation 
front. Eventually, we end up in a sort of stable scenario, whereby the 
ionisation front shrinks while density perturbations are accreted, and 
then expands (too far) again when these perturbations reach the central 
star. The HII region is still trapped, but is always evolving.

# Simulations

Since we found it to be impossible to mathematically show that the two 
temperature steady-state solution of Eric Keto is only marginally 
stable, we had to resort to numerical simulations. To this end, I used a 
simple 1D, spherically symmetric [hydro toy 
code](https://github.com/bwvdnbro/HydroCodeSpherical1D) that I wrote 
while preparing lectures about numerical hydrodynamics. The code is 
pretty much a textbook *finite volume solver* that uses a 1D radial grid 
of cells and accounts for spherical symmetry by including some 
correction terms. To perform radiation hydrodynamics, I assumed a very 
basic model for the radiation, that ties in well with Leon Mestel's two 
temperature approach: I assumed a constant ionising luminosity for the 
protostar, and then numerically computed how much volume could be 
ionised at any time by that radiation by accounting for the actual 
hydrodynamical density in each cell of the numerical grid. Inside that 
volume, I artificially multiplied the pressure the gas would have if it 
where neutral with a factor 32 (this is roughly the difference in 
pressure you expect between neutral and ionised gas).

We performed a number of simulations to show (a) how good this approach 
is, and (b) to show that the theoretical scenarios we proposed actually 
occur. We were for example able to show that if we set up the two 
temperature steady-state solution for which we derived a mathematical 
expression, our RHD code was able to keep this stable for a while, 
before numerical perturbations (caused by the finite precision of our 
computers) triggered the instability of the ionisation front. We also 
showed that a neutral accreting gas with a constant accretion rate will 
eventually settle into the two temperature steady-state solution, 
provided that we artificially keep the ionisation front radius $$R_I$$ 
fixed (to prevent the instability from developing).

The most tricky and most important thing however was to show that the 
instability is really catastrophic and cannot be undone once it occurs, 
and leads to the weird scenario that Kristin Lund discovered. To do 
this, we had to show that our numerical simulations were *converged*: we 
had to demonstrate that the results we found are a direct result of the 
physical equations we were solving numerically, and were not caused by 
the numerical code we used. I mentioned before that the finite precision 
of our computers can already trigger the instability: when this happens, 
the simulation results always depend on the numerical code and it is 
impossible to tell whether they are real or not. We had to come up with 
a way to prevent numerically seeded instabilities from dominating our 
result.

Eventually, we had to do two things: first of all, we made sure to 
*seed* the instability. Just as we introduced a density perturbation in 
our stability analysis above, we introduced an actual density 
perturbation in our simulations to manually trigger the instability. If 
this is done early enough (before numerical issues trigger 
instabilities), then the seeded perturbation will dominate the 
simulation, and allow us to study its evolution. By changing the size of 
the perturbation, we could show various scenarios whereby the HII region 
changed shape in different ways (but never restored to its original 
size).

To study the late-time behaviour of the HII region (after the first 
perturbation disappeared), we had to also *soften* the ionisation front 
transition. In the Mestel and Keto models, the ionisation front 
represents a sudden jump from ionised to neutral gas that happens at a 
single radius $$R_I$$. This kind of sudden jumps cannot be represented 
exactly on a computer, which again results in simulation results that 
depend on the details of the simulations. By making sure the ionisation 
transition happens more gradually (and hence *softening* it), we made it 
possible to represent it on a computer. This eventually allowed us to 
show that the Lund scenario is real and not just a numerical issue.

The computationally interesting aspect of the simulations was setting up 
appropriate initial conditions for the simulations with a softened 
ionisation front, since we did not have a mathematical expression for a 
softened two temperature steady-state solution. We instead had to 
*generate* our initial conditions: we ran a separate simulation that 
started out from a neutral accreting gas with a constant accretion rate 
at large distances from the star, and evolved this dynamically until it 
settled into a steady-state solution, by imposing a constant ionisation 
front radius and a constant soft transition size. We then used the final 
state of these simulations as initial conditions for simulations where 
we no longer kept the ionisation front radius fixed.

To keep track of all the different initial conditions and simulations, I 
used a scientific workflow management system (Makeflow), as discussed in 
[a previous post]({% post_url 
2019-02-03-tools-i-use-for-scientific-code-execution 
%}#workflow-management). In this case, the scientific workflow contained 
everything needed to run and analyse the simulations; when I changed 
something to the code (which happened quite a lot of times), I simply 
reran the entire workflow on my computer and all figures that eventually 
ended up in the paper were automatically regenerated. When the reviewer 
asked to include an additional set of simulations, I simply had to copy 
the instructions for some of the existing simulations in the Python 
script that generated my Workflow and rerun the workflow. Makeflow even 
made sure that only those simulations that needed to be rerun were 
actually run again.

# Conclusion

In my paper, we show that spherically symmetric trapped HII regions are 
not static, but are constantly evolving in size and shape. This result 
is somewhat counter-intuitive and not really what we expected when we 
first started modelling these HII regions. Scientifically, it is less 
relevant, as we know that real HII regions are not spherically symmetric 
and show a very different evolution.

Our results are very interesting from a numerical point of view as well. 
Radiation hydrodynamics is an upcoming field in astrophysical modelling, 
and having simple toy models like the ones we discuss in my paper is 
useful to understand future algorithms and techniques that try to model 
real astrophysical scenarios for which we don't have a clear 
mathematical description. The instability Kristin Lund discovered by 
accident made us doubt the accuracy of our methods, and only by 
rigorously investigating what was actually going on do we know we can 
still trust them. The same instability makes it very much impossible to 
model spherically symmetric ionised accretion in 3D simulations: a 
different numerical triggering of the instability in different 
directions triggers the formation of star-shaped HII regions that look 
very different from what we expect theoretically. Now we understand why.

Personally, I really enjoyed working on this paper because it gave me a 
perfect excuse to start using Makeflow. All 1D simulations in the paper 
could be run very easily on my desktop computer at work, so that I did 
not have to deal with any of the network complications that still make 
it very hard for me to use workflow management systems for large-scale 
simulations. On top of that, the large number of simulations and 
reasonable complexity of my workflow made this a very nice example of 
how workflows are useful; I already included it in a presentation about 
workflows I gave last year.
