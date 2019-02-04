---
layout: post
title: "Bondi paper (working title)"
#date: 2019-02-03
description: None
image: /assets/images/space_filling_curves.png
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Workflow management
  - Astronomy
---

Last week, a scientific paper that has been in my scientific pipeline 
for over a year got accepted for publication in Monthly Notices of the 
Royal Astronomical Society, and this seems a good opportunity to explain 
what this paper is about. It also allows me to tell you some more about 
the machinery behind the paper, which made extensive use of workflow 
management systems.

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
from a cold gas with temperatues around $$100-500$$ K into a hot gas 
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
a trapped HII region. Even though the star heats the surrouding gas, the 
accretion onto the star can still continue.

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
realistic scenario where the star forms inside an accretion disc. My 
paper (historically the second in the series, but due to the randomness 
of the scientific review process the first to be accepted for 
publication) deals with a theoretical detail of the spherically 
symmetric case, that I will describe in more detail below.

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
example of a *dynamic equilibrium*: it is a *stable* solution that does not
change over time, while at the same time being 
