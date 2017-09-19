---
layout: single
category: command
author_profile: false
excerpt: "Extract variable indicies"
title: getvariables
tags: [Low-level]
comments: true
date: '2017-09-18'
sidebar:
  nav: "commands"
---

[getvariables](/command/getvariables) returns the internal variable indicies of the variables used in an expression

### Syntax

````matlab
i = getvariables(p)
````

### Examples

[sdpvar](/commad/sdpvar) variables are built-up from internal variables referenced through indicies. For every new decision variable that is introduced, a new internal reference is added internally.

````matlab
yalmip('clear')
x = sdpvar(3,1);
y = sdpvar(3,1);

getvariables(x)

ans =

     1     2     3
     
getvariables(y)

ans =

     4     5     6
````

Variables do not have any connection to the name that is used in the work-space, but is uniquely defined by the index. By using the command recover, we can this to create the variable corresponding to a variable index

````matlab
x = sdpvar(3,1);
i = getvariables(x);
y = recover(i);
x-y
ans =

     0
     0
     0

````

Variable indicies are also use to reference nonlinear terms internally. Hence, for every new nonlinear term introduced, a new variable index is created. 

````matlab
yalmip('clear')
x = sdpvar(3,1);
y = x.^2;
i = getvariables(y)
i =

     4     5     6
>> recover(i)
Quadratic matrix variable 3x1 (full, real, 3 variables)
````

