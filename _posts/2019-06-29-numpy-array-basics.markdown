---
layout: post
title: "NumPy array basics"
description: A basic introduction to efficient use of NumPy arrays
date: 2019-06-29
author: Bert Vandenbroucke
tags:
  - Tools
---

Python is the most popular analysis tool in astronomy, and has been for 
a while. It is a *scripting* language that ships with tons of powerful 
libraries for data manipulation, analysis and plotting, and forms the 
backbone of many contemporary data pipelines, both in observational and 
computational astronomy.

Despite this, not all astronomers are aware of how to correctly use 
Python. I can think of a few examples: efforts to write high-performance 
simulation software in Python, people loading huge data sets in memory 
while running Jupyter notebooks on the login nodes of an HPC cluster... 
They are things that *work* (it is very hard to make something not work 
in Python), but they are for sure not the correct way to make use of the 
capabilities of Python and of your computing system. Proponents of the 
former might argue that compilers like [Cython](https://cython.org/) can 
turn inefficient Python code into efficient C code, but this puts so 
many restrictions on coding style that I cannot see any benefits of this 
method over simply writing your program in C (or C++, or even Fortran) 
from the start.

The aim of this post is to show some examples of how to use Python 
correctly. Again, I will try to distinguish between Python code that 
*works*, and Python code that actually uses Python libraries effectively 
to make efficient use of the system. I will focus on the usage of NumPy 
arrays, which are by far the most common and useful objects one can use 
in astronomical Python. Most of what I will discuss comes from direct 
experience; when I state that something is more efficient, I have 
usually experienced the difference myself at some point as a factor 10 
or more speedup between two versions of a script.

# What are NumPy arrays?

This point is already a very important one. NumPy arrays are usually 
considered the NumPy equivalent of Python lists, i.e. objects that 
contain a (large) number of elements (of the same type) and that can be 
indexed and edited. That they are not equivalent is immediately clear 
however from two important points:
 * Python lists do not necessarily need to contain elements of the same 
type (hence the brackets). If you have a list of floating point values, 
you can add a string to this and this will work.
 * NumPy arrays can only be edited in ways that do not change the size 
of the array, i.e. you cannot add elements to an array after it has been 
created.

This latter aspect is usually considered a nuisance by NumPy novices, as 
it makes it harder to convert your code that uses lists to code that 
uses arrays; instead of changing the type of your list to an array when 
you first declare it, you have to make sure your original list has its 
full size first, and then convert it into an array using code like this:

```
import numpy as np
my_list = []
my_list.append(2.)
my_list.append(3.)
my_list.append(4.)
my_array = np.array(my_list)
```

This is (surprise, surprise) not a very efficient way of using arrays. 
Although I consider this acceptable for small lists, and actually do 
this quite often. Since it is sometimes hard to know how many elements 
you will have before your list has been created, and the code below is 
better than using `numpy.append`:

```
import numpy as np
my_array = np.array([])
my_array = np.append(my_array, 2.)
my_array = np.append(my_array, 3.)
my_array = np.append(my_array, 4.)
```

The resulting array in this example is the same as in the first example, 
but the code is a lot less clean, and as is clear from the code, you are 
creating copies of the original array for each append operation. This is 
very inefficient.

Before showing how to do things correctly, I should first explain why 
NumPy arrays are so annoying. This is because of the way they work 
internally. When you create a NumPy array, you are actually creating a 
block of memory in the underlying NumPy library (written in C). This 
memory block needs to be *assigned*, and for this C needs to know 
exactly how big it will be in memory. This not only means it needs to 
know the *size* of the array, but it also means it needs to know the 
size of each *element*, and this size is encoded in its *data type*. A 
floating point value takes 4 or 8 bytes (depending on whether it is 
single or double precision). A string on the other hand has no 
predefined size (since the size depends on the number of characters in 
the string), which means that you cannot store a basic Python string in 
an array.

Since expanding an array requires a reallocation of the existing memory 
block, it automatically means that the entire array needs to be copied 
whenever an element is added to it. The developers of the NumPy library 
could have implemented an `append` function that does this, but this 
would have hidden the complexity of the append operation and opened up 
the library to bad usage. So instead they opted to provide 
`numpy.append`, a function that openly makes a copy of the array, to 
convince users to only use it when it is really necessary.

Next to the size and data type, a NumPy array also has a *shape*, 
defining its layout in memory. The array that is the equivalent of a 
Python list is a simple 1D array with a shape equal to its length. But 
arrays can also have higher dimensionalities, and then their shape is a 
*tuple* that defines their size in the different dimensions. The size of 
the memory block assigned by NumPy for the array is independent of its 
shape, so that you can change an array shape in place using 
`numpy.reshape`.

If you know the size of your array in advance, you should immediately 
create a NumPy array with that size. There are many convenient functions 
for this:

```
# create an empty, uninitialised array of 3 elements of default type float
my_array = np.empty(3)
# create an array of 3 elements of type int, all initialised to zero
my_array = np.zeros(3, dtype = int)
# create an array from the given list
my_array = np.array([2., 3., 4])
# create an array with consecutive elements from 2 to 5 (exclusive) and step
# size 1
my_array = np.arange(2., 5., 1.)
```

All of these will allocate the array memory block once, and optionally 
initialise the elements to predefined values. There are many more of 
these functions, including some that initialise values from files, and 
most other NumPy functions and functions from other libraries will also 
return arrays.

# Array operations

As explained in the previous paragraph, arrays cannot be edited as 
objects, i.e. you cannot change their size or data type without making a 
copy of the array. Array elements on the other hand can be changed in 
almost any way possible.

The most obvious way to change an array is by accessing and changing its 
individual elements:

```
# overwrite an array element with a new value based on the old value
my_array[0] = my_array[0] + 2.
# edit an array element in place
my_array[1] += 3.
# edit all array elements
for i in range(len(my_array)):
  my_array[i] *= 2.
```

The two first examples are perfectly acceptable, as they involve a 
single array element and a single operation. The Python interpreter can 
execute them pretty much as efficiently as the NumPy library can. The 
last example however should not be used, as it involves a *loop* over 
all array variables, to then apply exactly the same operation to all of 
them. This loop will be executed by the Python interpreter at run time, 
and will not be able to make use of any optimisation because of the way 
the Python interpreter works (in fact, *not doing loops in Python* is a 
very good way of avoiding a lot of inefficient Python code). Much better 
is to let NumPy itself handle the loop, by performing a *block* 
operation:

```
my_array *= 2.
```

This is not only much easier to read, but avoids having to perform a 
loop in Python, instead making use of highly optimised code inside the 
NumPy library. And the result is exactly the same.

But what if your loop involves the loop counter, `i`, like in the 
example below?

```
for i in range(len(my_array)):
  my_array[i] += 2.**i
```

Easy: we just create an array that contains the terms we want to add, 
and then perform the array addition in NumPy:

```
irange = np.arange(0., len(my_array), 1.)
my_array += 2.**irange
```

For special functions like `sin` and `cos`, you probably already need to 
use NumPy (you could use `math`, but I would advise against that). In 
this case, the function already works on arrays, so there is no need to 
do anything special:

```
my_array = np.sin(my_array)
```

Finally, NumPy provides a large number of functions to compute 
properties across array elements, like the mean, standard deviation or 
sum:

```
mean = my_array.mean()
std = my_array.std()
sum = my_array.sum()
```

# Advanced indexing

A last topic I want to touch upon is advanced indexing, i.e. ways to 
efficiently apply operations to a part of a (large) array without the 
need to create a new array or perform a (slow) Python loop.

The basic indexing mechanisms for arrays are the same as for Python 
lists:

```
# select all elements with indices between 2 and 4 (exclusive)
subset = my_array[2:4]
# select all elements from index 1 until the end of the array
subset = my_array[1:]
# select all elements from the start of the array until the one to last
subset = my_array[:-1]
```

The last two can be very effectively combined to compute an array with 
the midpoints of an array of bins:

```
midpoints = 0.5 * (bins[1:] + bins[:-1])
```

The list of basic indexing operations continues:

```
# select every second element in the array
subset = my_array[::2]
# reverse the array
subset = my_array[::-1]
```

Unlike for Python lists, these operations also work for 
*multidimensional* arrays, in which case it is instructive to think of 
them as carving out a small square or cube within a larger block:

```
my_array = np.array([[2., 3., 4.], [3., 4., 5], [3., 2., 1.]])
# select the last two rows of the first two columns
subset = my_array[1:,:-1]
```

An even more powerful way of indexing is provided via *array indexing*. 
In this case, you index the array with an array (or list) of boolean 
(`True`/`False`) values of the same size (and shape) that determine 
whether or not the corresponding element should be part of the 
selection:

```
my_array = np.array([2., 3., 4.])
subset = my_array[[True, False, True]]
```

This might seem inefficient at first, but becomes very powerful once you 
realise you can generate such boolean arrays by applying comparison 
operators to an array:

```
bool_array = my_array > 2.
subset = my_array[bool_array]
```

This bit of code will create an array that contains all elements in the 
original array that are larger than two, without requiring a loop!

Boolean arrays are hence very good to apply *filters* to arrays. These 
filters will also work across arrays. Assume for example that you have 
two arrays: one containing positions, and one containing values at those 
positions, and you want to compute the mean for all values at positions 
larger than some value. The following code will do the trick:

```
filtered_values = values[positions > 2.]
```

This is literally a one-liner!

Finally, there is the `numpy.where` function. In its most powerful form, 
this function takes a boolean array as input, and generates a new array 
by choosing from two other arrays (all arrays need to have the same 
shape). This can be very useful when you want to deal with problematic 
array elements:

```
a = np.array([2., 0., 4])
b = np.where(a > 0., 1. / a, np.zeros(a.shape))
```

In this case, `b` will consist of the inverse of all elements in `a` 
that are non-zero, and zero for the second element, that would otherwise 
result in `inf`.

If you somehow already have unwanted elements that have `inf` or `NaN` 
values, then you can use `numpy.isinf` or `numpy.isnan`, functions that 
return boolean arrays that select out these elements:

```
a = np.array([2., 0., 4])
b = 1. / a
b[np.isinf(b)] = 0.
```

# What's more?

A lot. In this post I have just scratched the surface of what is 
possible with NumPy arrays, but hopefully enough for most applications. 
I think the most important lesson I learned is that you should always 
avoid looping over arrays in Python, and that this is almost always 
possible once you know how to use advanced indexing, and the many (many) 
functions that the NumPy library provides. This might require a bit more 
creative thinking from your part, but will usually result in code that 
is a lot cleaner, and always result in code that is a lot faster to 
execute.

As with many things, the best way to learn how to use NumPy arrays is 
practice. And now I hope you know what to practice.
