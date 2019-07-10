---
layout: post
title: "The many ways to combine plots in Python"
description: >-
  A discussion on (some of?) the many ways to make multiple combined plots in
  Python.
date: 2019-07-10
author: Bert Vandenbroucke
tags:
  - Tools
---

Making plots is one of the core occupations of many astronomers, and 
probably of many other scientists too. These plots are used throughout 
the various stages of actual research, from visualising data for 
personal interpretation and to guide further analysis, to making 
high quality graphics to include in scientific publications that will 
convey your findings to fellow scientists.

Despite, or just because of this, there is a large variety of tools that 
can be used to generate scientific plots. These range from powerful, 
dedicated visualisation toolkits like [VisIt](https://visit.llnl.gov/) 
or [Paraview](https://www.paraview.org/), to the plotting capabilities 
that are built in to other tools, like 
[Matlab](https://en.wikipedia.org/wiki/MATLAB) or (god forbid) 
[LibreOffice](https://www.libreoffice.org/). While all of these can in 
principle be used to generate high quality graphics, some of them have 
clear advantages that make this task a lot easier.

Since a large fraction of scientific plots rely heavily on the outcome 
of a previous analysis of raw data, analysis tools with built in 
plotting capabilities form a natural choice when choosing between 
plotting tools. Full 3D analysis toolkits like VisIt or Paraview are 
very good at visualising complex multidimensional data for personal 
interpretation, but they are harder to manipulate to perform a more 
complex analysis, or to produce a high quality output image. This 
explains the popularity of tools like IDL (proprietary), Matlab 
(proprietary) or Python + Matplotlib (free software; my favourite) as 
default plotting tool for astrophysical research.

In this post, I will not go into further detail and explain why I think 
Python + Matplotlib is a much better choice than the two other options I 
mentioned. Instead, I will cover some of the more tedious aspects of 
making plots in Matplotlib: combining multiple plots into one image. 
These will highlight some of my personal history with the subject, and 
will hopefully show that it can be easy to generate complex images with 
subplots in Python.

# The many ways to make a single plot

The popularity of Python within the community at large is usually a very 
good thing: there are a huge amount of additional Python modules that 
contain high quality functionality that can be very easily installed, 
there is a huge online community that can help out with almost any 
problem you will encounter... On the flip side however, it also means 
that Python is incredibly flexible. A bit too flexible sometimes.

Making plots in Matplotlib is a very good example of this 
over-flexibility. When I make a plot, I usually load the `pylab` module, 
which is part of `matplotlib`, and proceed as follows:

```
import pylab as pl

pl.plot(xvalues, yvalues, "k.")
pl.show()
```

This does exactly what it should do. However, exactly the same thing can 
also be done with the following code:

```
import matplotlib.pyplot as plt

plt.plot(xvalues, yvalues)
plt.show()
```

This is what the [matplotlib plotting 
tutorial](https://matplotlib.org/users/pyplot_tutorial.html) will tell 
you to do. This is not too different. Now consider the following 
example:

```
import pylab as pl

fig, ax = pl.subplots(1, 1)
ax.plot(xvalues, yvalues)
pl.show()
```

In this case, we no longer let a module do the plotting, but instead we 
first create a `Figure` and an `Axes` object using the `pylab.subplots` 
function. We then invoke the `plot` function that is part of the `Axes` 
class to do the plotting. The result is still exactly the same.

The reason the result in these three cases is exactly the same, is that 
in the end, all three versions will eventually call the `Axes.plot` 
function to make the actual plot. They just do this at different levels 
in the object structure. `pylab` is not an actual module, but is a 
*namespace* that combines the `matplotlib.pyplot` and `numpy` modules 
into a single pseudo module for convenience. `pyplot` is the actual 
plotting module in `matplotlib`. By default, this module will create an 
image with a single plot represented by an `Axes` object. The 
`pyplot.plot` function will call the `Axes.plot` function on this 
object.

There are more ways to achieve the same thing, but I think this example 
shows my point: since a lot of images (especially the ones not meant for 
publication) contain a single plot, `pyplot` contains a sensible default 
that allows you to treat the entire module as if it was just this. This 
hides the underlying class structure of the `matplotlib` library, and 
makes it harder to understand what is happening when you want to produce 
images that do not contain a single plot.

# Multiple plots

This brings us to the main topic of this post: combining multiple plots 
into a single image. The [matplotlib tutorial on the 
subject](https://matplotlib.org/gallery/subplots_axes_and_figures/subplot.html) 
will advocate the following technique:

```
import matplotlib.pyplot as plt

plt.subplot(2, 1, 1)
plt.plot(xvalues, yvalues)

plt.subplot(2, 1, 2)
plt.plot(xvalues, yvalues)

plt.show()
```

This will create 2 rows of plots in a single column (`plt.subplot(2, 1, 
X)`), and will in turn act on the first and second plot (`X`). Again, 
this technique is fairly straightforward, but there are some immediate 
problems with it. First of all, in this case `matplotlib` breaks with 
the important convention that Python starts counting from zero, which I 
personally find totally unacceptable. Second, and arguably more 
important, it only allows you to access one subplot at a time. Each call 
to `plt.subplot` will *create* a new `Axes` object within the default 
`Figure`, and will make this the active `Axes` object to which all 
subsequent `plt.plot` commands (and other commands for that matter) 
apply. You can only change the active subplot by a new call to 
`plt.subplot`. Once this has been done, there is no (easy) way to make 
additional changes to the first subplot; calling `plt.subplot` again 
with the same arguments as before will *delete* the first subplot!

So in this case, hiding the underlying class structure leads to some 
important restrictions on what you can do. There are many easy solutions 
to this. The most straightforward solution is to use the return value of 
the `plt.subplot` function, which happens to be the newly created `Axes` 
object:

```
import matplotlib.pyplot as plt

ax0 = plt.subplot(2, 1, 1)
ax1 = plt.subplot(2, 1, 2)

ax0.plot(xvalues, yvalues)
ax1.plot(xvalues, yvalues)

plt.show()
```

We now first create the `Axes` objects, make sure we can access them 
independently, and then call their `plot` function directly to make the 
desired plots. If we want to make changes to any of the axes after this, 
we simply use the corresponding variable. No more issues!

Since many combined images contain very straightforward subplot layouts, 
there is a convenient function that can create all subplots at once:

```
import matplotlib.pyplot as plt

fig, ax = plt.subplots(2, 1)

ax[0].plot(xvalues, yvalues)
ax[1].plot(xvalues, yvalues)

plt.show()
```

Not only does this make things easier, it also solves the counting 
issue, as the returned list of `Axes` objects will automatically satisfy 
the default Python counting convention. The only annoying thing 
(although it actually really isn't annoying) is that this function also 
returns the `Figure` object that contains the `Axes` objects, so you 
need to provide an additional variable name to store this (or deal with 
the fact that the `plt.subplots` return value is a *tuple*).

When you create multiple rows *and* columns, `plt.subplots` will 
automatically create a multidimensional list. The first dimension will 
contain the rows, while the second dimension contains the columns:

```
import matplotlib.pyplot as plt

fig, ax = plt.subplots(2, 2)

# top left:
ax[0][0].plot(xvalues, yvalues)

# top right:
ax[0][1].plot(xvalues, yvalues)

# bottom left:
ax[1][0].plot(xvalues, yvalues)

# bottom right:
ax[1][1].plot(xvalues, yvalues)
```

# Adding colour bars

Colour bars are by far the most painful part of making plots in 
matplotlib. If you have one plot, you can very easily add a colour bar to 
it using the following code:

```
import matplotlib.pyplot as plt

plt.contourf([[0., 1.], [1., 0.]])
plt.colorbar()
plt.show()
```

Again, all of the complexity has been conveniently hidden by sensible 
defaults. The `plt.colorbar` function that maps to the underlying 
`Figure.colorbar` function, first of all requires a *mappable*, i.e. a 
reference to the plot it describes. This could for example be the return 
value of the `plt.contourf` function above. However, when called through 
the `pyplot` interface, this mappable defaults to the currently active 
plot. Second, the `colorbar` requires an `Axes` in which it can be 
drawn. This can be directly provided to the `colorbar` function through 
the `cax` argument, although this is rarely done. In most cases, a 
colour bar is directly associated with an existing `Axes` object. 
`Figure.colorbar` can automatically create a new `Axes` object within an 
existing `Axes` (stealing some of the space set aside for that plot) and 
use that. In this case the `Axes` object can be specified using the `ax` 
argument. In the absence of both `cax` and `ax`, the default uses the 
currently active `Axes` object as `ax`, which is exactly what we wanted 
in our example.

So all in all, color bars are not too hard, if you are happy to stick 
with the method of adding a color bar to an existing `Axes`. When you 
have multiple subplots, you just need to make sure to provide the 
correct `ax` argument. However, subtle problems might still arise 
because of the mappable associated with the color bar, which defaults 
to the active plot. The safest approach is to use the `Figure` object 
directly to call `colorbar`, and to provide all mappables explicitly:

```
import matplotlib.pyplot as plt

fig, ax = plt.subplots(2, 1)

plot0 = ax[0].contourf([[0., 1.], [1., 0.]])
plot1 = ax[1].contourf([[1., 0.], [0., 1.]])

fig.colorbar(plot0, ax = ax[0])
fig.colorbar(plot1, ax = ax[1])

plt.show()
```
 
This will fix potential issues with wrong colour bars being shown for a 
plot.

What if you want to add a single colorbar for multiple plots? This is 
when things start to become interesting. At the moment of writing, I 
have not been able to find a single elegant approach to solve this 
issue: there is no convenient `matplotlib` function that will create a 
new `Axes` object using the space of multiple existing ones. So the only 
way to make sure your color bar has a spot within your image, is by 
creating an `Axes` yourself, and passing it on using `cax`. However, 
this will likely create an `Axes` that is too wide or high for your 
color bar, as any convenient method of creating `Axes` objects assumes 
you want to use the `Axes` for a normal plot.

One approach (I have used quite often) is to manually specify the 
dimensions for the new `Axes` object, using a low level function like 
`Figure.add_axes`. This works, but requires a lot of fine tuning, as the 
positional arguments for this function use a weird internal length unit 
that changes as soon as anything else changes (or at least, that is what 
it looks like). It will also inevitably break convenient layout 
functions like `plt.tight_layout` that try to minimise white space and 
fix issues with overlapping labels.

My current favourite approach instead uses Matplotlib-managed subplot 
layouts, but uses them in a slightly more advanced way:

```
import matplotlib.pyplot as plt

ax = [plt.subplot2grid((2, 5), (0, 0), colspan = 4),
      plt.subplot2grid((2, 5), (1, 0), colspan = 4)]
cax = plt.subplot2grid((2, 5), (0, 4), rowspan = 2)

plot = ax[0].contourf([[0., 1.], [1., 0.]])
ax[1].contourf([[0., 1.], [1., 0.]])
plt.colorbar(plot, cax = cax)
plt.show()
```

`plt.subplot2grid` is similar to `plt.subplot`, and creates a new `Axes` 
object within a grid structure, in this case a 2 rows by 5 columns 
structure. However, the additional `colspan` and `rowspan` arguments for 
this function now allow us to make individual subplots span more than 
one spot within this grid structure. By making the two main subplots 
span 4 columns each, we make sure that they are 4 times wider than the 
color bar that occupies the last column. Similarly, the color bar can 
span two rows, so that it covers both subplots. As before, we have to manually
pass on a mappable to `plt.colorbar`. Here we exploited the fact that
both plots have the same range of colour values. For more complex plots this
might not be the case, and then you will need to find a better way to
provide an appropriate mappable.

# Conclusion

Once you start using `plt.subplot2grid`, you open up a whole world of 
possibilities that should cover almost all plots you would ever want to 
make. Despite this, I still hardly ever use this approach, as many of 
the other approaches I have shown you here are sufficient for most 
cases, and are a lot faster to write.

Also note that, once you start using `Axes` objects (as you probably 
should), many approaches can be used interchangeably: you simply change 
the way you create the `Axes` objects if you notice that you need more 
flexibility. This is what I tried to illustrate in the 
`plt.subplot2grid` example: the `ax` list in this example is the same as 
an equivalent `plt.subplots` call without color bar.
