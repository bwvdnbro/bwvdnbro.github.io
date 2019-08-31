---
layout: post
title: "The St Andrews Monte Carlo summer school"
description: >-
  A brief summary of my impressions of the St Andrews Monte Carlo summer
  school 2019
date: 2019-08-31
author: Bert Vandenbroucke
tags: 
  - Tools
  - Astronomy
---

The [St Andrews Monte Carlo Summer 
School](http://www-star.st-and.ac.uk/samcss/) or *SAMCSS* is a biyearly 
summer school that is primarily aimed at PhD students based in the UK, 
and has been organised since 2013. I have been involved in its 
organisation for both the 2017 and the 2019 edition, the latter of which 
was held this week.

The original topic of SAMCSS have been the use of Monte Carlo methods to 
study radiative transfer in astronomical problems. Since the community 
at large seems to have a current interest in combining all kinds of 
radiative transfer with hydrodynamics to perform radiation hydrodynamics 
(RHD), the 2017 and especially the 2019 edition expanded on this 
original topic by incorporating more hydrodynamics and lectures on how 
to couple both techniques. Below is a very personally biased summary of 
the summer school and its main topics.

# Monday: MCRT and hydro

Since the two main topics of the summer school were Monte Carlo 
radiative transfer (MCRT) and its combination with hydrodynamics, it 
made sense to devote the first day to a low-level introduction to both 
MCRT and numerical hydrodynamics. The former was introduced by Kenny 
Wood, the main organiser of the school, and was a reiteration of the 
introductory lecture he gave for previous editions of the school. Next 
up was a lecture by school alumnus Antonia Bevan; she gave a very down 
to Earth overview of what she had to do to develop her own MCRT code 
based on what she learned during the summer school she attended.

Having worked very intensively with Kenny over the past three years on 
the development of my own MCRT code, these lectures did not really 
provide me with any shocking new insights, but that couldn't really be 
expected either. I think my main lesson of the day was something very 
practical: Antonia mentioned how it is a really good idea to make the 
variables you use in loop counters (typically `i`, `j` and `k`) two 
characters instead of a single character (so `ii`, `jj` and `kk`), so 
that they become searchable. This is a very simple trick, but I think 
this could have saved me quite a lot of time in the past. I had seen 
this kind of loop counters before, but it had never occurred to me there 
was actually a good reason to do this.

The afternoon was entirely devoted to an introduction to numerical 
hydrodynamics that I lectured myself, so I will not comment any further 
on it.

# Tuesday: RHD

Having spent the first day learning all about MCRT and hydrodynamics, 
the second day provided the logical continuation: the combination of 
both for RHD. We started the day with a talk from Tim Harries about this 
very topic, although he did spent the first half of his lecture talking 
about MCRT optimisation techniques as well (you want to make sure you 
can do the radiative transfer very efficiently before even considering 
coupling it to a real-time hydrodynamics scheme). This lecture contained 
some techniques I hadn't encountered before (the modified random walk 
for example). But it most of all contained a lot of very clear images 
that showed the result of various optimisation techniques. So I think 
the most important lesson there was to use clear images when you are 
trying to make a point.

The morning session was closed by Stuart Sim, who introduced the notion 
of performing MCRT simulations in a highly dynamic, time-dependent 
environment: the study of supernova explosions. Again, the things I 
remember most of this lecture might be less relevant to the topic of the 
school, as they are more about the use case of these simulations. Stuart 
mentioned that most of the light from a supernova that we see is not 
actually created by the supernova explosion itself: it is partially due 
to the thermal afterglow of the explosion from the hot gas and other 
factors, but is primarily caused by the decay of highly unstable 
radioactive elements that are excited during the explosion. I guess this 
is something I simply didn't know.

The afternoon session was all about Lyman $$\alpha{}$$ (Ly$$\alpha{}$$) 
MCRT. Ly$$\alpha{}$$ is a particular frequency of light that gets 
emitted when an electron in a hydrogen atom falls back from its first 
excited state into the ground state. Aaron Smith explained in great 
detail how this light has great difficulty to actually escape the gas 
clouds from which it is emitted, as this light (obviously) has the exact 
right frequency to excite an electron from the ground state of a 
hydrogen atom to its first excited state, so that it is absorbed again. 
Ly$$\alpha{}$$ can only escape by (very) slowly shifting in frequency 
due to all kinds of small random Doppler shifts that happen within a real 
turbulent gas cloud. Modelling this slow escape process is extremely 
challenging, and Aaron discussed a lot of optimisation techniques that 
are required to model it nonetheless.

As a small aside: the Balmer $$\alpha{}$$ (H$$\alpha{}$$) line is 
emitted when an electron falls back from the second excited state to the 
first excited state of the hydrogen atom. Unlike Ly$$\alpha{}$$, this 
line has not much trouble escaping a gas cloud, since it is reasonably 
unlikely that another atom in the gas cloud will have an electron in its 
first excited state that can absorb this light to jump to the second 
excited state. This is why modelling H$$\alpha{}$$ is much easier; this 
is something I have done in the past.

# Wednesday: more codes and processes

After the introductory lectures of the first two days, the third day 
featured some more advanced topics. Kees Dullemond gave an overview of 
his widely used RADMC-3D code, from which I mainly remember that 
choosing a somewhat obscure name for a code (RADMC-RD, CMacIonize) 
instead of a nice sounding acronym (like Shadowfax, Gadget, SWIFT...) is 
actually a good idea, as it makes it a lot easier to find your code 
through a search engine. Again, a good point that I had not realised 
before.

Tom Haworth gave a very accessible introduction to the ALMA 
interferometer, and explained how to make model images of ALMA 
observations to support applications for telescope time. Nothing I 
particularly fancied, but definitely a good lecture.

In the afternoon, Stuart Sim introduced the notion of *macro atoms* to 
deal with atomic transitions in gas clouds. Some applications of MCRT 
try to model the spectrum that escapes from radiatively excited gas, and 
these applications need to model the atomic processes responsible for 
emitting and absorbing specific spectral lines. An important aspect of 
these models is energy conservation: you want to make sure that all the 
light that is emitted by the cloud eventually escapes from it; atoms are 
not exactly very good at storing energy.

The issue with this is that light that gets absorbed at one frequency 
might actually be reemitted in stages at different frequencies: light 
can excite an electron from the ground state of the hydrogen atom into 
its second state. The electron will then emit an H$$\alpha{}$$ photon to 
fall back to the first excited state, and only then will it fall back to 
the original ground state by emitting another Ly$$\alpha{}$$ photon.

One way of tracking this would be to actually split photons within the 
MCRT algorithm, but this becomes intractable very quickly. The macro 
atom formalism offers an alternative method by modelling the excitation 
and decay process for a single atom as a traffic flow problem, where 
various transition channels have probabilities based on the underlying 
atomic properties (that can be measured). For every absorption event, a 
single ingoing photon will set off a random walk through the subsequent 
excitation and deexcitation process using these probabilities, until a 
new photon is emitted. By modelling the traffic flow within the atom 
itself using a Monte Carlo technique, you can guarantee energy 
conservation, even though the frequencies of ingoing and outgoing 
photons are not necessarily the same.

The last lecture of the day was given by Kenny Wood and dealt with Monte 
Carlo photoionization. Since this is exactly what I have been working on 
for the past three years, there was nothing new there for me.

# Thursday: parallelisation and confidence

This last day of the summer school did not really introduce any new 
science topics, but instead dealt with some more practical and personal 
aspects of numerical science, as well as an interdisciplinary 
application of MCRT. During the morning session, Tim Harries introduced 
OpenMP and MPI as main techniques to parallelise a numerical algorithm, 
after which I explained why parallelisation is so important and why we 
should aim to develop our codes in a more inherently parallel way, by 
changing the way we think about algorithms. After this, Lewis McMillan 
gave a nice overview of the work he did using MCRT to model light 
propagation and tissue damage in skin.

In the afternoon, Antonia Bevan gave a very personal (and brave) talk 
about the struggles of academia and how the very competitive nature of 
academia can very easily cause confidence issues (the so called 
*imposter syndrome*). She explained what imposter syndrome is, how it 
manifests itself and affects you (and your work), and what you can do to 
help yourself and others to cope with it. She then also touched upon the 
diversity challenges science faces, since science is still dominated by 
white, middle-class males. This led to a very interesting open 
discussion in which various points were made about gender biases in 
academia and how we can work to improve the gender balance within 
numerical astrophysics (*why do so little girls code?* as Kenny would 
put it). Antonia made a very good point during the discussion about how 
it is up to white, middle-class males to actively counteract any 
unconscious biases that still exist, as it is mainly these biases that 
make it hard for any minority to feel at ease within the scientific 
community. For me she managed to drive home the message that any kind of 
stereotyping is really damaging, even if it is by no means intended to 
be harmful. And she convinced me that I should try to be more aware of 
these kinds of stereotyping and try to actively speak out against them.

# Summary

Although the scientific contents of the summer school was not really 
relevant to me personally (as I was already very familiar with the 
relevant topics), I still think this was a very interesting summer 
school. There was a very good interaction between the participants, and 
especially Antonia Bevan's final lecture made it very easy for me to 
interact with some of the participants on a more personal level. I saw a 
lot of very motivated and nice people that were willing to look at the 
future with a very open mind. This does give some hope that academia is 
changing for the better, although we still have a long way to go.

Scientifically, I had many interesting discussions with Aaron Smith, and 
for the first time really managed to discuss my new task-based MCRT 
algorithm with him and Tim Harries. I hope to have some time in the near 
future to write things up and get this algorithm out there; it is not 
really ready, but is innovative enough as it is to be very helpful for 
people in the field.
