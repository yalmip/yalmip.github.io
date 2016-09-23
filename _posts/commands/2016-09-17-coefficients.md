---
layout: single
category: command
author_profile: false
excerpt: "Extract coefficients in polynomial expression"
title: coefficients
tags: [Polynomials]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[coefficients](/command/coefficients) is used to extract coefficients in a polynomial.

### Syntax


````matlab
[c,v] = coefficients(p,x)
````

### Examples

Define a polynomial in variables **x** and **y**, with coefficients parameterized by **s** and **t**.

````matlab
sdpvar x y s t
p = x^2+x*y*(s+t)+s^2+t^2;
````

The coefficients are easily recovered

````matlab
c = coefficients(p,[x y]);
sdisplay(c)

ans =
    's^2+t^2'
    's+t'
    '1'
````

By adding a second output, the monomial basis is returned also.

````matlab
[c,v] = coefficients(p,[x y]);
sdisplay([c v])

ans =
    's^2+t^2'    '1'  
    's+t'        'xy'
    '1'          'x^2'
p-c'*v

ans =
     0
````

Of course, we might just as well consider this to be a polynomial in s and t with coefficients parameterized by x and y.

````matlab
[c,v] = coefficients(p,[s t]);
sdisplay([c v])

ans =
    'x^2'    '1'  
    'x*y'     't'  
    '1'      't^2'
    'x*y'     's'  
    '1'      's^2'
````
