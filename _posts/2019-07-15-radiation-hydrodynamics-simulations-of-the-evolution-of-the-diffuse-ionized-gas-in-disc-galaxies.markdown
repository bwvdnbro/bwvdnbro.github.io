---
layout: post
title: >-
  Radiation hydrodynamics simulations of the evolution of the diffuse 
  ionized gas in disc galaxies
description: >-
  A short overview of a scientific paper that has been accepted for publication.
date: 2019-07-15
author: Bert Vandenbroucke
tags:
  - Scientific computing
  - Astronomy
---

[A paper](https://doi.org/10.1093/mnras/stz1841) describing some of my 
recent work has recently been accepted for publication 
([pre-print](https://arxiv.org/abs/1907.02067)), and this seems like a 
good opportunity to tell you something more about it.

The paper summarises the work I have been doing in the last year to try 
to understand aspects of the vertical structure of the interstellar gas 
in star-forming disc galaxies like our own Milky Way. I have been able 
to show that some of the observed characteristics of the vertical 
interstellar gas correlate well with the dynamical impact of radiation 
of young stars in the galaxy, which gives us some insight into what 
processes are important to drive the vertical structure of the 
interstellar gas. For someone not familiar with the work, this probably 
sounds like a lot of fancy words with no meaning, so let's try to 
clarify this a bit...

# Star-forming disc galaxies

Galaxies are the largest known building blocks of the Universe. They 
consist of many millions to hundreds of billion stars that are 
gravitationally bound and orbit a common centre of mass, and also contain 
most of the cold interstellar gas from which new stars are born, as well 
as interstellar dust, black holes and of course planets. The observable 
Universe contains a hundred billion galaxies, ranging from massive 
galaxies with a diameter of more than 100,000 light years that contain 
10 trillion times the mass of our Sun, to small dwarf galaxies 
containing only a few million stars that are barely visible for even the 
most powerful telescopes.

Galaxies are very important for our understanding of astrophysics, as 
most of the interesting events in astrophysics are at least to some 
extent caused or influenced by the galactic environment in which they 
happen: stars will form in dense regions that are created by 
galaxy-scale movements of interstellar gas, and interstellar dust and 
planets form out of interstellar gas that was enriched with material 
synthesised in old stars that exploded in other parts of the galaxy.

Galaxy evolution is governed by a very complex interplay between gas 
dynamics, star formation and stellar feedback (e.g. the energy transfer 
from stars to interstellar gas when the latter absorbs stellar 
radiation, or the energy released into the interstellar medium by old 
stars that explode in big supernova explosions) and the very complex 
atomic processes that cause the interstellar gas to cool down. As such, 
we can only model it using complex numerical simulations that try to 
capture these effects as accurately as possible. Current models are 
nowhere near providing a full model that explains the whole of galaxy 
evolution, but we do understand some aspects.

First of all, most star formation nowadays takes place in star-forming 
*disc galaxies*, like e.g. our own Milky Way or our closest large 
neighbour, Andromeda. These galaxies consist of a central *bulge* that 
looks like a fuzzy sphere and that is embedded in a much larger *disc* 
that is approximately circular and is really flat and thin in one 
direction. Within this disc, more complex structures like spiral arms or 
bars are usually present. Almost all of the stars that form in a disc 
galaxy form in the thin disc, and when spiral arms are present, star 
formation regions are usually found close to or in the spiral arms.

# The vertical structure of a galactic disc

Most of the interstellar gas and a significant part of the stars in a 
disc galaxy are located in the thin disc, and it is clear that to 
understand star formation within these galaxies, we need to understand 
the interplay between gas dynamics and stellar feedback within this thin 
disc. A lot of efforts to understand these processes hence focus on this 
aspect. However, we cannot ignore the vertical direction perpendicular 
to the disc, as stellar feedback can expel interstellar gas from the 
thin disc and drive it to high altitudes above the disc, where it can 
eventually cool down and fall back into the disc, providing fuel for 
more star formation. Since our own Earth (and hence all of our 
telescopes) are located within the thin disc of our Milky Way, it is 
also a lot easier to observe this part of the Milky Way, as most of the 
thin disc is at least to some extent blocked by gas and dust in the thin 
disc.

From observations of our own Milky Way disc, we know that the 
interstellar gas in the vertical direction has three main components at 
different altitudes above the disc: there is a very thin cold disc that 
roughly coincides with the thin stellar disc and that mainly consists of 
atomic or molecular hydrogen gas with very high densities. Then there is 
a *warm* disc that is significantly more extended and that consists of 
an ionized hydrogen *plasma*: a mixture of positively charged protons 
(the nuclei of hydrogen atoms) and negatively charged electrons with a 
temperature of close to 10,000 degrees Kelvin and at significantly lower 
density. Finally, there is the much more extended *hot halo* that 
surrounds the entire galaxy and that consists of very low density plasma 
with a very high temperature (approximately 1,000,000 degrees Kelvin).

The presence of the first and last of these layers are not surprising: 
we know that cold molecular gas is required to reach the very high 
densities necessary to form stars, so we would expect this gas to be 
present in regions of star formation. We also know that the supernova 
explosions that occur when old stars die very effectively heat the 
surrounding interstellar gas to temperatures above 1,000,000 degrees 
Kelvin, and that gas that reaches these temperatures will cool down very 
slowly. What is more puzzling is the presence of the second layer of 
warm, ionized gas: cooling processes in this layer should be efficient 
enough to cool down this gas to lower temperatures and this should cause 
the protons and electrons in the plasma to *recombine* into atomic 
hydrogen. The fact that we do observe this layer means that some 
physical effect is actively heating up this gas and keeping it in its 
plasma state.

The questions we set out to answer is twofold:
 1. which process is responsible for heating up this warm ionized layer (also
known as the extended *diffuse ionized gas* or DIG), and
 2. what is the dynamical impact of this heating process on the vertical 
structure of the galactic disc?

# Young massive stars

The first question (which process is responsible for heating the DIG) 
has been the topic of my current boss' (Kenneth Wood) research for more 
than two decades already, and we are quite confident that we know the 
answer to it. From energetic arguments, it is easy to narrow down the 
list of possible heating mechanisms. Heating and ionizing interstellar 
gas requires a lot of energy, and only a small number of heating 
mechanisms actually have enough energy to be likely candidates. External 
heating mechanisms that depend on some source of energy outside the 
galaxy itself (e.g. external radiation fields) are nowhere near powerful 
enough to explain the DIG, nor are dynamic effects like shock heating 
caused by internal or external gas motions. In the end, the only likely 
mechanisms left all depend on internal energy sources that somehow are 
linked to the presence of stars and their radiation.

To heat and ionize gas with stellar radiation, you need highly energetic 
radiation at ultraviolet (UV) wavelengths. This is the same type of 
radiation responsible for sun burn, but with three to four times more 
energy. When this radiation is absorbed by a hydrogen atom, it can 
transfer enough energy to the atom to unbind the electron and kick it 
away. The unbinding ionizes the atom, while the additional kick is 
responsible for an increase of the motion energy in the plasma that 
causes its temperature to go up.

All stars emit some of their radiation in the UV, but the amount of UV 
radiation changes dramatically depending on the mass of the star. Stars 
like our Sun emit most of their radiation at less energetic wavelengths 
(the wavelengths that are visible to us), and only a very small fraction 
in the UV. The total energy they emit in the UV is completely negligible 
and nowhere near enough to ionize the DIG. However, very massive stars 
(with masses of 20 times that of our Sun or more) emit mainly in the UV, 
and are very efficient ionization sources. However, since the average 
life time of a star also depends strongly on its mass, these stars have 
very short life times, which for a star means they only live between a 
few million to maybe 20 million years. In stellar terms and galactic 
terms, this is really short, so that these are *young* stars.

Apart from these young, massive stars, there could also be a significant 
contribution from less massive old stellar remnants, like e.g. white 
dwarfs. These sources are a lot less luminous, but also emit most of 
their radiation in the UV part of the spectrum. And since they are a lot 
more common (and longer lived) than massive stars, their total 
contribution to the ionizing energy budget is only a bit weaker than 
that for young massive stars, especially in galaxies with a low level of 
active star formation. We are personally not convinced these stars are 
very important for heating the DIG, but the anonymous referee for our 
paper thought it was important to look at their contribution too.

For most of our work, we have focused on the contribution of the young 
massive stars. These stars will be located close to the location where 
they formed: in the dense, thin galactic disc. The question we need to 
address then is: how does radiation from these stars make it out into 
the interstellar gas at relatively high altitudes above the disc?

The answer is both simple and complex: to reach these altitudes, the 
ionizing radiation needs low density *channels*, i.e. regions within 
both the thin disc and the more extended warm disc where the 
interstellar gas density is low so that UV radiation can travel 
relatively far before being absorbed by the interstellar gas. We know 
that the interstellar medium is highly *turbulent*, which means that 
most of the gas in the interstellar medium is located in dense 
filamentary and clumpy structures, with large regions in between that 
have significantly lower densities. The low density channels are hence 
definitely there. The complexity comes from the fact that modelling 
these channels can only be done in 3D, and requires a good background 
model for a turbulent interstellar gas disc.

These full 3D models of an interstellar disc have only been possible for 
the last 5 years, using a technique called Monte Carlo radiation 
transfer. Initially, this work used existing models of a galactic disc 
and post-processed these with the radiation from a realistic 
distribution of young, massive stellar UV sources. This is the approach 
I used in my [2018 paper](https://doi.org/10.1093/mnras/sty554) 
([pre-print](https://arxiv.org/abs/1802.07749)), and that was similar to 
earlier work of Jo Barnes, one of Kenneth Wood's PhD students.

This work showed that a sufficiently turbulent interstellar gas model 
does produce a sufficient number of low density channels for UV 
radiation to escape out of the thin disc; young massive stars can 
definitely explain the ionization of the DIG.

# Heating of the DIG

To check that young massive stars can also explain the heating of the 
DIG, we need to look at more detailed observations of the temperature of 
this gas. This is done using *emission line ratios*. The warm ionized 
gas contains small traces of heavy elements (like oxygen, sulphur, 
nitrogen and neon) in various atomic and ionic states, and when these 
occasionally interact with one of the many free electrons in the plasma, 
this can give rise to emission of light at very specific wave lengths, 
so called *emission lines*. Which emission lines emit strongly depends a 
lot on the composition, density and temperature of the gas. The 
composition and density dictate how much of a specific element is 
present; the more there is, the stronger the emission will be. The 
temperature on the other hand dictates in what ionic state the element 
will be found, as well as what the relative strength of various emission 
lines for the same element will be.

If we assume a constant composition for the gas, then we can get some 
idea of the temperature structure of the gas by simply dividing the 
strength of an emission line for one element by that for another one, 
and for different locations in the gas. If this ratio changes between 
locations, then we know that the temperature is different between those 
locations, and we can get some idea of which location is hotter.

When we apply this technique to the DIG in our own Milky Way, there is a 
clear trend: the line ratios for various elements all tend to increase 
towards higher temperatures for gas at higher altitudes above the disc. 
This tells us that the heating in the vertical disc becomes more 
effective at higher altitudes. Recent observations of other galaxies 
seem to confirm this observation for other disc galaxies.

This is very hard to explain from our models. When we use the same 
models that nicely explain the ionization of the DIG and look at their 
predicted temperature structure and line emission ratios, then most 
models fail to reproduce this trend: the temperature throughout the DIG 
is fairly constant instead of increasing with altitude. To reproduce the 
observed trend, we have to play around with the total ionizing 
luminosity budget of the young massive stars. If this budget is *low* 
enough, then suddenly the temperature structure looks a lot more like 
what is observed. However, if the budget is too low, then the radiation 
no longer escapes the thin disc, and we no longer have a DIG layer.

This counter-intuitive result can be explained using subtle spectral 
effects. UV absorption throughout the interstellar medium is a 
complicated process, that has a small but noticeable dependence on the 
energy of the UV radiation itself. Very high energy radiation is 
actually less efficient at ionizing atoms than radiation that has an 
energy only slightly higher than the required energy to unbind the 
electron, so that low energy radiation on average is more likely to be 
absorbed. When a significant fraction of the radiation from a source has 
already been absorbed (at high altitudes in the DIG), there will hence 
be a relative excess of high energy photons that reach that altitude. 
Since the heating is effectively caused by the excess energy of the 
absorbed radiation, this remaining radiation will on average cause more 
heating and hence a higher gas temperature. Of course, this effect will 
only be noticeable if a large enough fraction of the original source 
radiation is absorbed, or if the total ionizing energy budget of the 
young massive stars is just enough to ionize out the DIG, but not much 
more. This effect is called *spectral hardening*.

# Radiation hydrodynamics

The fine-tuning we needed to perform to both ionize the DIG and have 
effective spectral hardening to get a realistic DIG temperature 
structure was a bit of a surprise to us, as there was a priori no reason 
why our interstellar gas models (that we just got from other groups) 
would require a specific ionizing energy budget. On the other hand, this 
would be something we would have expected if ionizing radiation was 
somehow partially responsible for creating this interstellar gas model 
dynamically.

This is something that we could expect to happen. We know that the DIG 
is heated and has a higher temperature than the cold, neutral gas in the 
thin disc. This temperature difference will cause a difference in 
pressure between the cold gas and the warm gas, and this pressure 
difference will cause a net force that drives dynamics (think for 
example of what happens to the lid of a cooking pan when the temperature 
inside increases enough). To study the dynamic impact of this increase 
in temperature, we need to couple the heating from our ionization model 
to an actual hydrodynamic solver that can model the dynamic evolution 
of the interstellar gas.

And this is exactly what we did for this paper. Instead of using 
interstellar gas models provided by other groups, we produced our own 
interstellar gas model using just a few physical ingredients:
 - the gravitational force from the stars in the galaxy
 - the ionization and heating from a time-dependent distribution of 
young massive stars

The results are illustrated very nicely in the movie below:

<iframe width="560" height="315" 
src="https://www.youtube.com/embed/JKvKDqrmKeg" frameborder="0" 
allowfullscreen></iframe>

Initially, the warm bubbles surrounding the UV sources expand and expel 
material out of the thin disc, at the same time creating large holes and 
denser filaments in the thin disc. When young stars die and disappear, 
these holes will close again, and new holes will appear at the locations 
where new stars are born. After a while (approximately 200 million years 
of evolution), the system reaches a dynamic equilibrium, in which part 
of the gas ends up in an extended layer of warm, ionized gas, and 
another part remains in the cold, neutral, thin disc.

The movie shows three simulations that are identical in every aspect 
except one: the ionizing luminosity budget assigned to each individual 
source. This budget increases by a factor of 10 in every column, moving 
from left to right. The rightmost column has the highest ionizing energy 
budget, and it looks dramatically different from the other two 
simulations: all of the gas in this simulation is highly ionized for the 
entire duration of the simulation, and there simply is no neutral thin 
disc. The other two simulations are more similar, but there are also 
clear differences: the simulation with the higher ionizing energy budget 
has larger holes and overall more ionized and less neutral gas.

Interestingly enough, when we now post-process these simulations using 
the same radiation that was also used for the dynamic model, we very 
consistently get a temperature structure that is similar to what is 
observed. It looks as if the dynamic impact of the ionizing radiation 
ensures that roughly the right amount of gas ends up in the DIG to get 
effective spectral hardening.

However, when we look at where the DIG in our simulations is located, we 
can also see the shortcomings of our current models. By not including 
the dynamic impact of supernova explosions, we significantly 
underestimate the turbulence in our model, and at the same time have 
less outflows of material out of the thin disc, which leads to a 
consistent underestimation of the altitude of the DIG.

# Conclusion

From the work we had been doing before, it was clear that the vertical 
structure of the discs of star-forming disc galaxies is very sensitive 
to the ionizing energy budget of young massive stars in the galactic 
disc, but we were unable to explain why.

With our new models, we for the first time provide a consistent 
explanation of both the presence and ionization state of the warm, 
ionized gas in these discs, and the observed temperature structure 
within this layer: the dynamic impact of the ionizing radiation sets the 
amount of gas that ends up in the DIG, and this correlation between the 
amount of gas and the amount of radiation naturally gives rise to 
effective spectral hardening.

We do need to stress however that these are still very basic models, and 
that a full model of the DIG that can also explain the observed altitude 
of the DIG will require more sophisticated models that also include the 
effect of supernova explosions, and perhaps other feedback effects. More 
to come!
