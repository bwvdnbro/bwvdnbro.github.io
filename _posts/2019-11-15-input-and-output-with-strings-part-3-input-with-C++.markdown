---
layout: post
title: "Input and output with strings, part 3: input with C(++)"
description: Reading values from strings using C(++)
date: 2019-11-15
author: Bert Vandenbroucke
tags:
  - Code development
---

A while ago I described how to read values from strings using Python, 
and I introduced the concept of regular expression parsing for this 
purpose. In this post I will describe how to achieve the same using C 
and C++. Surprisingly, it will turn out that these things are actually 
easier to do in C(++) than in Python, which is not usually what happens. 
There are however a few pitfalls that can make things very complicated 
very quickly (again).

# C++

I will start off with C++, as this is by far the most intuitive. As for 
its output, C++ uses *streams* for input, and these input streams are 
quite powerful. Consider the following example:

```
#include <iostream>
#include <sstream>
#include <string>

int main(int argc, char **argv){

  const std::string text = "100 3.14e-8";

  std::istringstream istream(text);
  int a;
  double b;

  istream >> a >> b;

  std::cout << a << " " << b << std::endl;

  return 0;
}
```

Our example string contains two numbers: an integer and a floating point 
value. By creating a `istringstream` (an input stream - `istream` - 
based on a single string) from this string, we can read these into 
variables `a` and `b` with those corresponding types. The `istream` will 
automatically take care of all necessary conversions; it recognises the 
exponent in the floating point value and knows how to deal with it.

The `istream` will also detect the end of each of the numbers in the 
string, based on the whitespace it encounters. If all numbers have 
clear types and are nicely separated by whitespace, then this approach 
will work very well.

There are a few caveats though. First of all, the `istream` conversion 
for integers is not as powerful as you might want: if we were to replace 
the `100` in the example with the equivalent `1e2`, then the parsing of 
the integer will fail. Second, if parsing for one of the values fails, 
the `istream` will fail to properly recognise the end of a number, which 
means that it will also fail to process any subsequent numbers. The 
replacement mentioned above will hence completely break the example.

You could argue that `1e2` is not a valid way of writing integers (this 
is what the C++ developers would argue) and that this behaviour is fine. 
And you could instead be amazed by another feature of `istream`: it can 
actually parse things like `0x1a` (the number `26` in hexadecimal 
notation):

```
const std::string text = "0x26 3.14e-8";
istream >> std::hex >> a >> b;
```

(only the lines that changes are shown). This is nice, but unfortunately 
not very flexible; if the first integer is not given in hexadecimal 
notation, then `istream` will still parse it as if it were. The number 
`100` then wrongly parses to `256`.

So despite the fact that input streams are incredibly easy to use, they 
are not quite as powerful as a good regular expression. If you really 
want to parse numbers with an unexpected format, then you need to write 
your own parser (this is what I did for 
[CMacIonize](https://github.com/bwvdnbro/CMacIonize/blob/master/src/Utilities.hpp#L119)).

# C

C has a number of functions for input that are the counterparts of the 
output functions we already encountered: `scanf` to read from the 
standard input (counterpart of `printf`), `sscanf` to read from a (C) 
string (`sprintf`) and `fscanf` (`fprintf`) to read from a (C) file.

The nice thing about these functions is that they are pretty much the 
exact counterparts of their respective output functions: they use the 
same format specifiers and almost the same syntax. The only difference 
is that while `printf` and its variants take *values* as additional 
arguments, `scanf` and counterparts take *pointers*. We can rewrite the 
C++ example from above as (using an allowed mixture of C and C++):

```
sscanf(text.c_str(), "%i %lg", &a, &b);
printf("%i %g\n", a, b);
```

Note that the format specifiers for floating point values are somewhat 
more restrictive for `scanf` than for `printf`: `%g` will work to output 
both single and double precision floating point values, but it will not 
work to read double precision values (you will get a compiler warning if 
you try this). The reason is that the precision of the value determines 
the size of the pointer passed on to `scanf`. If this size does not 
match what the function expects, undefined behaviour can occur. We can 
get rid of these compiler warnings by specifying the precision of the 
value: `%lg`.

`scanf` still has the same limitations as `istream`: it will fail to 
parse `1e2` as an integer and that replacement will break the parsing of 
the second value too. `scanf` also supports hexadecimal notation, but 
also requires a specific format specifier for this, which means you 
cannot parse values for which you do not know if they are decimal or 
hexadecimal. So while `scanf` feels a bit like the regular expressions 
you encounter in Python, it really is not as powerful.

# Regular expressions

At the start of this post, I might have given the impression that C and 
C++ have similar capabilities to the regular expressions available in 
Python. This is not true: while regular expressions can be used to find 
and parse values *anywhere* in a string, input in C(++) is much more 
limited: the available functions can only parse values if they know 
where in the string those values are and if they know what format to 
expect for these values.

But this makes sense. C and C++ are low-level languages that are 
intended to write actual machine code. They contain very detailed 
instructions about operations and memory management that are then used 
by a compiler to generate machine instructions that lead to optimal or 
close to optimal use of the available hardware. Regular expression 
parsing is a high-level feature, which requires a lot of operations 
executed according to complicated algorithmic rules. If C(++) were to 
have actual support for regular expressions, then this would in fact 
mean that all these complicated operations would need to be provided by 
the compiler that generates machine code, including all the memory 
management that goes with parsing the expression. This is not what 
compilers are made for, and also goes against the whole spirit of C and 
C++, where the programmer has maximal control over memory allocations 
and operations.

In other words: C and C++ cannot parse regular expressions in the way 
Python does, because Python uses a very complicated high-level library 
to do so. This library uses a lot of internal variables and operations 
to actually perform the regular expression parsing, all of which is 
hidden from the user. In C(++) you are not allowed to hide anything from 
the user, except if the user explicitly decides to do so, for example by 
using an external library. That is why the standard language does not 
offer any regular expression capabilities.

The fact that C(++) is not as flexible and forces you to write your own 
code to parse more complicated values might make you feel like parsing 
strings in these languages is incredibly inefficient. This is not true: 
the much more elegant regular expressions you use in Python internally 
use code that is at least as complicated and inefficient as the code you 
would write in C(++). But in the end, all this code is converted into 
very similar machine code, and should have roughly the same speed. This 
speed is a lot slower than performing some basic mathematical 
operations. But that is just the nature of strings...
