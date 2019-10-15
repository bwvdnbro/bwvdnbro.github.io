---
layout: post
title: "Input and output with strings, part 1: output"
description: About formatted string output in Python and C(++)
date: 2019-10-15
author: Bert Vandenbroucke
tags:
  - Code development
---

If you ever wrote a piece of code - proper Fortran or C(++) code, or a 
script in some other language - you almost certainly at some point got 
confronted with input and output, either from/to text or binary files 
stored on the local file system, or from/to the command line terminal. 
If this is the case, then you probably also got confronted with the need 
to do conversions: numerical values that are written out or read in as 
plain text need to be converted from or to strings; for binary in/output 
individual bytes need to be converted into a variable of the write byte 
size.

The latter of these is quite straightforward, albeit generally 
system-dependent as well. If the value you are trying to read/write is a 
32-bit (4 byte) integer number, then the conversion simply consists of 
writing out the individual 4 bytes of this number, or reading them in. 
The only thing that could go wrong is the order: some systems store 
integer values from large to small byte components (small endian) while 
others use the opposite approach. To avoid inter-system confusion, it is 
therefore a good idea to use standardised binary formats for binary 
in/output (e.g. HDF5), which means you will need to use an external 
library for all your input and output. This might be a bit annoying, but 
also means all conversions will be done for you.

String input and output is - from a programmer's point of view - much 
more annoying. Different (scripting) languages use different mechanisms 
to support it, and while these mechanisms are definitely powerful enough 
to achieve everything you might want to achieve, it takes some time to 
master them. In all cases, string output requires you to specify some 
kind of output format for all numerical values that determines how they 
are put in a string and with what precision they are displayed. String 
input requires *parsing* the input string and retrieving numerical 
values by applying some *regular expressions* to it.

In this and a next post, I will discuss these two topics in more detail. 
This post will focus on string output, i.e. ways of formatting numerical 
values so that they can be written out.

# The standard way

Before I start exploring the complicated ways of formatting output, it 
is worth pointing out that most programming/scripting languages ship 
with some form of default formatting, that in many cases is sufficient 
for your basic output needs.

## Python

In Python, terminal and file output is provided by the functions 
`print()` and `write()` (in the soon to be deprecated Python 2, `print` 
was a statement instead of a function, so the brackets could be 
omitted). The `print()` function support strings:

```
print("Hello!")
```

multiple strings (concatenated with a ` ` separator):

```
print("Hello", "there!")
```

And also output values of other types:

```
var = 42
print("Number:", var)
```

For numerical types containing a single value, `print()` will just print 
the value itself. For arrays, it will print the entire array. If your 
variable is a Python object, it will either print the internal reference 
to that object (which looks a bit like the linker symbols I described in 
a post a while back), or a custom output string provided by the optional 
`__str__()` member function for the class. The `write()` function is 
unfortunately not as flexible.

## Fortran

I am generally not a big fan of Fortran, but in this case, the default 
Fortran output mechanisms are actually very good. Fortran has a `print` 
statement that is very similar to the Python `print()` (although it does 
not add a space in between the different parts of the output string):

```
var = 42
print *, "Hello ", "there! Var:", var
```

Even better is that this also works for output to a file using the 
`write()` function:

```
var = 42
open(unit = 1, file = "test.txt")
write(1, *) "Hello, var:", var
close(1)
```

## C++

In C++, the recommended mechanism for all input and output are streams. 
For terminal output, this functionality is part of the `<iostream>` 
header, and looks like this:

```
const double var = 42.;
std::cout << "Hello " << "there! Var: " << var << std::endl;
```

For file output (the `<fstream>` header), this becomes

```
const double var = 42.;
std::ofstream ofile("test.txt");
ofile << "Hello, var: " << var << std::endl;
```

These are very similar to the Python and Fortran output, except that now 
you explicitly need to iterate for arrays:

```
const double array[4] = {0., 4., 2., 3.};
std::cout << array[0];
for(int i = 1; i < 4; ++i){
  std::cout << " " << array[i];
}
std::cout << std::endl;
```

You can immediately see that this is somewhat annoying if you want to 
include some separator (there are ways around this, but they are quite 
complex themselves).

## C

As far as I am aware, C does not have a default way of formatting values 
for output. This means that all C output always requires format 
specifiers.

# The hard way

# C

Since C does not have a default way to output numerical values, its 
standard output mechanism is in fact immediately the hard way. So let's 
start there.

To output to the terminal, C provides the `printf()` function (contained 
in the `stdio.h` header):

```
printf("Hello!");
```

Unlike the print functions we encountered above, this function only 
takes a single string argument, plus an unspecified number of additional 
arguments that matches the number of *format specifiers* contained in 
this string. These format specifiers are essentially placeholders for 
these additional arguments in the string, and their type needs to match 
the type of the corresponding argument. For e.g. a string, the format 
specifier is `%s`:

```
printf("Hello %s!", "there");
```

For numerical values, the format specifiers depend on the type, and 
sometimes also on the byte size of the variable. For floating point 
values, the easiest format specifier is `%g`:

```
const double var = 42.;
printf("Var: %g", var);
```

This will use either exponential notation (format specifier `%e` or `%E` 
depending on whether you want the exponential sign to be small or 
capital; the latter requires `%G`) or floating point notation (`%f`), 
depending on which one results in the shortest output. This works for 
both single and double precision values (but not for long double 
precision values).

For integers, the format specifier `%i` or `%u` can be used, depending 
on whether the integer is signed or not. If you want to output in 
hexadecimal notation, you can use `%x` or `%X`, and for pointer 
variables there is the specifier `%p`. This only works for integer 
values with a size of 32 bits or less. For larger values (long integer), 
an additional `l` needs to be added to the format specifier (e.g. 
`%lu`), and for even longer values this could sometimes be two (the 
compiler will tell you if you didn't add enough).

These basic format specifiers can be extended with additional parts, 
like this:

```
%[flags][width][.precision][length]specifier
```

The `length` part is the `l` already mentioned before. The `flags` part 
gives additional information about justification, padding and whether or 
not a sign character/hexadecimal prefix is written out. The `width` part 
specifies the *minimum* width of the output string in number of 
characters (shorter output is padded until this width is reached), and 
the `.precision` (always preceded by the `.`) gives the precision as 
number of digits behind the decimal point (`%e/E` or `%f`) or as total 
number of significant digits (`%g/G`). Note that the `width` and 
`precision` parts also accept the value `*`, which means the value is 
provided as an additional argument to the `printf` function that 
precedes the argument that is printed.

For file output, the format specifiers are the same, but the `fprintf` 
function needs to be used. This function takes an additional first 
argument that points to a file to write to.

# C++

The format specifiers described above also apply to C++, where the 
`printf` function is also available. An additional complication can 
arise when the values to be printed are the more flexible C++ integer 
types contained in the header `<cinttypes>`, since these types can have 
different byte sizes on different systems. For convenience, 
`<cinttypes>` defines macros that contain the correct format specifiers 
for these cases:

```
const uint_fast32_t var = 42;
printf("Var: %" PRIuFAST32, var);
```

Note the space in between `%"` and `PRIuFAST32` that is obligatory in 
this case. Also note the absence of commas in the first argument.

Alternatively, the output streams that are more recommended in C++ also 
have features to specify output precision and formatting. To change from 
the default decimal output to hexadecimal or octal for example, you can 
use

```
const uint_fast32_t var = 42;
std::cout << std::hex;
std::cout << "Hexadecimal var: " << var << std::endl;
std::cout << std::oct;
std::cout << "Octal var: " << var << std::endl;
```

To change the width or precision, you can use `setw` and `setprecision`:

```
std::cout << setw(10);
std::cout << setprecision(8);
```

The default precision is 6.

## Fortran

In Fortran, format specifiers can be provided as follows:

```
var = 42
print "Var: I3", var
```

Generally, Fortran only has two important formats: `I` for integer 
values and `F` (or `E`) for floating point values. `I` takes one 
required and one optional additional argument:

```
I[width].[minimum width]
```

`width` is the *maximum* width of the string in characters (values that 
are longer than this will *not* be displayed properly), `minimum width` 
the mimimal width (shorter values will be padded).

For `F` and `E`, the additional arguments are

```
F[width][.precision]
```

and

```
E[width][.precision]E[exponent width]
```

Note that all format specifiers can optionally be preceded by a counter 
that duplicates the given format specifier, separated by spaces:

```
var1 = 42
var2 = 43
print "2I3", var1, var2
```

## Python

Python - being Python - offers a range of different format specifiers. 
My personal favourite is the `format()` function for strings:

```
var = 42
print("Var: {0}".format(var))
```

Here, every occurrence of curly brackets is replaced with an argument to 
the `format()` function. The curly brackets either contain the index 
(starting from 0) of the argument in the list of arguments (arguments 
can be used multiple times or not be used at all), or a label that 
identifies an argument:

```
var = 42
print("Var: {var}".format(var = var))
```

If arguments are simply used in the order they are given, the indices 
and labels can also be omitted:

```
var1 = 42
var2 = 43
print("{} {}".format(var1, var2))
```

Type specifiers can also be provided as an additional argument to a 
value placeholder, separated by a colon (`:`):

```
var = 42.
print("Var: {0:.2f}".format(var))
```

The format specifiers are reasonably straightforward: `d`, `b`, `o` and 
`x/X` for respectively decimal, binary, octal and hexadecimal integers, 
`f` for floating point values and `e` for exponential notation. 
Additional flags, width and precision arguments can be specified as 
well, similar to C(++). For a zero-padded 3 digit counter for example, 
you can use

```
var = 42
print("Var: {0:03d}".format(var))
```

Note that it is possible to unpack arrays or lists as arguments for 
`format()`. Values will however only be displayed if they also have a 
placeholder in the string:

```
a = [42., 32., 12.]
print("Format: {} {} {}".format(*a))
```

Alternatively, Python also supports an old syntax, that is very similar 
but less flexible, and makes use of `%`:

```
var = 42
print("Var: %i" % (var))
```

Python 3 has an even more flexible way of formatting strings, called *f 
strings*. They allow you to include variable names, function calls and 
even code statements in a string:

```
def add(a, b):
  return a + b

var1 = 42
var2 = 43
print(f"Var1: {var1}")
print(f"var1 + var2: {var1 + var2}")
print(f"var1 + var2: {add(var1, var2)}")
```

Apart from this, the syntax is pretty much the same as for the 
`format()` function. F strings are supposedly faster than their `%` and 
`format()` counterparts, although I am not really familiar with their 
usage (yet).
