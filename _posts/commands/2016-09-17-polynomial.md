---
category: command
excerpt: ""
title: polynomial
tags: [Polynomials]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[polynomial](/command/polynomial) is used to generate a parameterized polynomial.

## Syntax  

````matlab
[p,c,v] = polynomial(x,degreemax,degreemin)
````

## Examples
[polynomial](/command/polynomial) is a convenient way to define parametrized polynomials in one command. The following example defines a quartic in 2 variables.
````matlabb
x = sdpvar(2,1);
[p,c,v] = polynomial(x,4);
sdisplay(p)
````

The second output are the coefficients that parametrize the polynomial and the third output are the involved monomials.

The command is equivalent to the following code.
````matlabb
v = monolist(x,4);
c = sdpvar(length(v),1);
p = c'*v;
````
