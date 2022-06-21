---
category: command
excerpt: "Create linear avoidanc constraint"
title: isoutside
tags: [Polytopes, big-M]
date: '2022-06-21'
sidebar:
  nav: "commands"
---

[isoutside](/command/isoutside) creates a [big-M](/tutorial/bigmandconvexhulls) model to avoid a region specified by linear inequalities.

## Syntax

````matlab
Model = isoutside(P)
````

## Example

The following code finds the vertex closest to the origin in a polytope by solving a non-convex problem involving an avoidance constraint. Note the required explicit bounds, as the model is constructed using [big-M](/tutorial/bigmandconvexhulls) strategies.

````matlab
x = sdpvar(2,1);
A = randn(10,2);
b = 10*rand(10,1);

Model = isoutside(A*x <= b)

optimize([Model, -100 <= x <= 100],x'*x)
plot(A*x <= b);hold on
plot(value(x(1)),value(x(2)),'*')
````

