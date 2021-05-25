---
category: command
excerpt: "Built-in meta-solver for polynomial optimization problem using semidefinite relaxations"
title: solvemoment
tags: [Semidefinite relaxations, Moment relaxations, Polynomial programming]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[solvemoment](/command/solvemoment)  computes lower bounds to polynomial programs using [Lasserre's moment-method](/reference/), i.e., semidefinite relaxations.

## Syntax

````matlab
[sol,xoptimal,momentdata,sos] = solvemoment(F,h,options,order)
````

## Examples

The command is used for finding lower bounds on a polynomial \\(h(x)\\), subject to constraints \\(F(x)\succeq 0\\), where \\(F(x)\\) is a collection of polynomial scalar or matrix inequalities (and equalities).

````matlab
x1 = sdpvar(1,1);x2 = sdpvar(1,1);x3 = sdpvar(1,1);
h = -2*x1+x2-x3;
F = [x1*(4*x1-4*x2+4*x3-20)+x2*(2*x2-2*x3+9)+x3*(2*x3-13)+24>=0,
     4-(x1+x2+x3)>=0,
     6-(3*x2+x3)>=0,
     2-x1>=0,
     3-x3>=0,
     x1>=0,
     x2>=0,
     x3>=0];
solvemoment(F,h);
value(h)
ans =
   -6.0000
````

In the code above, we solved the problem with the lowest possible lifting (decided by YALMIP), and the lower bound turned out to be -6 (this value can be obtained using [value](/command/value) since the objective is linear. In the general case, [relaxvalue] is required to retrieve relaxed values on expressions after solving relaxations). A higher order relaxation gives better bounds.

````matlab
solvemoment(F,h,[],2);
value(h)
ans =
   -5.69

solvemoment(F,h,[],3);
value(h)
ans =
   -4.0685

solvemoment(F,h,[],4);
value(h)
ans =
   -4.0000
````

For a more complete introduction, please study the [moment tutorial](/tutorial/momentrelaxations), and the quick introduction to semidefinite relaxations in [relaxvalue](/command/relaxvalue).
