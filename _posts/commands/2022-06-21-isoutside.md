---
category: command
excerpt: "Create linear polytopic avoidance constraint"
title: isoutside
tags: [Logic programming, Polytopes, big-M, Avoidance constraints]
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

The following code finds the vertex/point on facet closest to the origin in a polytope by solving a non-convex problem involving an avoidance constraint. Note the required explicit bounds, as the model is constructed using [big-M](/tutorial/bigmandconvexhulls) strategies.

````matlab
x = sdpvar(2,1);
A = randn(10,2);
b = 10*rand(10,1);

Model = isoutside(A*x <= b)

optimize([Model, -100 <= x <= 100],x'*x)
clf
plot(A*x <= b,[],[],[],sdpsettings('plot.shade',0.1));
hold on;grid on
plot(value(x(1)),value(x(2)),'*')
r = sqrt(value(x'*x));
t = 0:0.01:2*pi;
plot(r*cos(t),r*sin(t),'--')
````

![Point closest to origin]({{ site.url }}/images/isoutside.png){: .center-image }


## Comments

Note that [isoutside](/command/isoutside) will introduce binary variables.

