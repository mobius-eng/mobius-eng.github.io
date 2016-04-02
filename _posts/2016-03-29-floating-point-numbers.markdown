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
<!--more-->

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

{% newthought "A general approach" %} to floating-point arithmetic operations is the same one would use for scientific notation with the addition of keeping mantissa length and exponent bound. For example, in scientific notation addition

{% math %}
2.01\times10^{4} + 3.4\times10^{2}
{% endmath %}

will require equating exponents{% sidenote "sn-exp-sub" "To any value, easiest --- to the lowest." %}, multiplying mantissas by $$10^n$$ where $$n$$ is the change of exponent for each number, and finally adding mantissas and keeping the exponent parts{% sidenote "sn-dist" "This is the use of distribution law: $$ab + ac = a(b+c)$$ where $$a$$ is exponent part of the number." %}:

{% math %}
201.0\times10^{2} + 3.4\times10^{2} = 204.4\times10^{2} = 2.044\times10^{4}.
{% endmath %}

Multiplication is simpler: mantissas are multiplied, exponents are added:

{% math %}
(2.01\times10^4)\times(3.4\times10^{2}) = 6.834\times10^{6}.
{% endmath %}

Now let's look at strategies of keeping mantissas and exponents bound. Let's look at the addition of $$0.360$$ and $$12.5$$:

{% math %}
(+,360, 0) + (+,125,2)
{% endmath %}

First, their exponents are equalised:

{% math %}
\begin{align*}
(+,360,0) + (+,125,2) & = (+,00360,2) + (+,125,2)\\
& = (+,128\cdot60, 2)
\end{align*}
{% endmath %}

Now we have the excess of digits in mantissa (denoted by $$\cdot$$): last two digits need to be dropped. There are two possible ways of dropping: either just truncate them, or round them, in which case the last significant digit might increase if the following digit is $$\geq 5$${% sidenote "sn-add-round" "Notice that we have temporally extended mantissa's width for this operation. Alternatively, we could have brought exponents to the largest number, rounding the mantissa of the smallest number *before* the addition. While this would produce the same result for addition, this strategy will have some detrimental properties for subtraction and thus avoided." %}. Simple truncation, although faster, leads to a higher rate of precision loss. Considering our low starting precision, we will use rounding. Thus the final result becomes:

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
i.e. there is 1 *before* the point. This overflow needs to be consumed by the
exponent:

{% math %}
(+,1'100, 1) = (+, 110, 2)
{% endmath %}

Obviously, if two numbers are very large, their sum might exceed
the largest possible number in which case their result will become infinite:

{% math %}
(+,700,9) + (+,600,9) = (+,999,9) = +\infty
{% endmath %}

Multiplication of floating-point numbers resembles the multiplication in scientific notation with one complication: we need to take care of
the fact that mantissas are fractional parts. For example in multiplication
$$3.0\times20.0=60.0$$ mantissa terms will multiply as $$200\times300=60000$$.
This result needs to be adjusted by dividing it by $$10^{3}$$, i.e. 10 raised to
the power of mantissa's width.

{% math %}
\begin{align*}
3.0\times 20.0 = (+,300,1) \times (+,200,2) & = (+,060\cdot000, 1+2)\\
& = (+, 060, 3) = (+, 600,2) = 60.0
\end{align*}
{% endmath %}

This example shows *mantissa underflow* that is happened during multiplication: the result was a subnormal number $$(+, 060, 3)$$ that needed to be normalised by decreasing exponent and shifting the mantissa to the left. At this point it is important to keep one{% sidenote "sn-mul" "In fact, two digit: the first digit to take place and the second digit to be used for rounding" %} extra digit from the multiplication result to populate the mantissa's last digit. To illustrate this point, consider multiplication:

{% math %}
(+,301,1) \times (+,202,2) = (+,060\cdot802,3)=(+,608,2)
{% endmath %}

Subtraction is very similar to addition. For example,

{% math %}
\begin{align*}
(+, 283, 1) - (+, 532,-1) & = (+,283, 1) - (+,00532,1)\\
 & = (+, 277\cdot68,1) = (+, 278, 1)
\end{align*}
{% endmath %}

Division is best viewed as a multiplication by the reciprocal of the denominator:

{% math %}
(+, 283, 1) / (+, 532, -1) = (+,283,1) \times \text{recip}(+,532,-1)
{% endmath %}

The reciprocal is found as: (a) the exponent is negated and increased by 1, and (b) a rounded division of 1.0 by mantissa. Part (a) is mostly intuitive: reciprocal does imply the negation of exponent. The increase by 1 is due to the fact that numbers are represented with leading zero before point. Consider number $$1.0=(+,100,1)$$ its reciprocal should be itself. But negation of 1 will produce $$-1$$, that needs will be partially consumed by the increase and partially by mantissa overflow (see further below). Part (b) needs to be modified to operate on integer representation of mantissa. First, by multiplying both 1.0 and mantissa by 1000, the operation of 1.0 dividing by mantissa-as-a-fraction is equivalent to the division of 1000 by the mantissa-as-an-integer. However, since mantissa ranges from 100 to 999, the division will produce the result in the range of 1 to 10. To avoid dealing with non-trivial underflows, we can, instead, divide the number which is 100 times larger than 1000 by the mantissa. The range of the results will be from 100 to 1000 with the overflowed results occurring  only for mantissa 100 (and coincidently, it will provide extra 1 to exponent for reciprocal of $$1.0=(+,100,1)$$). Thus,

{% math %}
\text{recip}(+, 532, -1) = (+, 188, 2)
{% endmath %}

and division is reduced to multiplication:

{% math %}
\begin{align*}
(+,283,1)/(+,532,-1) = (+,283,1) \times (+,188,2) & = (+,53\cdot204, 3)\\
& = (+, 532, 2).
\end{align*}
{% endmath %}

## Machine epsilon or dangers of floating-point arithmetic

{% newthought "Let's look at some properties" %} of floating-point arithmetic in  bit more detail. All of these properties arise from limited precision defined by the fixed length of mantissa. In floating-point addition, for example, if a very large number is added to a very small number, the effect of the small number on the large number is zero:

{% math %}
(+,100,9) + (+,100,-9) = (+,100\cdot0\ldots1, 9) = (+,100,9)
{% endmath %}

This effect is obviously relative: what matters the most is the difference between the exponents. To have some point of reference, let's fix the largest number at 1.0. The largest positive number that won't make a difference in the
floating-point system with mantissa's width 3 is:

{% math %}
(+,100,1) + (+,499,-2) = (+,100\cdot499,1)=(+,100,1)
{% endmath %}

The smallest positive number that when added to 1.0 produces the result different from one is called *machine precision*. In this system it is the number $$\boldsymbol{u}=(+,500,-2)=0.005$$.

*Negative machine precision* is defined as the smallest number that upon subtraction from 1.0 will produce the result smaller than 1.0. Surprisingly, it is about ten times smaller than machine precision{% sidenote "sn-neg-eps" "This calculation depends on the intermediate result to have extended mantissa. If subtrahend here is rounded to fit the precision width of the minuend prior subtraction, negative machine precision will be larger, $$(+,500,-2)$$." %}:

{% math %}
(+,100,1)-(+,501,-3)=(+,099\cdot9499,1) = (+,999,0)
{% endmath %}

Now let's look at a different scenario: two close numbers are subtracted, for example, $$1.0$$ and $$0.999$$:

{% math %}
(+,100,1) - (+,999,0) = (+,000\cdot1,1) = (+,100,-2)
{% endmath %}

Notice what has happened: we subtracted two numbers with the precision
of up to three digits, but the result has a precision of only one
digit{% sidenote "sn-sub-loss" "We filled essentially non-existent extra two digits of significance with zeros. However, there is no guarantee that in real computing system that will happen, the result just may have been $$(+,107,-2)$$" %}. This is called *loss of significance*.

In multiplication both mantissa underflow and exponent underflow can result in zero product while multiplying two non-zero numbers:

{% math %}
\begin{align*}
(+,200,-4) \times (+,300,-5) &= (+,060\cdot000,-9)\\
&\{\text{mantissa underflow}\} = (+,600,-10)\\
&\{\text{exponent underflow}\} = (+, 000, 0)
\end{align*}
{% endmath %}

Multiplication by a very large number or, equivalently, division by a very small number while does not present the problem purely arithmetically, the danger is uncovered once the result is analysed.{% marginfigure "mfig-reciprocal" "assets/img/reciprocal.png" "The uncertainty interval increases as the reciprocal of a small number is taken." %} Consider, for example, reciprocal of a number $$(+, 532, -4)$$:

{% math %}
\text{recip}(5.32\times10^{-5})=\text{recip}(+,532, -4) = (+,188,5) = 18800
{% endmath %}

Now, let's imagine this number was a result of a measurement and there is an uncertainty about its last digit: $$(5.32\pm0.01)\times10^{-5}$$. Also, let's assume that this particular measurement was unusually small, normally it would fall into the range of $$0.1-1.0$$. For normal range of the measurement the uncertainty of reciprocal (with three digits accuracy) is not exceeding $$0.2$$. For this measurement, however, it becomes larger than $$70$$. While $$70$$ is still small relative to the result 18800, the result itself is blown up, causing the error to grow with it *relative to other results*.

As a consequence of above mentioned properties, floating-point arithmetic does not strictly obey theoretical laws, such as distribution and associativity. For example, in the equation:

{% math %}
a(b+c) = ab +ac
{% endmath %}

if all numbers are sufficiently small, both multiplications of right-hand-side may result in zero, causing the sum to become zero. Yet, if $$b$$ and $$c$$ are still big enough for their sum not to produce zero as it is multiplied by $$a$$, the left-hand-side will be non-zero. Or more concrete:

{% math %}
\begin{align*}
(+,200,-4) \times\left((+,300,-5) + (+,300,-5)\right) & = (+,200,-4)\times (+,600,-5)\\
(+, 120, -9)
\end{align*}

Yet,

{% math %}
(+,200,-4)\times(+,300,-5) + (+,200,-4)\times(+,300,-5) = (+, 0, 0)
{% endmath %}

as was shown before. Similarly, addition is not associative:

{% math %}
(1.0 + \varepsilon) + \varepsilon \neq 1.0 + (\varepsilon + \varepsilon),
{% endmath %}

where $$\varepsilon$$ is a number in the range $$1/2\boldsymbol{u}<\varepsilon<\boldsymbol{u}$$. The left hand side is equal to one since each addend is below machine precision. In contrast, the value larger than machine precision is added to one on the right hand side.

A practical implication of round off error effect can be seen in approximating the derivative by finite difference:

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
