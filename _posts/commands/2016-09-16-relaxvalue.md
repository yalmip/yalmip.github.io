---
layout: single
category: command
author_profile: false
excerpt: "Extract value of expression after solving relaxation"
title: relaxvalue
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[value](/command/value) is used to extract the numerical value of an expression after solving a relaxation.

### Syntax

````matlab
Y = relaxvalue(X)
````

### Examples

Consider the following trivial relaxed problem


````matlab
sdpvar x
Model = [2 <= [x x^2]];
Objective = x + x^2
optimize(Model,Objective,sdpsettings('relax',1))
````

The variable \\x\\) and the relaxed variable of \\(x^2\\) will both have the value 2 at the optimum.

The optimal value of the relaxed problem is 
````matlab
value(x)
value(x^2)
relaxvalue(x^2)
````

