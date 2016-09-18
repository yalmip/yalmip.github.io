---
layout: single
category: command
author_profile: false
excerpt: ""
title: alldifferent
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

Images:mini-category.gif  [!Logic constraints], [!Mixed-integer based nonlinear operators]

### Syntax

````matlab
F = alldifferent(X)
````

### Examples

The following example finds an integer vector with all numbers between  1 and 5.
````matlabb
x = intvar(5,1);
F = [alldifferent(x), 1<=x<=5];
optimize(F)
value(x)
````

### Comments
Since [alldifferent] is implemented using a [Tutorials.Big-MAndConvexHulls big-M] approach, it is crucial that all involved variables have explicit bound constraints. Additionally, it only makes sense to apply the operator on integer valued variables.

Applying [alldifferent] on a variable with '''n''' variables will introduce '''n(n-1)/2''' binary variables.
