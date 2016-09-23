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
Model = [[2 3] <= [x x^2]];
Objective = x + x^2
optimize(Model,Objective,sdpsettings('relax',1))
````

The variable \\(x\\) and the relaxed variable of \\(x^2\\) which will be independent in the problem will have values 2 and 3 at the optimum.

````matlab
value(x)
ans =

    2.0000
    
value(x^2)
ans =

    4.0000
    
relaxvalue(x^2)
ans =

    3.0000
    
````

