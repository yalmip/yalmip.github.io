---
layout: single
category: command
author_profile: false
excerpt: "Find the internal variable index"
title: getvariables
tags: [Low-level]
comments: true
date: '2017-09-18'
sidebar:
  nav: "commands"
---

[getvariables](/command/getvariables) returns the internal index associated with an [sdpvar](/command/sdpvar) variable

### Syntax

````matlab
k = getvariables(x)
````

### Examples

[sdpvar](/commad/sdpvar) variables are built-up from internal variable indicies (i.e., the only internal name is a number). 

````matlab
yalmip('clear')
x = sdpvar(1);
getvariables(x)

ans = 
    1

y = sdpvar(2,3);
getvariables(y)
ans = 
    2 3 4 5 6 7 
````

Also nonlinear terms have internal indicies (and assosciated data to explain how the term relates to other linear variables)

````matlab
z = x^2;
getvariables(z)
ans = 
    8
````

Expressions are built up from variables

````matlab
z = x + 3*x^2;
getvariables(z)
ans = 
    1 8
````

