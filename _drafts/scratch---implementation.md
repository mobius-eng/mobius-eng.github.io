---
title: "Scratch - implementation"
layout: post
---


[The previous post]({% post_url 2016-03-29-floating-point-numbers %}) explained how practical floating-point numbers can be represented and what properties we expect from them. This post provides the implementation of floating-point numbers in Smalltalk (Pharo-4).

## Instance variables

{% newthought "To represent a number" %} we need to keep the sign, mantissa and exponent. Furthermore, we will add a bit extra flexibility by storing mantissa's width and maximum value of the exponent. This way we can experiment with different precision width. Thus, the class instance is created as follows:

```smalltalk
Object subclass: #NMFloat
	instanceVariableNames: 'isPositive mantissaWidth mantissa exponentMax exponent'
	classVariableNames: ''
	category: 'NumericalMethods'
```
<!--more-->

Just as a reminder of Smalltalk syntax, here we create a new class `NMFloat` by sending the message

```smalltalk
subclass:instanceVariableNames:classVariableNames:category:
```

to class `Object`. The shape of the number is defined by `mantissaWidth` and `exponentMax`. The sign is stored as a Boolean variable `isPositive`.

For simplicity, this class defines the accessors to all instance variables (i.e. it defines methods `isPositive` and `isPositive:`, `mantissa` and `mantissa:`, etc.). However, we will treat the numbers as constant objects.

## Integrity of floating-point numbers

{% newthought "Each arithmetic operation" %} might produce a number with mantissa or exponent underflow or overflow, or with mantissa that is too long to
fit into mantissa's width. We will implement the method `checkIntegrity` which will put the number into normal form, producing infinities or zero in case of exponent overflow or underflow respectively.

```smalltalk
checkIntegrity
	| diff |
	diff := mantissa decimalDigitLength - mantissaWidth.
	"Mantissa is too large"
	diff > 0 ifTrue: [
		mantissa := (mantissa / (10 raisedTo: diff)) rounded.
		exponent := exponent + diff.
		^ self checkIntegrity ].
	"Mantissa is too small, but not zero"
	(mantissa ~= 0 and: [ diff < 0 ])
		ifTrue: [
			mantissa := mantissa * (10 raisedTo: diff negated).
			exponent := exponent + diff. ].
	"Exponent overflow"
	exponent > exponentMax
		ifTrue: [ self makeInfinity ].
	"Exponent underflow"
	exponent < exponentMax negated
		ifTrue: [ self makeZero ]
```

There is one little cheat inside this method: `(mantissa / (10 raisedTo: diff)) rounded`. Strictly speaking, since we are allowing only integer operations, we should have produced quotient and remainder and implemented rounding by checking the size of the remainder {% sidenote "sn-int-div" "On the other hand, in Smalltalk the division of two integers produces a rational fraction. Thus, this operation still does not involve floating-point numbers" %}.

Notice, this method calls itself recursively after the rounding of the mantissa is performed: the reason is that rounding might increase number of decimal digits, for example, $$(+,999\cdot5,1)$$ once rounded becomes $$(+,1000,1)$$ and another mantissa adjustment is in order.

## Double dispatch arithmetic operations

{% newthought "Most arithmetic operations are binary" %}, for example $$a - b$$. Since Smalltalk cannot dispatch (or *specialise*) on two arguments of the method (the first being `self`), we have to use the trick called *double dispatch*. Most object-oriented programming books point out that double dispatch is usually the sign of the *code smell*. However, when it comes to arithmetic it is the nature of the operations that dictate the use of the double dispatch{% sidenote "sn-math-smell" "One can argue, it is the problem of the design of mathematics: after all such a seemingly simple operation as the summation is, in fact, incredibly complex once we realise that we can add numbers, vectors, polynomials, functions, tensors etc." %}. The idea is that when a floating-point number receives the message `+ aNumber` it doesn't know what this number is. But it knows that `self` is a floating-point number, so it will delegate the addition to the method `addingFloatingPointNumber:` that must be implemented by `aNumber`. In `addingFloatingPointNumber: aFloat` it is known that the argument is `NMFloat` kind. Thus, the specialisaion on both arguments occur. It looks as a bit of the long way around compared to multi-methods (as for example in Common Lisp), but the benefit is that each method is still local to the class and there is no need for a global dispatch structure.

```smalltalk
+ aNumber
	^ aNumber addingFloatingPointNumber: self
```

Other operations are implemented similarly:

```smalltalk
* aNumber
	^ aNumber multiplyingFloatingPointNumber: self

- aNumber
	^ aNumber subtractingFromFloatingPointNumber: self

/ aNumber
	^ aNumber dividingFloatingPointNumber: self
```

Since we interested in these operations between floating-point numbers, all the delegate methods need to be implemented by `NMFloat`.

## Float-float arithmetic operations

{% newthought "We can group arithmetic operations" %} to summation-subtraction and multiplication-division as the implementation within the group is somewhat similar.

Addition and subtraction are quite interconnected: addition of two numbers of opposite signs can be replaced by the subtraction of the first number from the negative of the second number. Thus, we will only process numbers of the same signs directly. And if signs are different, one of the numbers will be negated and sent from addition to subtraction or from subtraction to addition.

Let's look at the addition:

```smalltalk
addingFloatingPointNumber: aFloat
	| result maxExp m1 m2 m |
	"If signs are different - send it to subtraction"
	self isPositive = aFloat isPositive
		ifFalse: [ ^ self negated subtractingFromFloatingPointNumber: aFloat ].
	"initialize the result"
	result := NMFloat new.
	result mantissaWidth: (mantissaWidth max: (aFloat mantissaWidth)).
	result exponentMax: (exponentMax max: aFloat exponentMax).
	"The exponent of the result will maximum of two exponents
		or, in case of mantissa overflow, maximum plus one."
	maxExp := self exponent max: aFloat exponent.
	"Bring both mantissas to the maxExp and increase the length if necessary"
	m1 := (self mantissa / (10 raisedTo: maxExp - exponent - result mantissaWidth + mantissaWidth )) rounded.
	m2 := (aFloat mantissa / (10 raisedTo: maxExp - aFloat exponent - result mantissaWidth + aFloat mantissaWidth )) rounded.
	"Add the mantissas"
	m := m1 + m2.
	"Populate the result"
	result mantissa: m.
	result exponent: maxExp.
	result isPositive: isPositive.
	"Deal with overflow"
	result checkIntegrity.
	^ result
```

One complication of this method implementation is that floating-point numbers might have different mantissa's widths of maximum exponents. It takes this into account by extending each mantissa to the maximum width. As the result might produce overflow in mantissa, need to perform `result checkIntegrity`.

Subtraction is similar:

```smalltalk
subtractingFromFloatingPointNumber: aFloat
	| result  m1 m2 m minExp |
	self isPositive ~= aFloat isPositive
		ifTrue: [ ^ self negated addingFloatingPointNumber: aFloat ].
	result := NMFloat new.
	result mantissaWidth: (mantissaWidth max: (aFloat mantissaWidth)).
	result exponentMax: (exponentMax max: aFloat exponentMax).
	minExp := self exponent min: aFloat exponent.
	m1 := self mantissa * (10 raisedTo: exponent - minExp + result mantissaWidth - mantissaWidth).
	m2 := aFloat mantissa * (10 raisedTo: aFloat exponent - minExp + result mantissaWidth - aFloat mantissaWidth).
	m := m2 - m1.
	result isPositive: aFloat isPositive.
	m < 0
		ifTrue: [
			result isPositive: result isPositive not.
			m := m negated ].
	result mantissa: m.
	m = 0 ifTrue: [ result exponent: 0 ] ifFalse: [ result exponent: minExp ].
	result checkIntegrity.
	^ result
```

Note the semantics of this method: `self` is subtracted from `aFloat`. Here there is slight difference compared to addition: mantissas are extra extended before the actual subtraction takes place. As it was [discussed previously]({% post_link 2016-03-29-floating-point-numbers subtraction-strategies %}), premature rounding of the mantissas in subtraction leads to significant loss of precision.

Multiplication is actually simpler:

```smalltalk
multiplyingFloatingPointNumber: aFloat
	| result |
	result := NMFloat new.
	result mantissaWidth: (mantissaWidth max: aFloat mantissaWidth).
	result exponentMax: (exponentMax max: aFloat exponentMax).
	"'+' * '+' = '+', '-' * '-' = '+', '-' * '+' = '-'"
	result isPositive: (isPositive xor: aFloat isPositive) not.
	result mantissa: (mantissa * aFloat mantissa).
	"add exponents and take into account that mantissa multiplication is
		the multiplication of fractionals"
	result exponent: (exponent + aFloat exponent - mantissaWidth - aFloat mantissaWidth + result mantissaWidth).
	result checkIntegrity.
	^ result
```

The only non-trivial part is the resulting exponent. Consider the multiplication

{% math %}
(0.x_1x_2\dots x_n\times10^{p})\times(0.y_1\dots y_m\times10^{q})
{% endmath %}

Exponent parts are simply added:

{% math %}
10^{p}\times10^{q} = 10^{p+1}
{% endmath %}

Mantissas multiplication can be expressed as following

{% math %}
0.x_1\dots x_n \times 0.y_1\dots y_m = x_1\dots x_n\times y_1\dots y_m \times10^{-n-m}
{% endmath %}

This explains `-mantissaWidth - aFloat mantissaWidth` part of the result exponent. Finally, since the result of of mantissas multiplication

{% math %}
x_1\dots x_n\times y_1\dots y_m = z_1\dots z_k
{% endmath %}

is stored in `mantissa` slot, this result is viewed as

{% math %}
z_1 \dots z_l . z_{l+1}\dots z_k
{% endmath %}

with $$k-l = \max{(n,m)}$$ being the mantissa width of the result and the first $$l$$ digits being  the mantissa overflow. To account for this view, an extra `result mantissaWidth` needs to be added to the exponent.

Division is reduced to multiplication by reciprocal:

```smalltalk
dividingFloatingPointNumber: aFloat
	^ aFloat * (self reciprocal)
```

Reciprocal implements the [explanation from the previous post]({% post_link 2016-03-29-floating-point-numbers reciprocal %}):

```smalltalk
reciprocal
	| result m1 |
	self isZero
		ifTrue: [ isPositive
				ifTrue: [ ^ self positiveInfinity ]
				ifFalse: [ ^ self negativeInfinity ] ].
	result := self copy.
	m1 := 10 raisedTo: (2 * mantissaWidth - 1).
	result mantissa: (m1 / mantissa) rounded.
	result exponent: exponent negated + 1.
	result checkIntegrity.
	^ result
```

There are still a few other methods that need to be implement for the above code to work, but they are quite trivial and left as an exercise{% sidenote "sn-repo" "The full implementation of floating point numbers in Smalltalk will be made openly available soon." %}.

## Further extensions

{% newthought "This implementation of floating point numbers" %} offers an insight of how these numbers work. To properly incorporate them into the numerical tower is a bit more challenging. Consider message `+`: if in expression `a + b` object `a` is an instance of `NMFloat`, `addingFloatingPointNumber: a` will be sent to `b`. If `b` is not `NMFloat`, `addingFloatingPointNumber:` needs to implemented for its type. This means, essentially, the implementation of conversion of integers and built-in floats to `NMFloat`. Things get more complicated if `a` is *not* `NMFloat`. In Pharo, the `Integer` implements `+` as follows:

```smalltalk
+ aNumber
	"Refer to the comment in Number + "
	aNumber isInteger ifTrue:
		[self negative == aNumber negative
			ifTrue: [^ (self digitAdd: aNumber) normalize]
			ifFalse: [^ self digitSubtract: aNumber]].
	aNumber isFraction ifTrue:
		[^Fraction numerator: self * aNumber denominator + aNumber numerator denominator: aNumber denominator].
	^ aNumber adaptToInteger: self andSend: #+
```

So, to be able to work with integers, `NMFloat` needs to respond to `adaptToInteger:andSend:` message. Similarly, for the `Fraction` it needs to respond to `adaptToFraction:andSend:`, and so on. And as the number of number-like classes increases, the number of such messages and their implementations increase quadratically. This is the complexity inherited from the complexity of actual arithmetic and not much can be done in the programming to simplify it (other than trying not to increase the complexity).
