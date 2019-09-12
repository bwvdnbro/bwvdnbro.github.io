---
layout: post
title: "Numerically inverting a matrix"
description: A summary of some things I learned this week.
date: 2019-09-12
author: Bert Vandenbroucke
tags: 
  - Code development
  - Numerical recipes
---

Suppose you have a square matrix, like the one below:
<div>
$$
A = \begin{pmatrix}
0 & 5 & 5 \\
2 & 9 & 0 \\
6 & 8 & 8
\end{pmatrix},
$$
</div>

and that you want to compute the inverse of this matrix (numerically). 
The result should be (thanks 
[WolframAlpha](https://www.wolframalpha.com/input/?i=invert+%28%5B%5B0%2C+5%2C+5%5D%2C+%5B2%2C+9%2C+0%5D%2C+%5B6%2C+8%2C+8%5D%5D%29)!):

<div>
$$
A^{-1} = \begin{pmatrix}
-\frac{4}{15} & 0 & \frac{1}{6} \\
\frac{8}{135} & \frac{1}{9} & -\frac{1}{27} \\
\frac{19}{135} & -\frac{1}{9} & \frac{1}{27}
\end{pmatrix}
$$
</div>

A first option to invert this matrix is to realise that this is a 
$$3\times{}3$$ matrix, for which 
[Wikipedia](https://en.wikipedia.org/wiki/Invertible_matrix#Inversion_of_3_%C3%97_3_matrices) 
lists a special formula. The problem of this approach is that it is not 
very general.

More generally, 
[Wikipedia](https://en.wikipedia.org/wiki/Invertible_matrix#Cayley%E2%80%93Hamilton_method) 
lists straightforward inversion algorithms that can be readily executed 
for matrices of any size (they obviously need to be square to be 
invertible). These involve *a lot* of operations, and require a lot of 
intermediate variables to be stored. Furthermore, these algorithms are 
not very stable against round off error; if the matrix contains elements 
with significantly different sizes, then accumulated round off error can 
lead to a significant loss of accuracy, or to numerical issues that 
cause the inversion algorithm to fail altogether, even if the matrix is 
strictly speaking invertible.

Fortunately, there is an elegant matrix inversion algorithm that does 
not suffer from these issues, and that furthermore can almost be 
executed *in place*, i.e. intermediate steps are mostly stored within 
the matrix itself, and only limited additional memory is required.

# PLU decomposition and matrix inversion

The basic idea of this better matrix inversion algorithm is to perform a
so called PLU decomposition of the original matrix. This means rewriting
the matrix as

$$A = P \times{} L \times{} U.$$

Here, $$P$$, $$L$$ and $$U$$ are all special matrices, that is, $$P$$ is 
a so called *permutation matrix* (a unit matrix of which some of the 
rows/columns have been swapped), $$L$$ is a *lower triangular matrix 
with a unit diagonal* (a matrix with $$1$$s along the diagonal and all 
elements above and to the right of the diagonal $$0$$), and $$U$$ is an 
*upper diagonal matrix* (I will let you guess what that means).

A possible PLU decomposition for $$A$$ (this decomposition is *not* 
unique) is:

<div>
$$
A = \begin{pmatrix}
0 & 0 & 1 \\
0 & 1 & 0 \\
1 & 0 & 0
\end{pmatrix} \times{} \begin{pmatrix}
1 & 0 & 0 \\
\frac{1}{3} & 1 & 0 \\
0 & \frac{15}{19} & 1
\end{pmatrix} \times{} \begin{pmatrix}
6 & 8 & 8 \\
0 & \frac{19}{3} & -\frac{8}{3} \\
0 & 0 & \frac{135}{19}
\end{pmatrix}.
$$
</div>

I will explain later how to obtain this decomposition. Note that the PLU 
decomposition is a variant of the so called LU decomposition, where the 
matrix is simply decomposed in a lower and upper triangular matrix, 
without the permutation matrix. When the permutation matrix is used, we 
call this an LU decomposition with partial *pivoting*. I will come back 
to this point later, as it is very important for the stability of the 
matrix inversion algorithm.

Once the matrix has been decomposed, inverting it is done by inverting 
(some) of its factors:

$$
A^{-1} = \left( P \times{} L \times{} U\right)^{-1} =
U^{-1} \times{} L^{-1} \times{} P^{-1}.
$$

There are straightforward algorithms to invert triangular matrices (I 
will show one later), and the inverse of a permutation matrix is simply 
its transpose. So using the PLU decomposition, computing the inverse 
matrix is straightforward:

<div>
$$
A^{-1} = \begin{pmatrix}
\frac{1}{6} & -\frac{4}{19} & -\frac{4}{15} \\
0 & \frac{3}{19} & \frac{8}{135} \\
0 & 0 & \frac{19}{135}
\end{pmatrix} \times{} \begin{pmatrix}
1 & 0 & 0 \\
-\frac{1}{3} & 1 & 0 \\
\frac{5}{19} & -\frac{15}{19} & 1
\end{pmatrix} \times{} \begin{pmatrix}
0 & 0 & 1 \\
0 & 1 & 0 \\
1 & 0 & 0
\end{pmatrix} \\ = \begin{pmatrix}
\frac{1}{6} & 0 & -\frac{4}{15} \\
-\frac{1}{27} & \frac{1}{9} & \frac{8}{135} \\
\frac{1}{27} & -\frac{1}{9} & \frac{19}{135}
\end{pmatrix} \times{} \begin{pmatrix}
0 & 0 & 1 \\
0 & 1 & 0 \\
1 & 0 & 0
\end{pmatrix}
$$
</div>

You can easily see that the permutation matrix (or its transpose) now 
leads to column swaps in the matrix that eventually lead to the inverse 
matrix WolframAlpha found.

However, an even more practical algorithm to compute the inverse is 
given by rewriting the above equations as

$$
\left( P^{-1} \times{} A \right)^{-1} \times{} L = U^{-1},
$$

and solving for $$\left( P^{-1} \times{} A \right)^{-1}$$. This 
algorithm is better, as it avoids having to compute the inverse of 
$$L$$, and also avoids the need to store intermediate matrices. Once you 
have computed $$\left( P^{-1} \times{} A \right)^{-1}$$, the inverse 
matrix can be found by multiplying it with the permutation matrix, as 
before:

$$
\left( P^{-1} \times{} A \right)^{-1} \times{} P^{-1} = A^{-1}.
$$

# A PLU inversion algorithm

To find the PLU decomposition, we can use the [Doolittle 
algorithm](https://en.wikipedia.org/wiki/LU_decomposition#Doolittle_algorithm). 
This algorithm can compute $$L$$ and $$U$$ and store them in the 
original matrix, and only requires one additional array with a size that 
corresponds to the size of the matrix in one dimension (3 in our 
example) that is used to store the permutation matrix (in a compact 
form). The resulting matrix will contain the upper triangular and 
diagonal elements from $$U$$, and the lower triangular elements from 
$$L$$. We don't need to store the diagonal elements of $$L$$, since we 
know they are all $$1$$.

The algorithm works as follows: we traverse the rows of $$A$$, and for 
each row $$i$$ try to find the element $$A_{ji}$$ (column $$i$$ in row 
$$j$$, with $$j \geq{} i$$) with the largest absolute value. The index 
$$j$$ is called the *pivot*. We record that index in an auxiliary *pivot 
array*. Once we have found and recorded the pivot, we swap rows $$i$$ 
and $$j$$ (note that $$i$$ and $$j$$ could be equal, in which case we 
don't do anything).

The next step is to use row $$i$$ (this will be row $$j$$, but since we 
swapped rows, this row will now be on row $$i$$ in the original matrix) 
to eliminate all rows $$j > i$$ in column $$i$$ (i.e. make these 
elements $$0$$). We can do this by multiplying the top row with 
$$\frac{A_{ji}}{A_{ii}}$$ and subtracting this from the original row. 
This is allowed, provided that we multiply the matrix with a 
transformation matrix that happens to be a lower triangular matrix. 
After we have done this for all rows, it will turn out that all these 
transformation matrices multiplied together give us $$L$$, and the 
elements $$L_{ji}$$ will simply be the factors $$\frac{A_{ji}}{A{ii}}$$
(you can check this if you really want).

So in practice, all we need to do is:
 1. Divide all elements in column $$i$$ below row $$i$$ by $$A_{ii}$$; this
gives us the elements of $$L$$ in that column.
 2. Subtract this factor times row $$i$$ from all columns $$j > i$$ in 
all rows $$k > i$$. This does the elimination for the part of the matrix 
that does not vanish after the elimination (and will give us the 
elements of $$U$$).

For our example matrix, we get the following (for clarity, I have 
indicated the pivot array in an additional column):

<div>
$$
\begin{pmatrix}
0 & 5 & 5 & | & ? \\
2 & 9 & 0 & | & ? \\
6 & 8 & 8 & | & ?
\end{pmatrix} \rightarrow{} \begin{pmatrix}
6 & 8 & 8 & | & 3 \\
2 & 9 & 0 & | & ? \\
0 & 5 & 5 & | & ?
\end{pmatrix} \rightarrow{} \begin{pmatrix}
6 & 8 & 8 & | & 3 \\
\frac{1}{3} & \frac{19}{3} & -\frac{8}{3} & | & ? \\
0 & 5 & 5 & | & ?
\end{pmatrix} \\ \rightarrow{} \begin{pmatrix}
6 & 8 & 8 & | & 3 \\
\frac{1}{3} & \frac{19}{3} & -\frac{8}{3} & | & 2 \\
0 & 5 & 5 & | & ?
\end{pmatrix} \rightarrow{} \begin{pmatrix}
6 & 8 & 8 & | & 3 \\
\frac{1}{3} & \frac{19}{3} & -\frac{8}{3} & | & 2 \\
0 & \frac{15}{19} & \frac{135}{19} & | & ?
\end{pmatrix} \rightarrow{} \begin{pmatrix}
6 & 8 & 8 & | & 3 \\
\frac{1}{3} & \frac{19}{3} & -\frac{8}{3} & | & 2 \\
0 & \frac{15}{19} & \frac{135}{19} & | & 3
\end{pmatrix}
$$
</div>

You can recognise the $$L$$ and $$U$$ matrices I showed before.

Inverting the upper diagonal matrix $$U$$ is reasonably straightforward. 
First of all, it can be shown that the inverse of an upper/lower 
triangular matrix is always an upper/lower triangular matrix too. This 
immediately means that the diagonal elements of the inverted matrix are 
simply the inverse of the diagonal elements of the original matrix. If 
we denote the elements of the original matrix $$U$$ with upper case 
$$U_{ij}$$, and the elements of the inverse matrix $$U^{-1}$$ with lower 
case $$u_{ij}$$, then we need to solve the following equation:

<div>
$$
\begin{pmatrix}
u_{11} & u_{12} & u_{13} \\
0 & u_{22} & u_{23} \\
0 & 0 & u_{33}
\end{pmatrix} \times{} \begin{pmatrix}
U_{11} & U_{12} & U_{13} \\
0 & U_{22} & U_{23} \\
0 & 0 & U_{33}
\end{pmatrix} \\= \begin{pmatrix}
u_{11} U_{11} & u_{11} U_{12} + u_{12} U_{22} & u_{11} U_{13} + u_{12} U_{23} + u_{13} U_{33} \\
0 & u_{22} U_{22} & u_{22} U_{23} + u_{23} U_{33} \\
0 & 0 & u_{33} U_{33}
\end{pmatrix} = \begin{pmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{pmatrix},
$$
</div>

this is simply the definition of the inverse matrix.

You can see a pattern emerge that allows you to solve for the $$u_{ij}$$ 
recursively, starting from the top row and working down (a so called 
*forward substitution*). This algorithm only requires multiplication and 
subtraction, and a single division per element that only involves the 
diagonal elements of $$U$$ (this will be important later). 
Mathematically, we can write down the following formula for the elements 
$$u_{ij}$$ ($$j > i$$):

$$
u_{ij} = -\frac{1}{U_{jj}} \sum_{k=i}^j u_{ik} U_{kj}.
$$

You can check that a forward substitution algorithm can be used *in 
place*, i.e. the $$u_{ik}$$ and $$U_{kj}$$ in this formula are all 
elements of the corresponding separate matrices (despite being stored in 
the same matrix), as long as you (a) store $$u_{ij}$$ in a temporary 
variable during the summation, and (b) traverse the matrix from top to 
bottom (from small to large $$i$$), and within a row from left to right 
(from small to large $$j$$). I will leave it up to you to check that 
this indeed yields the same inverse matrix $$U^{-1}$$ as I showed 
before.

The next step is to solve the equation $$\left(P^{-1} \times{} A 
\right)^{-1} \times{} L = U^{-1}$$. If we denote the elements of the 
matrix $$\left(P^{-1} \times{} A\right)^{-1}$$ with $$a_{ij}$$, the 
elements of $$L$$ with $$L_{ij}$$, and the elements of $$U^{-1}$$ with 
$$u_{ij}$$ as before, we get

<div>
$$
\begin{pmatrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & a_{33}
\end{pmatrix} \times{} \begin{pmatrix}
1 & 0 & 0 \\
L_{21} & 1 & 0 \\
L_{31} & L_{32} & 1
\end{pmatrix} \\ = \begin{pmatrix}
a_{11} + a_{12} L_{21} + a_{13} L_{31} & a_{12} + a_{13} L_{32} & a_{13} \\
a_{21} + a_{22} L_{21} + a_{23} L_{31} & a_{22} + a_{23} L_{32} & a_{23} \\
a_{31} + a_{32} L_{21} + a_{33} L_{31} & a_{32} + a_{33} L_{32} & a_{33}
\end{pmatrix} = \begin{pmatrix}
u_{11} & u_{12} & u_{13} \\
0 & u_{22} & u_{23} \\
0 & 0 & u_{33}
\end{pmatrix}
$$
</div>

If you look carefully, you can see a clear pattern: the last column of 
the matrix $$\left(P^{-1} \times{} A\right)^{-1}$$ simply equals the 
last column of $$U^{-1}$$, while the other columns can be computed using 
only values from $$L$$, $$U$$ and the columns that were already 
computed, if we traverse the columns from right to left. Again, we can 
use an in place algorithm, as long as we make sure to store the new 
column in an intermediate array while it is being computed, as the 
calculation for each element of the new column requires all elements of 
$$L$$ for that same column, and we don't want to overwrite them before 
we finished the column.

Mathematically, this backward substitution algorithm can be expressed as 
follows:

$$
a_{ij} = u_{ij} - \sum_{k = j+1}^N a_{ik} L_{kj}.
$$

If we were to apply this algorithm to the intermediate matrix, we would 
end up with the same intermediate matrix I showed before, and we also 
still have the pivot array:

<div>
$$
\begin{pmatrix}
\frac{1}{6} & 0 & -\frac{4}{15} & | & 3 \\
-\frac{1}{27} & \frac{1}{9} & \frac{8}{135} & | & 2 \\
\frac{1}{27} & -\frac{1}{9} & \frac{19}{135} & | & 3
\end{pmatrix}
$$
</div>

The only thing left to do is use the pivot array to rearrange the 
columns. Since mathematically we have to use the inverse of the 
permutation matrix, we will need to use a backwards algorithm, were we 
start from the last column, and then swap that column with the column 
with the index given by the last element of the pivot array, and so on:

<div>
$$
\begin{pmatrix}
\frac{1}{6} & 0 & -\frac{4}{15} & | & 3 \\
-\frac{1}{27} & \frac{1}{9} & \frac{8}{135} & | & 2 \\
\frac{1}{27} & -\frac{1}{9} & \frac{19}{135} & | & \checkmark{}
\end{pmatrix} \rightarrow{}
\begin{pmatrix}
\frac{1}{6} & 0 & -\frac{4}{15} & | & 3 \\
-\frac{1}{27} & \frac{1}{9} & \frac{8}{135} & | & \checkmark{} \\
\frac{1}{27} & -\frac{1}{9} & \frac{19}{135} & | & \checkmark{}
\end{pmatrix} \rightarrow{}
\begin{pmatrix}
-\frac{4}{15} & 0 & \frac{1}{6} & | & \checkmark{} \\
\frac{8}{135} & \frac{1}{9} & -\frac{1}{27} & | & \checkmark{} \\
\frac{19}{135} & -\frac{1}{9} & \frac{1}{27} & | & \checkmark{}
\end{pmatrix}
$$
</div>

The same inverse matrix WolframAlpha gave us before!

# Why is this algorithm good?

The key reason that this PLU inversion algorithm is preferable over 
other algorithms is the low number of divisions that is involved, 
combined with the partial pivoting. In the entire algorithm above, we 
need to divide by only three different numbers: the diagonal elements in 
the upper triangular matrix $$U$$. Divisions are always a bit 
problematic for numerical algorithms, as their round off error is 
theoretically unbound, unlike the round off error for additions, 
subtractions and multiplications that is guaranteed to stay within some 
limits. Every division could hence lead to a serious loss of accuracy, 
especially if the numbers in the denominator of the division are small.

In a naive matrix inversion algorithm, the number of divisions might be 
lower, but the denominators of the divisions are fixed at the start and 
could hence be anything. In the PLU inversion algorithm we have some 
control over the denominators of the divisions, as the elements on the 
diagonal of $$U$$ will depend on the permutations we perform during the 
PLU decomposition. In other words: we can choose which numbers we want 
to use for divisions when choosing the pivot for a row. If we choose the 
pivot wisely (the element in the column with the highest absolute value 
like we did), then we can significantly reduce the round off error that 
occurs during the later inversion of the $$U$$ matrix. If this still 
leads to significant round off error, then this can only be because the 
matrix elements are very small, and numerical inversion would be tricky 
with any algorithm.

As already mentioned, a second advantage of the PLU inversion algorithm 
is that it requires very little memory: almost all intermediate steps 
can simply be stored in the matrix itself, and we only require two 
additional arrays with the length of the 1D size of the matrix: the 
pivot array and the temporary array required to store new columns during 
the third step of the algorithm. This is especially important for large 
matrices.
