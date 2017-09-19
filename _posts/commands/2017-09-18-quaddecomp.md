---
layout: single
category: command
author_profile: false
excerpt: "Extract variable value"
title: quaddecomp
tags: [Low-level]
comments: true
date: '2017-09-18'
sidebar:
  nav: "commands"
---

[quaddecomp](/command/quaddecomp) is used to extract the numerical representation of a quadratic expression

### Syntax

````matlab
[Q,c,f,z,info] = quaddecomp(p)
````

### Examples

Define a quadratic function and extract data generating the quadratic function, and check that the data is correct. Note that the order of the variables might be completely different compared to your natural ordering, hence the 4th output collects variables in the used order.

````matlab
x = sdpvar(3,1);
y = sdpvar(3,1);

p = sum(x+2*y)^2 - sum(2*x.^2+y.^2) + sum(x+2*y);

[Q,c,f,z] = quaddecomp(p);

p - (z'*Q*z + c'*z + f)
````

If you don't know if the expression is a quadratic your can use a 5th output where non-zero value indicates non-quadratic. If you only use 4 outputs and the expression is of degree larger than 2, an error will be issued.

````matlab
[Q,c,f,z,info] = quaddecomp(sdpvar(1)^3);
>> info

info =

     1
````
