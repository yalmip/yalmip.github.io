---
category: command
excerpt: ""
title: allvariables
tags: []
date: '2021-05-03'
sidebar:
  nav: "commands"
---

[allvariables](/command/allvariables) creates a vector with variables used in a set of objects

### Syntax  

````matlab
y = allvariables(A,B,...)
````

### Examples

Create a set of constraints and an objective, and then create an new objective function which adds a penalty on all involved variables

````matlab
x = sdpvar(3,1);y = sdpvar(1);
Model = [-1 <= x <=1, y == sum(x)];

Obj = sum(x) + y^2;
Obj2 = Obj + norm(allvariables(Model,Obj))

````
