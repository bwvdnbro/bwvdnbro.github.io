---
layout: post
title: "The CMacIonize unit system"
description: >-
  An overview of how the unit system in the simulation code CMacIonize works.
date: 2019-09-25
author: Bert Vandenbroucke
tags: 
  - Code development
  - Scientific computing
---

I have certainly mentioned the importance of units before. Units are not 
only important for physical reasons, because they are the things that 
link numbers that live in the pure world of mathematics to physical 
quantities that live in the real world. They are also important for 
practical reasons, because they are both so easy to deal with, as easy 
to neglect or accidentally get wrong.

That is why it is always a very good idea to think about units in an 
early stage of the development of a scientific code, so that they are 
there and you don't need to worry about them in the future. And I would 
also strongly recommend to be consistent with the system of units that 
is used internally inside the code, so that the only way units appear 
explicitly is in the unit conversion for input and output values.

In this post, I will discuss the unit system I devised during the 
development of my simulation code 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize), because I think it 
is fairly elegant and simple, yet also incredibly powerful. Within this 
unit system, adding new units is literally a one-liner, and most unit 
conversions are very conveniently hidden from developers by a clever use 
of C++ templates. I am not saying this is the *ultimate* unit system, 
but it is definitely something that has made my life a lot easier, and 
something I am proud of.

# Overview

The CMacIonize unit system consists of two C++ header files with just 
over 700 lines of text in total, including Doxygen comment blocks, 
license headers and inline comments (and with a line width of 80 
characters). It is hence not an awful lot of code. The two files contain 
the two essential parts of the unit system: a class that represents a 
unit 
([Unit.hpp](https://github.com/bwvdnbro/CMacIonize/blob/master/src/Unit.hpp)), 
and a class that deals with the conversion between different units for 
the same quantity (or different quantities that are in some way 
compatible, like wavelength - a *length* -, frequency - an inverse 
*time* - and energy - *mass times length squared divided by time 
squared* - for light; 
[UnitConverter.hpp](https://github.com/bwvdnbro/CMacIonize/blob/master/src/UnitConverter.hpp)).

Below, I will explain these two parts in more detail, starting with the 
easy one.

# The Unit

A physical unit has two purposes. First of all, it needs to tell us what 
*quantity* we are dealing with: a length, time, mass, temperature, 
current or angle (these are the quantities used in CMacIonize), or a 
combination of these. Second, the unit tells us what the size of that 
specific quantity is in comparison with some reference, which we are 
free to choose.

In order to make abstraction of a unit, we hence need to store two 
pieces of information: the quantity the unit represents, and the value 
of that quantity compared to the reference. We also need to choose a 
suitable reference; in the case of CMacIonize I opted to use SI units 
(but I could have chosen any other system of units). In SI units:
 * length is given in m (metres)
 * time is given in s (seconds)
 * mass is given in kg (kilograms)
 * temperature is given in K (Kelvin)
 * current is given in A (Amp&egrave;re)
 * angle is given in radians (no abbreviation here)

Representing the value of the quantity is relatively easy: we simply 
store the conversion factor from the specific unit represented by the 
`Unit` class as a floating point member variable. Representing the 
quantity is a bit more tricky, as the quantity could be any combination 
of basic quantities imaginable. Although that is a bit exaggerated; 
there are very strict rules about how basic quantities can be combined 
into new quantities: this can only happen by multiplying integer powers 
of basic quantities. For example, "kg m$$^2$$ s$$^{-2}$$" represents the 
unit of a new quantity (energy, this unit is sometimes also written as 
"J"), things like "kg $$-$$ s$$^{0.5}$$" are completely unacceptable for 
physical quantities.

In order to store information about the quantity a `Unit` represents, it 
is hence sufficient to store the integer exponents of the basic 
quantities that make up the derived quantity. The quantity for the 
energy unit "J" can then e.g. be represented as $$[2, -2, 1, 0, 0, 0]$$ 
using the order of the basic quantities given above.

This way of storing the quantity also makes it very easy to manipulate 
the unit during physical operations: dividing an energy ($$[2, -2, 1, 0, 
0, 0]$$) by a velocity squared ($$[1, -1, 0, 0, 0, 0]^2 = [2, -2, 0, 0, 
0, 0]$$) is as easy as subtracting the exponents of the latter from 
those of the former: $$[0, 0, 1, 0, 0, 0]$$ - a mass. Similarly, taking 
integer powers of a quantity is as simple as multiplying the exponents 
by that same power, as already illustrated in the previous example.

Most of what the `Unit` class does is implementing exactly this kind of 
functionality. It contains multiplication and division operators that 
correctly derive the new quantity that is the result of multiplying or 
dividing two quantities (and that also derives the correct new 
conversion factor by multiplying or dividing the two conversion 
factors), and a power operator that can raise a unit to an integer 
power. It also contains functionality to check whether two units are 
*compatible* (they represent the *same quantity*) or the *same* (they 
represent the *same quantity and the same conversion factor*).

The class also contains convenient functions to actually convert values: 
a multiplication operator that can convert a value *from* the unit 
represented by a specific `Unit` object *to* SI units, and a division 
operator that converts *from* SI units to the specific unit.

Lastly, the `Unit` class contains a `to_string` method that returns a 
string containing the conversion factor followed by a human readable 
representation of the unit, which is simply the SI units of the 6 basic 
quantities mentioned above raised to their respective integer power, 
separated by spaces. The order of the quantities is fixed to the order 
given above. Units for exponents that are zero are not present in the 
string, while exponents that are one are omitted from the string. The SI 
units are hard coded in this function, so the choice of reference unit 
systems is an essential part of the class.

# The UnitConverter

The `Unit` class above contains everything that is needed to work with 
units, but does not contain any actual information about units that are 
not the reference SI units. It can represent a quantity and its unit and 
write out the SI representation of the quantity as a string, but it 
cannot parse a string and convert it into a unit, nor can it deal with 
units that are not the basic 6 SI unit symbols (like "J", "erg" or 
"kpc"). These things are dealt with in the `UnitConverter` class.

Most of this class consists of hard coded information that is required to 
deal with additional quantities and units: an `enum` called `Quantity` 
that lists all physical quantities that we want to be able to use 
elsewhere in the code (I will come back to this later), a function that 
returns the `Unit` object corresponding to some unit symbol (like the 
"J"$$=1.0\times{}[2, -2, 1, 0, 0, 0]$$ we encountered before), and a 
function that returns the basic SI unit string for every item in the 
`Quantity enum` (e.g. `QUANTITY_ENERGY -> "kg m^2 s^-2"`). If we want to 
add an additional unit symbol, this is simply a matter of adding an 
additional `Unit` entry in the first function. If we want to add an 
additional quantity, we simply add a corresponding entry in the `enum` 
and an SI unit representation in the second function.

In order to practically work with `Unit` objects, we need one additional 
element: a function that can parse a unit string that can contain any 
quantity into a valid `Unit` object. This function should take input 
that roughly matches the formatting rules of the `to_string` method in 
the `Unit` class, but should be more flexible, as it cannot enforce a 
specific order of unit symbols, and should be able to recognise unit 
symbols that are not basic SI unit symbols. This function itself is 
quite complicated (as string parsing tends to be), but essentially does 
nothing more than going through the string, symbol by symbol, calling 
the unit symbol to `Unit` object class for every symbol and then 
applying the power, multiplication and division operators on the 
combined unit to eventually end up with a single unit that represents 
the whole string. If a unit symbol is found that does not have an entry, 
an error message is shown; either the symbol is wrong (a typo?), or a 
new entry needs to be made for a unit that is not part of the code yet.

Using this `get_unit` function, we can convert any string representation 
of a unit into a `Unit` object that we can then use to make unit 
conversions from and to that unit, using the `Unit` class member 
functions. To make this process even easier, the `UnitConverter` class 
also contains a member function called `convert` that simply takes a 
value, its input unit string and a desired output unit string, and 
outputs the value of that quantity in the output units. If the given 
input and output units are not compatible (e.g. one is "J" - an energy - 
and the other is "kpc" - a length), then the function will yield an 
error message.

While this functions is already quite powerful, it requires two unit 
strings: one for the input and one for the output. If the output value 
is to be used internally, then we know what the output unit string 
should be, provided that we know what quantity our input value is. Or in 
other words, we only need the output unit string to determine the 
quantity. We can also provide this information in a different way: using 
the quantities listed in the `Quantity enum`. To this end, the 
`UnitConverter` class contains additional member functions `to_unit` and 
`to_SI` that simply take a value and its input or desired output unit, 
and additionally take the quantity as a template argument.

The nice thing about these template functions is that they can be used 
for parameter input. In this case, the value and unit string are read in 
from some user provided source (a parameter file or simply the command 
line), and the code developer simply needs to specify the quantity that 
needs to be read in. The template functions will then automatically take 
care of the unit string parsing, and will check if the resulting unit is 
compatible with the desired quantity, before automatically converting 
the value to the internal SI units. This feature is used in almost all 
CMacIonize classes; most classes have a so called parameter file 
constructor that simply calls the normal class constructor using values 
read from the parameter file. Values that represent physical quantities 
will use a template `get_physical_value` or `get_physical_vector` 
function of the `ParameterFile` class that accepts a `Quantity` as 
template parameter. These functions will then call the `to_SI` member 
function of the `UnitConverter` class to parse the parameter value into 
an SI value. From the point of view of the developer, no units or 
conversion factors are present in the constructor, yet they are still 
invoked correctly. And the input unit is even validated against the 
desired quantity to check if it makes sense.

What about these weird quantities that are interchangeable: like 
wavelength, frequency and energy for radiation? To deal with these, the 
`UnitConverter` class contains one additional function called 
`try_conversion`, that is called whenever the `convert`, `to_SI` or 
`to_unit` functions detect incompatible units. This function contains a 
hard coded list of A and B quantities that are interchangeable, and the 
conversion factors between these. It will simply loop over this list and 
check whether the input and output unit strings are compatible with A 
and B, or B and A respectively. If this is the case, the conversion 
factor is used to make the conversion between quantities, before the 
usual unit conversion is applied. If no matching entries are found in 
the list, an error message is displayed.

# What about the overhead?

A significant fraction of the unit functionality is implemented using 
inlined operators that have no significant overhead at all: the compiler 
is usually capable of optimising these out to the raw unit conversion. 
There is a significant overhead associated with the unit string parsing 
however, so all functions that invoke this in any way (which means: 
almost all of the `UnitConverter` functionality) will be relatively 
slow. Not so slow that it cannot be used to read all of the input 
parameters in this way, but slow enough that you really don't want to 
use these functions in the inner bits of the inner loops of the code 
that take up most of the computation time. If you do need to do some 
unit conversions in a computationally intensive part of the code (e.g. 
if you are reading in a significantly sized data table that contains 
weird units), then you really need to make sure that you precompute the 
conversion factors using a few calls to the appropriate `UnitConverter` 
function before the loop, and then use that factor inside the loop.

What about the overhead of writing this unit system? All in all, I think 
writing this took a day or two (maybe three), including writing a simple 
unit test that guarantees all my units and quantities are implemented 
correctly. Judging from my experiences with codes that do not have such 
a user friendly unit system, I think I already made up a multiple of 
that in time not lost trying to figure out the units of my results, or 
trying to debug faulty unit conversions. So it was definitely totally 
worth it.
