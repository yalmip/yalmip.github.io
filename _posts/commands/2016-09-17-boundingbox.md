---
layout: single
category: command
author_profile: false
excerpt: "Create model representing the bounding box of set of constraints"
title: boundingbox
tags: 
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[[boundingbox]] computes the smallest possible bounding box of a set.

### Syntax

````matlab
[B,L,U] = boundingbox(Constraint,options,x)
````

### Example
The typical use is as follows

````matlab
sdpvar x y
Ball = x^2+y^2 <= 1
Box = boundingbox(Ball);
plot(Box,[x y]);
hold on
plot(Ball,[x y],'y')
````

By using more outputs, we can extract the numerical bounds also

````matlab
[Bound,L,U] = boundingbox(Ball);
````

One can also compute the bounding box w.r.t a subset of the variables, i.e., the bounding box of a projection (this is the recommended way to use the command, since you will have no knowledge about the relation between your variables of interest and the ordering in ***L*** and ***U*** otherwise)

````matlab
sdpvar x y z
Ball = x^2+y^2 +z^2 <= 1
Box = boundingbox(Ball,[],[x y]);
plot(Box,[x y]);
hold on
plot(Ball,[x y z],'y')
````

### Comments

The command solves a series of optimization problems in order to find the maximum and minimum value of all variables, and use these values to construct a constraint defining the bounding box. Since the computation is based on repeated optimization (i.e., the bounding box is not computed analytically) it is assumed that the model is convex so that the global optima is found in every direction.
