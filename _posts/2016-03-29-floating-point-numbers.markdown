---
layout: post
title:  "1. Practical and theoretical numbers"
date:   2016-03-29 16:30:18 +0200
categories: numerical-methods
---

## Introduction

{% newthought "Mathematicians operate on numbers" %} for which we do not have
proper representations. Consider, for example, number {% m %}\pi {% em %} or
{% m %}\sqrt{2}{% em %}: we cannot write full fractional representation of any
of them. Furthermore, let {% m %}k = \pi / \sqrt{2}{% em %}; we cannot write
neither fractional representation of $$k$$, nor it could be represented via
either {% m %}\sqrt{2}{% em %} or {% m %}\pi{% em %}. We will call such numbers
*theoretical*.

In contrast, *practical* numbers are those we can actually use for
calculations{% sidenote "one" "R.W. Hamming in \"Numerical methods for
scientists and engineers\" uses word *real* numbers,
but this may create confusion with numbers from set $$\\mathbb{R}$$"%}.
There are numerous ways how we can define these practical numbers: they can be
just usual decimals or binaries, or, perhaps, something more esoteric
as having numbers such as

{% math %} x = \sum_{i=-A}^{+B} x_i \sqrt{2^i} {% endmath %}

for some positive values {% m %}A{% em %} and {% m %}B{% em %}. Of course,
since we want numbers to be practical, we'll need to settle on something less
idiosyncratic.

Since we are going to use a computational machine (computer), it makes
sense to look at numbers as they are represented there. For
illustration purposes we will make a couple of changes. First, we will use
decimal notation instead of binary, as it is more familiar to most people.
Secondly, the number of digits to represent the number will be limited.

## Intergers and fixed-point numbers

{% newthought "Without going into too deep details"%} about *integers*, we will
essentially equate a theoretical integer and a practical
integer. While in some systems (for example, C or Fortran) integers
might be capped to some value, most modern systems (Lisp, Python,
Smalltalk, F#, Haskell, etc.) offer arbitrary large integers (still
capped by the size of the available memory, but large enough for us
not to worry): 12356, -4567, 572168632689567.

*Fixed-point* numbers are essentially the same as integers, but with the
decimal point put in the middle (usually, considering last four digits
to be fractional part of a number): 1235.600, -45.6700,
57216863268.9567. The point does not move, thus the smallest positive
number would be 0.001{% sidenote "mn-electron" "This is way too large, for
example, to represent the mass of an electron, $$9.11\\times10^{-31}$$ kg"%}.
As integers, these numbers can be arbitrary large. However, the representation
of something as large as the mass of Earth{% sidenote "sn-earth"
"$$5.97\\times10^{24}$$ kg"%} would lead to prohibitively slow arithmetic
operations. One area where fixed-point fractional numbers are successfully used
is the finance: the number representation guarantees that no money can be
made out of thin air, neither the money can disappear there (when
people do make such things, they have to use different methods).

## Floating-point numbers

{% newthought "Engineers and scientists"%} have to rely on *floating-point*
numbers for their calculations. First, floating-point numbers cover a large range
of values{% sidenote "sn-float-range" "On modern personal computers they range
from about $$10^{-300}$$ to $$10^{300}$$ for both positive and negative
numbers"%} enabling to use them from quantum mechanics to statistical physics
and cosmology. Secondly, the operations on these numbers are relatively fast:
while they are slower than on (capped) integers, they are much faster compared
to arbitrary large integers or fixed-point numbers. As a trade off,
floating-pont numbers offer only limited precision. For example, number
{% m %}\pi{% em %} (which is irrational and hence has infinitely long tail
of digits without cycles) would be approximated by
{% m %}3.141592653589793{% em %} and the tail of $$238462643383279\ldots$$ is
dropped.

Representation of floating-points numbers is close to a scientific notation.
For example, Avogadro's number $$N_A = 6.022\times10^{23}$$ will have the
following representation:

{% math %}
+0.6022\times10^{24}.
{% endmath %}

Here 0.6022 is *mantissa* (also referred as significand) and 23 is *exponent*.
For illustration purposes, following Hamming, we will limit mantissa to three
digits after the point and exponent to be in the range $$-9$$ to
$$9$$. Notice leading 0 before the point: this is done for convenience so that
we can operate directly on fractional part of number. Also notice that the same
number can have a different representation:

{% math %}
+0.06022\times10^{25}.
{% endmath %}

Numbers for which the fractional part of mantissa starts with zero are called
*subnormal* numbers. We will follow the convention of most computer systems by
not allowing subnormal numbers. Thus, fractional part of mantissa must begin
with a non-zero digit.

There is no need to store leading zero or base 10 of the
representation. But we do need to make a provision for the sign of the
number{% sidenote "sn-exp-sing" "Strictly speaking we need to take care of the
sign of the exponent as well. However, actual way of storing the exponent avoids
this problem, but we will omit this detail"%}. For example, the approximation of
number $$\pi$$ can written as:

{% math %}
0.3141592653589793 \times 10^1 = (+,3141592653589793,1)
{% endmath %}

To keep the notation simple and not allow ourselves to be distracted by
unimportant details we will limit mantissa to contain only three digits and
exponent to fall in range $$-9$$ to $$9$$.

If a number requires exponent to be higher (or lower) the limit, we encounter
*exponent overflow* (*underflow*). In computer systems we have two options: we
either signal an error (*raise an exception* using the terms of most modern
programming languages) and let the user decide what to do or we can silently
assign an *infinity* value (or zero in case of underflow). While the latter
might not be the best option from system design point of view, most of the
systems implement it due to its low computational cost and the fact that it
produces sensible (in practical terms) results. Negative and positive
infinities, will be $$\pm\infty = (\pm, 999, 9)$$. Thus, the largest (by
magnitude) proper number is $$(+,998,9)$$.

To represent number zero we will require that both mantissa and exponent of the
number are zero: $$(\pm,000,0)$$. Notice, we can have positive and negative
zeros. We will require that they are arithmetically equivalent.

Before looking into operations on floating-point numbers it's worth
understanding some of their properties. First, there exists the
smallest positive floating-point number: $$(+,100, -9)=0.1\times10^{-9}$$.
Between $$0.1\times10^{-9}$$ (including) and
$$0.1\times10^{-8}$$ (excluding) there are exactly 900 numbers (with
mantissas ranging from 100 to 999). These numbers are uniformly distributed with
the interval $$10^{-12}$$. However, numbers in interval $$0.1\times10^{-8}$$
(including) and $$0.1\times^{-7}$$ (excluding) are uniformly distributed with
the interval 10 times larger, $$10^{-11}$$. Continuing this process, one
concludes that 899 numbers in the interval $$0.1\times^{9}$$ (including) to
$$0.999\times{9}$$ (excluding) are distributed with the interval $$10^{6}$$.
However, floating-point numbers as whole are not uniformly distributed.

There is one case in which subnormal numbers can be sensibly allowed: consider
the following number close to zero, $$(+,001, -9)$$. It is hundred times smaller
than a proper smallest number. However, it only has one digit of precision. To
keep our discussion simple we will not allow even these subnormal numbers.

## Arithmetic on floating-point numbers

Let's consider the addition of $$0.360$$ and $$12.5$$:

{% math %}
(+,360, 0) + (+,125,2)
{% endmath %}

To add them we need to equate their exponents
first (by extending one of the mantissas):

{% math %}
(+,360,0) + (+,125,2) = (+,00360,2) + (+,125,2)
= (+,12860, 2)
{% endmath %}

Now we have the excess of digits in mantissa: last two digits need to be
dropped. There are two possible ways of dropping: either just truncate
them, or round them, in which case the last significant digit might
increase if the following digit is $$\geq 5$$. Simple truncation,
although faster, leads to a higher rate of precision loss. Considering
our low starting precision, we will use rounding. Thus the final
result becomes:

{% math %}
(+,129, 2) = 12.9
{% endmath %}

Another caveat of the addition is *mantissa overflow*. Consider, the following
example:

{% math %}
4.0 + 7.0 = (+,400,10 1) + (+,700, 1) = (+,1'100, 1)
{% endmath %}

Here mantissa became too large (notice, not just too long due to too high
precision required, but actually too large): now it represents number 1.100,
i.e. there is one *before* the point. This overflow needs to be consumed by the
exponent:

{% math %}
(+,1'100, 1) = (+, 110, 2)
{% endmath %}

Obviously, if two numbers are very large, their sum might exceed
the largest possible number in which case their result will become infinite:

{% math %}
(+,700,9) + (+,600,9) = (+,999,9) = +\infty
{% endmath %}

Multiplication is performed similarly, but exponents are added and the
mantissas are multiplied{% sidenote "sn-man-mul" "Notice we need to take care of
the fact that mantissas are fractional parts. For example in multiplication
$$0.2\\times0.3=0.06$$ mantissa terms will multiply as $$200\\times300=60000$$.
This result needs to be adjusted by dividing it by $$10^{3}$$, i.e. 10 raised to
the power of mantissa width."%}:

{% math %}
\begin{align*}
4.0\times 50.0 = (+,400,10 1) \times (+,500,2) & = (+,200000, 1+2-3)\\
& = (+, 200, 3) = 200.0
\end{align*}
{% endmath %}

We allow in our calculations to produce a technical mantissa overflow. However,
strictly speaking, the overflow is not possible as we multiply two numbers less
than one. Mantissa underflow, however, is possible in principle:

{% math %}
3.0\times 20.0 = (+,300,1) \times (+,200,2) = (+, 060,3) = (+, 600, 2) = 60.0.
{% endmath %}

Once again, the exponent is used to promote the first non-zero digit in mantissa
to the first position. Furthermore, the underflow of
multiplication of two very small numbers can lead to the zero result:

{% math %}
(+,200,-8)\times(+,500,-9) = (+,000,0)
{% endmath %}

Subtraction is very similar to addition, but it contains a subtle
danger. Consider the following expression:

{% math %}
1.0 - 0.999 = (+,100,1) - (+,999,0) = (+,0001,1) = (+,100,-2) = 0.001
{% endmath %}

Notice what has happened: we subtracted two numbers with the precision
of up to three digits, but the result has a precision of only one
digit. This is called *loss of significance*.

Floating-point numbers division is similar to multiplication, except
that exponents are subtracted and mantissas are divided. The caveat
with division is when a number is divided by a very small number
(similar to multiplication by a very large number): since the
magnitude increases significantly, the roundoff error is blown up as
well. The worst case scenario is when the significance is lost due to
subtraction and then this number is divided by something very
small. Consider, for example, approximating the derivative by finite
difference:

{% math %}
D[f](x) = \frac{df}{dx}(x) = \frac{f(x+h) - f(x-h)}{2h} +O(h^2)
{% endmath %}

Operated on theoretical numbers, this approximation should reduce the
error quadratically as $$h$$ gets smaller. Unfortunately, to see this effect
three digits in mantissa is not enough: for this example mantissa is extended to
six digits. To be practical, let's consider function $$y=\exp{x}$$. The
derivative of this function at $$x=0$$ is known, $$D[\exp{}](0) = 1$$. Thus,
the difference

{% math %}
\Delta = \frac{\exp{h} - \exp{(-h)}}{2h} - 1
{% endmath %}

measures the overall error of derivative approximation at $$x=0$$. First, if
$$h$$ is taken as $$10^{-n}$$ for $$n=0,1,2,\ldots$$ the
corresponding values of $$\Delta$$ are:

{% marginnote "tab-der-1" "The effect of decreasing the step of approximation on
overall error for the modeled floating-point numbers with mantissa width = 6"%}

| $$n$$                       | $$\Delta$$                 |
|----------------------------:+---------------------------:|
| 0                           | $$+0.166670\times10^{0}$$  |
| -1                          | $$+0.165000\times10^{-2}$$ |
| -2 -- -5                    | $$+0.000000\times10^{0}$$  |
| < -6                        | $$-0.100000\times10^{1}$$  |

When the value of $$h$$ becomes too small the difference $$\exp{h}-\exp{(-h)}$$
can no longer be captured by the limited precision numbers. Thus, the error
becomes $$-1$$. In real computer systems the promotion of mantissa from
underflow after subtraction may cause some garbage appear in the tail of the
mantissa. For example, using `SINGLE-FLOAT` values in Common Lisp,
$$\exp{(10^{-4})}$$ and $$\exp{(-10^{-4})}$$ evaluate to $$1.0001$$ and $$0.9999$$
(with trailing zeros). Their difference, however is not $$0.0002$$ but
$$0.00020003319$$. This extra garbage causes slightly faster deterioration of
the resulting error.

{% marginnote "tab-der-2" "The effect of the decreasing step of approximation on
overall error for `SINGLE-FLOAT` Common Lisp numbers"%}

| $$n$$                       | $$\Delta$$                  |
|----------------------------:+----------------------------:|
| 0                           | $$0.17520118\times10^{0}$$  |
| -1                          | $$0.16676188\times10^{-2}$$ |
| -2                          | $$0.1692772\times10^{-4}$$  |
| -3                          | $$0.1692772\times10^{-4}$$  |
| -4                          | $$0.16593933\times10^{-3}$$ |
| -5                          | $$0.13580322\times10^{-2}$$ |
|=============================+=============================|
| -8                          | $$-0.1\times10^{1}$$        |
