---
layout: post
title: "Unit testing: a practical example"
description: >-
  An example of how to use unit tests during the development of a new function.
date: 2019-12-20
author: Bert Vandenbroucke
tags:
  - Code development
---

Last week I explained how to use CTest and how to configure CMake to 
automatically compile and run unit tests. I also mentioned how great and 
useful unit tests are, especially when you use them during the 
development process. In this post, I want to illustrate this using a 
practical example.

# The example function

The code I will be developing is a simple function to compute the 
Clebsch-Gordan coefficient $$C^{NM}_{n_1m_1n_2m_2} = \langle{} 
n_1m_1n_2m_2 | NM \rangle{}$$, a coefficient that appears naturally in a 
certain expansion in spherical coordinates, most notably when dealing 
with angular momentum eigen states in quantum mechanics. The details of 
how these coefficients arise and are defined are not really important.

These coefficients are only defined for $$n_1\geq{}0$$, $$n_2\geq{}0$$, 
$$N\in{}[|n_1-n_2|,n_1+n_2]$$, $$m_1 \in{} [-n_1,n_1]$$, $$m_2 \in{} 
[-n_2,n_2]$$ and $$M \in{} [-N,N]$$.

We can compute the Clebsch-Gordan coefficients using two recursion 
relations (given on 
[Wikipedia](https://en.wikipedia.org/wiki/Clebsch%E2%80%93Gordan_coefficients#Recursion_relations)):

$$
C_{+}(n_1, m_1-1) C^{NN}_{n_1(m_1-1)n_2(N-m_1+1)} +
      C_{+}(n_2, N-m_1) C^{NN}_{n_1m_1n_2(N-m_1)} = 0
$$

$$
C_{-}(N, M) C^{N(M-1)}_{n_1m_1n_2(M-1-m_1)} =
      C_{-}(n_1,m_1+1) C^{NM}_{n_1(m_1+1)n_2(M-m_1-1)} +
        C_{-}(n_2, M-m_1) C^{NM}_{n_1m_1n_2(M-m_1)},
$$

with

$$
C_{+}(n, m) = \sqrt{(n - m) (n + m + 1)}
$$

and

$$
C_{-}(n, m) = \sqrt{(n + m) (n - m + 1)}.
$$

The coefficients are only fixed by also assuming the following 
normalisation relation:

$$
\sum_{m_1=-n_1}^{n_1} \sum_{m_2=-n_2}^{n_2}
      \left(C^{NM}_{n_1m_1n_2m_2}\right)^2 = 1,
$$

and by imposing a sign convention, like the Condon-Shortley convention:

$$
C^{NN}_{n_1n_1n_2(N-n_1)} > 0.
$$

Lastly, all coefficients $$M\neq{}m_1+m_2$$ are defined to be zero.

[Wikipedia](https://en.wikipedia.org/wiki/Table_of_Clebsch%E2%80%93Gordan_coefficients) 
further also lists some example values of the coefficients for different 
input values, most notably:

$$C^{22}_{1111} = 1$$

$$C^{21}_{1110} = \sqrt{\frac{1}{2}}$$

$$C^{1-1}_{101-1} = \sqrt{\frac{1}{2}}$$

$$C^{22}_{1121} = \sqrt{\frac{1}{3}}$$

(there are many more, but these ones do a good job at testing various 
combinations of parameter values).

# The starting point: program skeleton and unit test

The function we want to implement will take 6 integers as input: the 
values of the parameters $$n_1,m_1,n_2,m_2,N,M$$. It will return a 
single floating point value that contains the corresponding 
Clebsch-Gordan coefficient. Internally, we will need to use the 
recursion relations to compute this value.

Before we start implementing this function, we simply write a function 
skeleton in the file where we want to implement this function, e.g. 
`ClebschGordan.hpp`:

```
#include <cassert>
#include <cmath>

double get_clebsch_gordan_coefficient(const int_fast32_t n1,
                                      const int_fast32_t m1,
                                      const int_fast32_t n2,
                                      const int_fast32_t m2,
                                      const int_fast32_t N,
                                      const int_fast32_t M) {
  assert(n1 >= 0);
  assert(n2 >= 0);
  assert(N >= 0);
  assert(N <= n1 + n2);
  assert(N >= std::abs(n1 - n2));
  assert(m1 <= n1);
  assert(m1 >= -n1);
  assert(m2 <= n2);
  assert(m2 >= -n2);
  assert(M <= N);
  assert(M >= -N);

  if(m1 + m2 != M){
    return  0.;
  }

  return 0.;
}
```

This prototype does not do anything yet, but it already outlines what 
the rest of the code will see once the function is implemented: a 
function that takes six arguments and spits out a single value. We also 
already hard-coded some of the logic: we added assertions that check the 
input values are sensible (these assertions are optimised out by the 
compiler if optimisations are enabled, but are very useful for 
debugging), and already dealt with the trivial case where the 
coefficients are guaranteed to be zero.

Before moving out to actually implementing the rest of the function, we 
need to set up the unit test. As explained before, the unit test is a 
simple program that calls the function we want to write:

```
#include "ClebschGordan.hpp"

#define assert_values_equal_rel(a, b, tol)                                     \
  if (std::abs((a) - (b)) > tol * std::abs((a) + (b))) {                       \
    fprintf(stderr,                                                            \
        "Assertion failed: %s (%g) != %s (%g) (relative_difference: %g)", #a,  \
        a, #b, b, std::abs((a) - (b)) / std::abs((a) + (b)));                  \
    abort();                                                                   \
  }

int main(int argc, char **argv){

  const double C111122 = get_clebsch_gordan_coefficient(1, 1, 1, 1, 2, 2);
  assert_values_equal_rel(double(C111122), 1., 1.e-10);

  const double C111021 = get_clebsch_gordan_coefficient(1, 1, 1, 0, 2, 1);
  assert_values_equal_rel(double(C111021), std::std::sqrt(0.5), 1.e-10);

  const double C101m11m1 = get_clebsch_gordan_coefficient(1, 0, 1, -1, 1, -1);
  assert_values_equal_rel(double(C101m11m1), std::std::sqrt(0.5), 1.e-10);

  const double C112122 = get_clebsch_gordan_coefficient(1, 1, 2, 1, 2, 2);
  assert_values_equal_rel(double(C211122), std::std::sqrt(1. / 3.), 1.e-10);

  return 0;
}
```

To configure and compile this program, you can use the unit test macro I 
introduced in the previous post.

This test program is very simple: it just calls the function we are 
going to write with the four input values from Wikipedia and checks that 
the values are what they should be. Since we expect our values to only 
be accurate up to some round off error, we cannot use an assertion to 
check that the values match exactly, but instead provide our own 
assertion macro that checks the values are equal up to some allowed 
relative error.

The next to do is to actually compile and run the unit test. This should 
fail, since we did not implement the function yet. By running it and 
making sure it fails, we know that our unit test works, and that it 
hence will be able to fail if our future implementation is wrong.

# Step 1: recursion start point

Since the Clebsch-Gordan coefficients are computed from the recursion 
relations, we will use a step-wise strategy to compute them:
 - First, we use the Condon-Shortley convention to set the value of 
$$C^{NN}_{n_1n_1n_2(N-n_1)}=1$$.
 - Second, we use the first recursion relation to compute all other 
non-trivial coefficients $$C^{NN}_{n_1m_1n_2(N-m_1)}$$.
 - Once this is done, we can normalise the coefficients we just computed 
using the normalisation condition.
 - Finally, we use the second recursion relation to compute all other 
non-trivial coefficients, up to the $$M$$ value that we need.
 - Once we have all coefficients for the requested $$M$$, we simply 
return the requested coefficient.

This step-wise algorithm can be implemented in steps too: since our 
first test value correspond to an $$M=N$$ coefficient, we only need the 
first step in our algorithm to work for this test to pass. We can hence 
start by implementing this step:

```
// assertions

const int_fast32_t nsize = 2 * n1 + 1;
std::vector<double> C((N + 1) * nsize, 0.);
 
C[N * nsize + n1 + n1] = 1.;
double norm = 1.;
for (int_fast32_t m1temp = n1 - 1; m1temp >= std::max(-n1, N - n2 - 1);
     --m1temp) {
 
  // Cplus = sqrt((n - m) * (n + m + 1))
  const double Cplusn2m2 =
    std::sqrt((n2 - N + m1temp + 1.) * (n2 + N - m1temp));
  const double Cplusn1m1m1 = std::sqrt((n1 - m1temp) * (n1 + m1temp + 1.));
  const double Cn1m1p1 = C[N * nsize + m1temp + 1 + n1];
  const double Cnext = -Cplusn2m2 * Cn1m1p1 / Cplusn1m1m1;
 
  // make sure the coefficient is not NaN
  assert(Cnext == Cnext);
 
  C[N * nsize + m1temp + n1] = Cnext;
 
  // add its norm contribution
  norm += Cnext * Cnext;
}

// compute the inverse norm
const double inverse_norm = 1. / std::sqrt(norm);
// normalise the coefficients we just computed
for (int_fast32_t m1temp = n1; m1temp >= std::max(-n1, N - n2 - 1);
     --m1temp) {
  C[N * nsize + m1temp + n1] *= inverse_norm;
 
  // make sure they are still not NaN
  assert(C[N * nsize + m1temp + n1] == C[N * nsize + m1temp + n1]);
}

return C[M * nsize + m1 + n1];
```

As you can see, I added a lot of additional assertions to the 
calculation to make sure that the coefficients are well-behaved. The 
return in the end assumes that all coefficients for the $$M$$ value that 
was requested have been computed and are stored in a specific order.

If you now run the unit test again, you should manage to pass the first 
test, since the first test coefficient does not require the second 
recursion relation. The version here should work (since I actually 
tested it), but any other faulty version of the initial step will likely 
fail; there is no point in moving to the second step of the algorithm as 
long as the first step does not work.

# Step 2: second recursion relation

The second recursion relation allows us to compute all coefficients with 
$$M'=M-1$$ if we already have the coefficients for $$M$$. If the 
coefficients for $$M$$ have been normalised (this is what we did for 
$$M=N$$), then the $$M'$$ coefficients will also be normalised, so we 
don't need to worry about that any more. Since we only want to compute a 
single coefficient, we can stop recursing down as soon as we reach the 
right $$M$$ level.

An example implementation of the second recursion step could read

```
// step 1

// now recurse down in M for the other coefficients (we don't explicitly
// compute coefficients for M < 0, as we can link those to M > 0
// coefficients.
for (int_fast32_t Mtemp = N - 1; Mtemp >= 0; --Mtemp) {

  // precompute the inverse ladder coefficient, since it does not change
  const double Cmin_inverse = 1. / std::sqrt((N + Mtemp + 1.) * (N - Mtemp));

  // the conditions for a valid element with M = Mtemp are
  // m1temp >= -n1, m1temp <= n1
  // m2temp = Mtemp - m1temp >= -n2 => m1temp <= Mtemp + n2
  // m2temp = Mtemp - m1temp <= n2 => m1temp >= Mtemp - n2
  for (int_fast32_t m1temp = std::max(-n1, Mtemp - n2);
       m1temp <= std::min(n1, Mtemp + n2); ++m1temp) {

    // note that the conditions above do not guarantee that the elements
    // with M = Mtemp + 1 in both terms exist
    // we hence need an additional check for these
    const int_fast32_t m2temp = Mtemp - m1temp;
    double term1 = 0.;
    if (m1temp + 1 <= n1) {
      // Cmin = sqrt((n + m) * (n - m + 1))
      const double Cminn1m1p1 = std::sqrt((n1 + m1temp + 1.) * (n1 - m1temp));
      const double Cn1m1p1 = C[(Mtemp + 1) * nsize + m1temp + 1 + n1];
      term1 = Cminn1m1p1 * Cn1m1p1;
    }
    double term2 = 0.;
    if (m2temp + 1 <= n2) {
      // Cmin = sqrt((n + m) * (n - m + 1))
      const double Cminn2m2p1 = std::sqrt((n2 + m2temp + 1.) * (n2 - m2temp));
      const double Cn2m2p2 = C[(Mtemp + 1) * nsize + m1temp + n1];
      term2 = Cminn2m2p1 * Cn2m2p2;
    }

    // now compute the element with M = Mtemp
    const double Cnext = Cmin_inverse * (term1 + term2);

    // check it is not NaN
    assert(Cnext == Cnext);

    C[Mtemp * nsize + m1temp + n1] = Cnext;
  }
}
```

Note that we stop recursing for $$M=0$$, for reasons that I will explain 
shortly. Combined with the same return expression as before, this 
version should now be able to move past the second unit test. It will 
fail for the third unit test, since that one requires a negative $$M$$ 
value. Again, there is no point moving on with the implementation as 
long as the second unit test keeps failing, which should again not be 
the case for the example implementation given here.

# Step 3: negative $$M$$

One useful relation I didn't mention before that helps us with the 
computation is the following symmetry relation:

$$
C^{NM}_{n_1m_1n_2m_2} = (-1)^{N-n_1-n_2} C^{N-M}_{n_1-m_1n_2-m_2}.
$$

We can exploit this relation to compute the coefficients for negative 
$$M$$ from the ones for positive $$M$$, as follows:

```
// step 1

// step 2

if (M >= 0) {
  return C[M * nsize + m1 + n1];
} else {
  // compute N - n1 - n2
  const int_fast32_t Nmn1mn2 = N - n1 - n2;
  // (-1)^{N - n1 - n2} is +1 if (N - n1 - n2) is even and -1 if it is odd
  int_fast8_t sign;
  if (Nmn1mn2 % 2 == 0) {
    sign = +1;
  } else {
    sign = -1;
  }
  // now return the right element with the right sign
  return sign * C[-M * nsize + n1 - m1];
}
```

This replaces the original return expression. That's it! We should now 
be able to pass all unit tests: the third test that requires a negative 
$$M$$ value, and the fourth test that is only there because there are 
some subtleties involved with computing $$n_1 < n_2$$ coefficients 
(subtleties that I dealt with in my example implementation by not really 
dealing with them...).

# Further testing

The tests we have done so far are fine to check a proper implementation 
of the various steps in the recursive algorithm, since they cover all 
possible algorithmic paths through the code. These tests have helped me 
a lot during the actual implementation of the algorithm, mainly to 
figure out subtleties like the conditions in the second step.

Of course, if I was only interested in using Clebsch-Gordan coefficients 
for low values of the input parameters, I could simply hard-code the 
values from Wikipedia; the reason I implemented this function is to be 
able to handle much larger values of $$n_1,m_1,n_2,m_2,N,M$$. Testing 
these is no longer strictly possible, but it is still possible to test 
some aspects of the calculation:
 - We can test that all values are nicely behaved, i.e. are not `NaN` or 
infinite.
 - We can test that the normalisation relation and symmetry relation 
still hold. The latter should trivially hold if you use my example 
implementation, as it exploits this relation explicitly.

I will not show any example code to perform these tests (it is quite 
tedious code), but it should be straightforward enough to implement this 
(my own unit test version does this kind of tests).

# Further improvements

Apart from the additional tests, there are also additional improvements 
possible:
 - The example code always recurses down to $$M=0$$, which is not 
strictly necessary if a coefficient $$|M|>0$$ is requested.
 - The example code uses quite a lot of storage space, which is also not 
strictly necessary.

Now that the unit test is in place and passes, it is quite easy to 
*refactor* the code: change the way it works without changing what it 
does. The latter means that the unit tests should keep passing 
regardless of the changes you make. Having a unit test that covers all 
possible code paths provides a lot of reassurance while making these 
changes: you should notice it immediately if a change you made has 
unexpected consequences.

# Conclusion

I hope this simple (?) example has shown you some of the advantages of 
using unit tests during code development. Note that it is important to
 1. Test your unit tests. Always make sure that your unit tests fail for 
wrong output. Don't trust a unit test that has never failed.
 2. Try to come up with incremental unit tests for step-wise algorithms. 
This means that you can implement your functionality in stages. Having 
tests for small portions of an algorithm reduces the number of lines of 
code that can contain bugs, which usually makes debugging easier.
 3. Make sure that you test functionality in a realistic regime, i.e. 
for realistic input values. Some problems are only triggered for large 
values or for input values with a specific sign or specific internal 
order (e.g. $$n_1 < n_2$$). Think about what kind of tests are possible 
in these regimes; having mathematical relations like e.g. a 
normalisation condition that needs to be satisfied is always helpful.
 4. Don't be afraid of using assertions on top of our unit tests. These 
will test your code in real-life (debugging) runs and are more likely to 
capture problems that are only caused by e.g. large values. Assertions 
are optimised out when you enable compiler optimisations, so there is 
literally no overhead associated with them.
 5. It is also a good idea to have assertions for function input values. 
A lot of algorithms will fail if the input you provide is wrong, and 
having these tests will immediately tell you if the bug is in your 
function or in the function that calls your function.
 6. Almost all complex algorithms can be further optimised after you 
have written a first version, and having working unit tests in place 
makes the refactoring process required to make those optimisations a lot 
easier.
