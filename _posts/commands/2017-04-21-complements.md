---
layout: single
category: command
author_profile: false
excerpt: ""
title: complements
tags: [Logic programming]
comments: true
date: '2017-04-21'
sidebar:
  nav: "commands"
---

[complements](/command/complements) implements a complementarity constraint \\(x\geq 0, y\geq 0, x^Ty = 0\\)

### Syntax

````matlab
F = implies(Constraint1,Constraint2)
````

### Examples

[complements](/command/complements) applies to linear constraints and defines the associated complementarity constraints that one of the constraints is active. As an example, the following model states that \\(y=0\\) and \\(x+z\geq 1\\), or \\(x+z = 1\\) while \\(y\geq 0\\).
````matlab
sdpvar x y z
F = complements(y >= 0, x+z >= 1)
````


### Comments
Since [complements](/command/complements) is implemented using a [big-M](/tutorial/bigmandconvexhulls) approach when you use a mixed-integer solver, it is crucial that all involved variables have explicit bound constraints.

When a general nonlinear solver is used, the model is implemented by simply using the nonlinear form \\(x^Ty=0\\).
