---
layout: post
title: "The danger of complex models"
description: A rant about the progress of complex astrophysical modelling.
date: 2019-05-05
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
---

I am currently on a tour of astrophysics departments in the South of the 
UK, and have very little time to come up with a good topic for this 
week's post. Hence my choice for a somewhat more controversial and 
possibly more subjective title, and a post that might contain less 
rigorously motivated content that stems from my personal frustrations. I 
still hope it might contain some value at least.

# The context

Last month, I was attending a scientific meeting on a topic I am not 
really too familiar with: modelling of the solar atmosphere (fortunately 
for me, the meeting had other topics that are less alien to me). These 
models are probably among the most relevant astrophysical models out 
there, as a good model for our Sun could help predict the solar weather, 
i.e. the occurrence of strong magnetic flares that can damage satellites 
in their orbit around the Earth and can cause disruptions in radio 
transmissions and radar monitoring in the Earth's atmosphere.

Modelling the solar atmosphere is incredibly difficult, as it is 
dominated by strong magnetic fields that interact with a highly 
turbulent plasma. We have very detailed satellite observations of the 
solar disc that show a huge variety of complex phenomena, and very 
little models that actually agree with these observations.

Yet, within the field of solar atmospheric modelling, there seems to be 
a growing optimism about the state of the field. Where in the past 
proposals for funding would focus on *idealised models of parts of a 
solar like disc*, more and more proposals tend towards a *full model for 
the entire solar atmosphere*. So it looks like we are getting somewhere.

Except that we are not. The meeting I attended was a relatively small 
meeting where most of the attendants knew each other and people were 
generally very friendly towards each other. More like a gathering of old 
friends. In such a meeting, people are less afraid to admit the flaws in 
their work, because there is less of a competition between them. And as 
such, some speakers openly questioned the feeling that the field of 
solar atmospheric modelling is getting somewhere, or openly admitted 
that the models have not significantly improved the last decade, but 
there exists an increasing drive towards making the models sound better 
in grant proposals. So while the field itself is still progressing at 
the same pace as before, there seems to be a growing gap between the 
actual state-of-the-art and the assumed state-of-the-art that creates 
the impression of significant progress.

# A possible explanation

The existence of this gap does not really surprise me. It would be 
unethical to openly lie in a grant proposal, but it is common practice 
to make things sound more positive than they actually are, or to be 
cautious when mentioning known drawbacks of a certain approach. After 
all, you want to convince people that your research with a fundamentally 
unknown outcome has a 100% (or 99.99%) chance of success. And every 
respectable grant application process has at least one peer review step, 
where a respectable scientist with full knowledge of your field has to 
objectively judge your proposal and point out the known flaws that you 
did not mention. Or at least, that is the theory.

The problem, in my eyes, is the decline in quality of this peer review 
step. While each scientist should in principle be willing and able to 
act as a referee for both scientific publications and grant proposals, 
there is very little actual motivation to do this. Peer review is almost 
exclusively done on a voluntary basis and is almost exclusively 
anonymous, i.e. you do not get rewarded for doing it and nobody can see 
how good or bad you do it. So from an ethical point of view, a good 
scientist will still be motivated to perform a good job, but the lack of 
a proper reward will demote the priority of this task within the 
increasingly busy schedule of that scientist. So in the end, peer review 
might be done less thoroughly than desired.

The selection of good reviewers is also non-trivial. Every scientist is 
almost per definition an expert in a very small field of research, so 
the best reviewer for her particular proposal is probably that same 
scientist. Finding the next best thing is hard, as it requires a 
thorough knowledge of the small field of research. The person who has to 
find a reviewer very likely does not have this knowledge. So in the end, 
the reviewer that gets selected will not be the next best thing, but a 
person that has hopefully sufficient knowledge to at least give a 
partial appreciation of the proposal.

Within astrophysics, there is a significant distinction between 
observers (people that actually use telescopes) and theorists (people 
that use pen and paper or computers to make models of astrophysical 
phenomena). Both a theorist and an observer might be active within the 
same small field of research, but very little people are both a theorist 
and observer combined. A theorist is usually perfectly capable of 
judging the astrophysical merit of an observational proposal and vice 
versa, but very rarely capable of judging the technical aspects of the 
proposal. So if by chance a solar observer gets to judge the scientific 
merits of modelling the entire solar atmosphere, that person will 
probably conclude this is a very good idea. But that same person might 
not be aware of the technical difficulties that poses.

So my view is that there is an increasing gap between the technical 
knowledge of observers and theorists that drives an increasing gap 
between actual state-of-the-art modelling and assumed state-of-the-art 
modelling. This view is partially based on my own experience: when I 
applied for computation time at the end of last year, one of the 
reviewers of this proposal questioned the efficiency of my 
state-of-the-art RHD code, claiming that the use of a fixed regular grid 
structure is outdated. While I can understand the origin of this claim 
(the use of adaptive grid structures is widespread in hydrodynamic 
modelling), this is technically not correct, as efficient RHD methods of 
the same kind that use adaptive grids do simply not exist.

# Towards a breaking point?

The problem with this increasing gap in the assumed state-of-the-art and 
the actual state-of-the-art is that it works incremental: once a 
proposal is granted based on a slightly exaggerated claim, the 
impression might be generated that this exaggerated claim reflects 
reality. A future proposal that makes exactly the same claim will no 
longer be innovative enough to be funded, urging the applicant of the 
proposal to exaggerate a bit more.

In my own field of research, it is common practice to include more and 
more physical effects within one simulation. It is very easy to generate 
the impression that a specific effect works without actually lying: you 
can set up a dedicated test that singles out that effect and shows it 
does what it should do. It is much harder to test that the effect actual 
works within the full combined simulation. Some physical effects are 
additive and combining them is simply a matter of turning the relevant 
switches on at the start of the simulation. Other effects will interfere 
with one another in subtle ways and it is very hard to disentangle them 
and check that the combined effect is actually physically correct.

So a typical grant proposal will say something like *we have this model 
that includes A, B and C, and in this project we will add D*. In 
reality, A and B might have been added in 1512 A.D. by PhD student 10001 
who later left academia. C was added 100 years later by PhD student 20041.
That student discovers that C works when combined with A, but 
this requires a small change in the parameters for A. This small change 
of parameters turns out to dramatically change the combination of A with 
B, but since student 10001 is no longer there, nobody notices this. So 
now A and C work together, and A and B are assumed to do so, and from 
this the assumption is made that A, B and C work together. While in 
reality student 20041 only showed that A and C work together.

As more and more effects are combined, more and more layers of 
complexity and possible interferences are created. Properly checking all 
of these would ideally require the presence of all researchers that were 
involved in building up the complex model, which unfortunately is not 
how academia works. The next best thing would be to have increasingly 
more manpower to add additional physical effects (as there is more work 
required), which unfortunately is also not how academia works. So in the 
end, the gap between reality and assumed state-of-the-art will continue 
to grow, until even theorists themselves no longer realise what the 
actual state-of-the-art is.

Such a growing discrepancy has to eventually lead to a breaking point, 
when the gap between reality and assumption becomes too large to not be 
obvious. It is my feeling that this breaking point has been reached for 
solar atmosphere modelling, and I think it will happen for my field as 
well. What will happen after that is very unclear. It might lead to a 
reset of the assumed model towards the actual model, after which the 
whole cycle is repeated. Or it might force us to acknowledge that peer 
review in its current form is fundamentally flawed and urge the 
astrophysical community to do something about this. Since I am a 
pessimist, I expect the former to happen. But we will see.
