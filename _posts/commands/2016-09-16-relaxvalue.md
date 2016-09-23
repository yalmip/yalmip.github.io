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

[relaxvalue](/command/relaxvalue) is used to extract the relaxed numerical value of an expression after solving a relaxation.

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

The variable \\(x\\) and the relaxed variable of \\(x^2\\) which will be independent in the problem will both have the value 2 at the optimum.

````matlab
value(x)
value(x^2)
relaxvalue(x^2)
````

