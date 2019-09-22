---
layout: post
title: "Computing special functions"
description: Why special functions are not as bad as we think they are.
date: 2019-09-22
author: Bert Vandenbroucke
tags: 
  - Code development
  - Numerical recipes
---

I might have mentioned this once or twice before, but computers are 
essentially very stupid, since all they can do is add, subtract, 
multiply, divide and make simple yes/no decisions. Everything else a 
computer can do is done by combining these operations in very inventive 
ways and making use of the absurd speed at which a computer can perform 
these operations.

In real world applications, these basic operations are not all we need 
to compute complicated models. We would like to do more: powers of 
numbers (just a lot of multiplications, right?) also with non-integer 
exponents (oh, oh...), logarithms... And of course we also need *special 
functions*, like the trigonometric functions $$\sin{}$$, $$\cos{}$$ and 
$$\tan{}$$, their inverse functions and related functions.

As I already mentioned, it is technically not possible to compute these 
special functions directly; this has to be done using the basic 
operations a computer can perform. As a user or scientific code 
developer however, you don't want to worry about this; someone once 
figured out how to compute trigonometric functions numerically, and that 
code is now part of many standard code libraries that you can simply use 
in your programs. You can trust that this code is correct (it has been 
tested by many, many people by now) and that it provides the most 
accurate answer possible for the finite precision available on your 
computer. So all is well.

Unfortunately, all is not necessarily well. While modern programming 
languages like C++ include [a lot of special 
functions](http://www.cplusplus.com/reference/cmath/), and [more are 
added all the 
time](https://en.cppreference.com/w/cpp/experimental/special_math), you 
might not be able to find the special function you require within those 
easy to use existing libraries. Simply because there are [a lot of 
special functions described in mathematical 
literature](https://en.wikipedia.org/wiki/List_of_mathematical_functions), 
and only so many capable scientific software engineers to implement and 
test them. When this happens, you have two options: you can either find 
a less easy to use library (like [Boost](https://www.boost.org) or 
[GSL](https://www.gnu.org/software/gsl/)) that usually adds a pretty 
heavy dependency to your code, or you can choose to implement the 
special function yourself. In this post, I will outline how the latter 
can be achieved.

# What is a special function?

Before I can get into the practical details of actually computing 
special functions, it is important to realise what a special function, 
or in fact, any function in programming really is: it is a construct 
that takes one or multiple *input values* and converts those into an 
*output value*. In programming, the number of output values is not 
necessarily limited, but for the purpose of mathematical special 
functions we will assume it is limited to one. This is exactly what we 
also assume when we write down a mathematical equation containing
special functions, like:

$$
2\cos(x) + 3\sin(x).
$$

In this equation, the $$\cos$$ and $$\sin$$ are symbols that represent 
some value $$y$$ that solely depends on $$x$$ and for which we have an 
unambiguous procedure to calculate it. Although we write these symbols 
down for a general, *unknown* $$x$$ value, we can at any time replace 
$$x$$ with an actual value, unambiguously calculate the values for all 
the symbols, and get a numerical result for the equation that is as 
accurate as we want it to be. In the meantime, we can still manipulate the
equation symbolically, and we can use known *properties* of the special
functions $$\cos$$ and $$\sin$$ to rewrite it, e.g.

$$
2\cos(x) + 3\sqrt{1 - \cos^2(x)}.
$$

While this is very easy to see and accept for basic trigonometric 
functions with which we are all very familiar, it is very important to 
realise that this is also true for other special functions, like e.g. 
the spherical Bessel functions of the first kind $$j_n(x)$$ (pretty much 
the spherical counterpart of the basic trigonometric functions). Just 
like for $$\cos$$ and $$\sin$$, we can usually write down an unambiguous 
procedure to compute $$j_n(x)$$ (I will come back to this later) for any 
specific value of $$x$$, while we can also choose to manipulate symbolic 
equations containing $$j_n(x)$$ without having to worry about how to 
compute it unless we really need it. So for all intents and purposes, 
the spherical Bessel functions of the first kind are numerically 
equivalent to basic trigonometric functions. Yet, for some reason, 
people feel generally a lot more comfortable with the latter.

To conclude: all we need to program a special function is an unambiguous 
method to convert *any* given input parameter or set of input parameters 
into a single output value. What happens during that process can be 
quite complicated, but as long as the result does indeed represent the 
underlying special function, this is not an issue.

# Series expansions

A first method to compute special functions is by using series 
expansions. A [Taylor 
series](https://en.wikipedia.org/wiki/Taylor_series) is a mathematically 
unambiguous procedure that allows us to rewrite a special function as a 
polynomial with infinitely many terms and factors that only involve the 
special function and its derivatives evaluated for specific values, and 
that can be shown to converge to the real special function under some 
conditions (i.e. the polynomial gets closer and closer to the real 
special function as the number of terms in the polynomial increases). 
This means that, if we are able to calculate the value of the special 
function and its derivatives in one point (e.g. $$x=0$$, where 
$$\cos(x)=1$$ and $$\sin(x)=0$$), then we can use those values to write 
down a polynomial that represents the special function in any other 
point. Since computing polynomials only requires multiplication and 
addition, this can be done numerically.

In order to reach convergence, it is important that successive terms in 
a series expansions get progressively smaller. In a Taylor series, this 
is usually guaranteed by the prefactors that involve division with large 
factorials, as long as the derivatives of the special function are well 
behaved. If this is the case, then it is also fairly straightforward to 
check how accurate your approximation for the special function value is 
during the evaluation of the series expansion: if the next term in the 
series expansions drops below the round off threshold of your computer, 
you can safely assume that your value will be accurate to almost machine 
precision.

Unfortunately, series expansions do not always work. Some functions do 
not have convergent Taylor series for arbitrary input values. And for 
those that do, the series expansion can sometimes converge very slowly, 
making the evaluation of the special function very slow.

# Recursion relations

Many special functions naturally arise when solving differential 
equations, so that in many cases, the special function itself is simply 
defined as the canonical solution of some differential equation. For the 
spherical Bessel functions of the first kind I mentioned before, the 
defining equation is

$$
\frac{d^2}{dx^2} j_n(x) + \frac{2}{x} \frac{d}{dx} j_n(x) +
\left( 1 - \frac{n(n+1)}{x^2} \right) j_n(x) = 0.
$$

In this case, the constant factor $$n$$ that appears in the defining 
equation actually enumerates a whole range of Bessel functions of 
different orders. By manipulating the defining equation, it is possible 
to derive a recursion relation between Bessel functions with different 
orders $$n$$:

$$
j_{n-1}(x) + j_{n+1}(x) = \frac{2n+1}{x} j_n(x).
$$

This kind of recursion relation can be extremely powerful when computing 
$$j_n(x)$$ for some arbitrary (integer) order $$n$$. The reason is that 
we have explicit expressions for $$j_n(x)$$ for some values of $$n$$ 
(again, we can derive these from the defining equation):

$$
j_0(x) = \frac{\sin(x)}{x},
$$

$$
j_1(x) = \frac{\sin(x)}{x^2} - \frac{\cos(x)}{x}.
$$

Starting from $$j_0(x)$$ and $$j_1(x)$$, which we can simply compute 
using the trigonometric functions already present in the easy to use 
library, we can then use the recursion relation to compute $$j_2(x)$$, 
$$j_3(x)$$... all the way up to the arbitrary value of $$n$$ we need.

Recursion relations like this one exist for many special functions, and 
they can usually be found by consulting the corresponding Wikipedia 
page. Since they usually only involve basic operations, they are very 
easy to program and reasonably accurate.

# Forward and backward recursion

When a recursion relation is applied from low orders to higher orders, 
starting from some known function expression, like above, we call this 
*forward recursion*. Unfortunately, in case of the spherical Bessel 
functions of the first kind, it turns out that the forward recursion 
relation does not always yield very accurate results, simply because of 
the round off error that occurs when repeatedly applying the relation 
for high orders $$n$$.

In this case, a backward recursion algorithm can be more stable. This
algorithm is based on the mathematical asymptotic behaviour of $$j_n(x)$$ for
high $$n$$:

$$
\lim_{n\rightarrow{}\infty{}} \left(\frac{j_n(x)}{j_{n-1}(x)}\right) =
\frac{x}{2n+1}.
$$

This means that for high enough order (say $$n=1000$$), we know very 
accurately what the ratio $$\rho{}_n(x)$$ of two successive spherical 
Bessel functions is for a specific input value $$x$$. We can also rewrite
the recursion relation for the spherical Bessel functions as

$$
\frac{1}{\rho{}_n(x)} + \rho{}_{n+1}(x) = \frac{2n+1}{x},
$$

which gives us a recursion relation for the ratio that we can use to 
*recurse down* from $$n=1000$$ to $$n=0$$. Once we reach $$n=0$$, we can 
then multiply $$j_0(x)$$ with $$\rho{}_1(x)$$ to get $$j_1(x)$$, and so 
on. This turns out to be a lot more accurate than forward recursion.

# Other methods

Since I am not a computer scientist myself, the two methods above are 
pretty much the extent of my knowledge on this subject. This means that 
I am not aware of other methods of computing special functions, although 
I am pretty sure they exist. In my experience, many special functions 
have been described very thoroughly in mathematical literature and on 
Wikipedia, and it is usually reasonably easy to find algorithms to 
compute them in these sources. These algorithms are usually based on 
some series expansion or special relations that can be derived 
mathematically and that looks horrible on paper, but is actually 
reasonably easy to code. And while most of these will involve quite a 
lot of tedious calculations that might seem very approximate, there are 
strong mathematical concepts like convergence and accompanying numerical 
checks that can guarantee accurate results for arbitrary input values, 
meaning these functions can be made as robust as our familiar $$\cos$$ 
and $$\sin$$.

So I guess the real point I am trying to make is that special functions 
are safe to use, and that they are nothing to be afraid of. Writing and 
testing your own special function implementation can be challenging at 
first, but is ultimately very straightforward, especially since there 
are so many libraries around against which you can test your 
implementation. And once you have your special function, you can treat 
it with just as much (or as little) confidence as you treat any other 
special function, like the basic trigonometric functions.
