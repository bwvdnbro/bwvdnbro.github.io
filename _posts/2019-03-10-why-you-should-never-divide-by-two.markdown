---
layout: post
title: "Why you should never divide by two (on a computer)"
description: >-
  A brief introduction to integer and floating point arithmetic using a very
  bold and simple statement.
date: 2019-03-10
author: Bert Vandenbroucke
tags: 
  - Scientific Computing
  - Code Development
---

In this week's post, I will try to make you appreciate how computers 
perform mathematical operations using the very basic example of dividing 
a number by two. I will try to convince you that dividing by two 
(explicitly) is probably never a good idea (if you care about speed). 
And then I'll tell you why you might get away with it anyway. But by 
then you should know enough about how computational arithmetic works to 
not want to divide by two any more. Or that is the idea.

When dividing a number by two, the first thing to know is what type of 
number we are dealing with. Computers deal with two different types of 
numbers: integers and floating point numbers. Both of these exist in 
various flavours: there are 32-bit and 64-bit floating point numbers, 
and many more flavours of integers. The flavour of the variable deals 
with the *precision* of the underlying value (I will use *precision* 
instead of *flavour* from now on), but also with how much memory is 
occupied by storing that variable. More precision requires more memory. 
For our purposes, the precision of the variables does not really matter. 
The type however determines how a division by two is performed.

# Integer division

Integers are conceptually the easiest variables to understand. They are 
the computer equivalent of positive and negative *whole numbers*, i.e. 
numbers that do not have any fractional component. An integer is 
represented in memory as a bit sequence like this (I will use a 4-bit 
integer as example):

$$0101$$

This bit sequence is simply the binary representation of the whole 
number. Each bit in the sequence (from right to left) represents the 
pre-factor for the corresponding power of two in the number's 
decomposition:

$$0101 = 0 \times{} 2^3 + 1 \times{} 2^2 + 0 \times{} 2^1 + 1 \times{} 
2^0 = 5.$$

To also represent negative integers, the highest bit is used as a sign 
bit, with $$0$$ meaning a positive integer and $$1$$ meaning a negative 
integer. Because this effectively means you lose one bit of precision, 
it is important to make a clear distinction between *signed* and 
*unsigned* integers. There are various ways to represent negative 
integers, and I will not go into more detail here.

You also have to be very careful when converting between the two 
integer types: any *unsigned* integer that is large enough to use the 
highest bit in its representation corresponds to a negative *signed* 
integer with the same precision, and hence to a completely different 
value! Similarly, subtracting one from an *unsigned* integer value of 
zero and then interpreting the result again as an *unsigned* integer 
leads to a very large *positive* value instead of the expected negative 
value. This is a very common mistake in loop conditions that use 
unsigned integer counters. Fortunately, compilers can warn you for this 
kind of mistake.

To implement arithmetic operations with integers, computers use basic 
binary operations like *bit shifts* and *bit-wise or*, *and* and *xor* 
operations.

A *bit-shift* moves all bits in the sequence to the left or right for a 
certain number of places, throwing away the bits at the far end, and 
filling up the empty bit positions with $$0$$. For our example 4-bit 
integer, a left shift by 1 position leads to

$$(0101) \ll{} 1 = 1010 = 10,$$

while a right shift by 1 position leads to

$$(0101) \gg{} 1 = 0010 = 2.$$

These values are respectively double and half (rounded down) the 
original value. This is not a coincidence, but is a direct consequence 
of the binary representation of integers.

*or*, *and* and *xor* operations take two bit sequences as input and 
return a single bit sequence that contains respectively the bit-by-bit 
*or*, *and* and *xor* value of the individual bit pairs:

*or*:

```
(0101) | (1010) = 1111
```

*and*:

```
(0101) & (1001) = 0001
```

*xor*:

```
(0101) ^ (1001) = 1100
```

Apart from these, there is also the *not* operator, that simply changes 
every $$0$$ bit into $$1$$ and vice versa for a single input value.

Computers use these operations to carry out additions, subtractions, 
multiplications and divisions for integer values. These are pretty much 
defined as the algorisms taught in primary school, where you write down 
your integer values and carry out complicated calculations by applying 
simple operations on the individual bits (and by carrying auxiliary 
variables in between steps). For the particular case of division, this 
means division is implemented similar to a long division. In practice, 
this means integer division is quite a complicated operation that 
involves a lot of intermediate additions and multiplications; an integer 
division is a lot more expensive in terms of computations than any of 
the other integer operations. Furthermore, computers completely ignore 
the remainder of the integer long division, which means that integer 
divisions always *round down*.

To get back to the point of this post: if we want an integer division by 
two operation that always rounds down, there is a much simpler 
alternative than the general integer long division algorithm: we can 
simply right shift the integer bit sequence by one position! Since this 
is a basic bit-shift operation, it is much cheaper to compute, but the 
result is exactly the same. Similarly, integer division by 4, 8, 16... 
can simply be replaced by a right shift with 2, 3, 4... positions. Any 
other denominator for the division requires the much more expensive long 
division algorithm.

# Floating point division

Integers are a very natural numerical type for computers, as they 
intrinsically use bit sequences through the binary representation of 
numbers. Unfortunately, they offer very little dynamic range in values: 
a 32-bit unsigned integer can only represent numbers in between $$0$$ 
and $$2^{32} = 4,294,967,296$$, which is not a lot for most scientific 
applications. Integer arithmetic also means that the accuracy of some 
operations (especially division) will depend a lot on the numbers that 
are involved, which again is not ideal for scientific purposes.

To address these issues, a second type of numerical variable was 
created: a floating point variable. As the name suggests, these 
variables can represent numbers that have a fractional component. They 
also offer a much larger dynamic range of values. The price you pay for 
these features is a loss of accuracy: while most integer additions and 
subtractions are exact (as long as the numbers involved are small 
enough), floating point arithmetic introduces potential round off 
errors for all mathematical operations, within some reasonable limits.

A floating point value, irrespective of its precision, is represented 
similar to a number written down in scientific notation: it consists of 
a *mantissa* that contains the significant digits for the underlying 
value, and an *exponent* that specifies the order of magnitude for the 
value. Unlike scientific notation, the entire floating point is still 
represented in a binary representation, which means that the exponent is 
a power of two rather than ten. To represent negative numbers, floating 
point values also have a sign bit. There are a number of possible ways 
to put all of these parts together (and a number of ways to distribute 
the available bits among the parts) that vary slightly from 
implementation to implementation, so I will not go into to much detail 
here.

Floating point operations, just like their integer counterparts, are 
constructed by means of simple binary operations on the mantissa and 
exponent of the variables that are involved. A multiplication for 
example consists of an addition of the exponents of the two variables 
and a multiplication of the two mantissas, followed by a renormalisation 
of the exponent (by convention, the mantissa only contains the digits 
behind the decimal point and assumes the part before the decimal point 
is 1; this means that every floating point value can only be written in 
one unique way). As before, a division is much more complicated and 
involves more work. The exact speed difference between a multiplication 
and a division depends on the precise computer architecture, but factors 
in between 5 and 10 are common.

So to get back to the point of the post (again): a division by two is 
equivalent to a multiplication by $$0.5$$. Since $$0.5$$ can be exactly 
represented in floating point representation, and since a floating point 
multiplication is always faster than a floating point division, you are 
much better off converting your division by two to a multiplication by 
half.

For other denominators things might be more complicated, as not every 
fraction can be exactly represented in floating point format. But if you 
really care about speed, you are generally better off converting the 
division into an equivalent multiplication. There might be a little loss 
in accuracy, but floating point operations are usually not perfectly 
accurate anyway. The only reason to explicitly perform floating point 
divisions is if they involve a variable. And even in that case it can 
sometimes be advantageous to compute the inverse of that variable first 
and the multiply with it, e.g. if you need to do a lot of divisions by 
that same variable.

# Compilers

By now, it should be clear that you should never explicitly divide by 
two: for integers, you can simply bit-shift instead, while for floating 
point variables you can (and should) multiply with half. That being 
said, a lot still depends on how you are developing your code, as things 
might not be quite as bad as I painted them here.

If you write code in a high-level language like C(++) or Fortran, your 
code needs to be converted into machine readable code by a compiler. 
Current compilers are pretty powerful, and they can automatically 
convert some of your divisions by two into multiplications.

Consider the following example C++ code snippets that perform a number 
of divisions by 2 either explicitly or by multiplying by half instead:

```
#include <iostream>

int main(int argc, char **argv) {
  double result = 0.;
  for(int i = 0; i < 100; ++i){
    double x = i * 24.9;
    x /= 2.;
    result += x;
  }

  std::cout << result << std::endl;

  return 0;
}
```

```
#include <iostream>

int main(int argc, char **argv) {
  double result = 0.;
  for(int i = 0; i < 100; ++i){
    double x = i * 24.9;
    x *= 0.5;
    result += x;
  }

  std::cout << result << std::endl;

  return 0;
}
```

We can compile both these code snippets, run them, and then compare the 
speed difference. That is however quite tricky, as the loop is quite 
short and the whole program takes less time to run than the time it 
takes the operating system to start it. Furthermore, there are a number 
of other factors that can affect speed.

A better way to compare both snippets is by looking at how they 
translate into machine code. When a compiler creates machine code, it 
converts the C++ into a set of machine instructions that is called 
*assembly*. Assembly is itself a programming language (with many 
different flavours), but it is very low-level: it consists of a list of 
*instructions* for the CPU like *move this variable into this cache 
register* or *add the variable in this register to the one in this 
register*. Compilers usually have a way of outputting the assembly code 
they generate; for the GCC compiler this is done with the command line 
option `-S`.

We can compile both snippets of code above with `-S` and then look at the
resulting code. Or even better: compare the two output files using `diff`.
If you compile using the most basic compilation command:

```
> g++ -S division.cpp
> g++ -S multiplication.cpp
> diff division.s multiplication.s
```

then this comparison will probably return you a whole list of weird 
instructions, showing that the instructions generated for both versions 
are different. Things look a lot better when you enable compiler 
optimisations:

```
> g++ -O3 -S division.cpp
> g++ -O3 -S multiplication.cpp
> diff division.s multiplication.s
```

This should only show you one difference between both versions: the 
name of the source code file. In other words: the compiler 
automatically converted the division into the much cheaper 
multiplication! So if you compile with optimisations, dividing by two 
is probably not a real problem.

Note that this only works for programming languages that are compiled 
and not for *interpreted* languages like Python or Java. The Python or 
Java interpreter might be good enough to automatically convert your 
divisions by two into multiplications by half, but you should probably 
not count on it. It is a lot easier for a compiler to do this kind of 
optimisations at compile time, when it can thoroughly analyse your code 
and decide what optimisations to do, than it is for an interpreter to 
make these decision based on single lines of code at run time.

# Memory-bound algorithms

Apart from your compiler helping you by optimising your badly written 
code, there is the more fundamental question whether or not the speed 
difference between multiplication and division actually matters for 
your specific use case. Division might be almost an order of magnitude 
slower than multiplication, it is still incredibly fast compared to 
some other CPU operations, like fetching variables from memory. Many 
algorithms nowadays are *memory-bound*, which means that their 
execution time completely depends on the speed with which variables can 
be read from memory rather than the speed with which operations on 
these variables are performed. This means that the CPU is very likely 
to be spending most of its time waiting for variables to arrive, and 
comparatively little time actually performing operations on these 
variables. If you have little operations overall, the overhead of a 
division might be completely invisible in your overall performance.

That being said, performance issues like these are very 
system-dependent, and it is very hard to make specific claims about 
your algorithm without actually measuring them very carefully. A 
multiplication is guaranteed to always be faster than a division, so it 
is probably still a good idea to assume it will be, even if it does not 
really matter for your use case. I personally think it is a good habit 
to try to minimise the number of divisions, even if it is not always 
necessary.

# Readability

Once you can be confident that your compiler will optimise out your 
divisions by two, you can concentrate on another important aspect of 
good code: readability. Division by two or multiplication by half makes 
very little difference for someone that reads it. Division by 16 or 
multiplication by 0.0625 could make a significant difference. A factor 
1/16 is easier to interpret that some arbitrary looking number. Either 
way, it is probably a good idea to also provide some comments that 
explain where that factor comes from.

For integer division by two this is even more true: the bit-shift 
operator `>>` is not a common mathematical operator that scientists 
necessarily know, so `a/2` and `a>>1` might look completely different 
to them, despite being exactly the same thing. I generally prefer only 
using bit-wise operators when the code in question requires them and I 
can be sure that whoever reads the code knows what `<<` means. But then 
again, if you are using integers in a bit of the code that does not 
require bit-wise operators, you are probably doing something weird, as 
most scientific applications require floating points.
