---
layout: single
category: command
author_profile: false
excerpt: "Computes the largest possible inscribed ball in a polytope"
title: boundingbox
tags: [Polytopes]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[chebyball](/command/chebyball) computes the largest possible ball inscribed in a polytope.

### Syntax

````matlab
[xc,r] = chebyball(Constraint,options)
````

### Example
The typical use is as follows

````matlab
sdpvar x y
Box = -1 <= [x y] <= 1;
[xc,r] = chebyball(Box);
plot(Box);
hold on
plot((x-xc(1)^2 + (y-xc(2))^2 <= r^2,[],'y')
````

The command solves a linear program, hence a linear programming solver must be available. If you want to control the selection of LP solver, you use the second options argument.


